---
layout: post
title:  "Dynamically adding associations with Live View"
date:   2024-01-11 15:11:00 -0300
categories: ["phoenix", "live view", "elixir", "tutorial"]
---

Sometimes we have entities that are associated with a form of has many association and we want to be able to, on the same form we create one entity, create one or more of it's associated entities. In other words, we want nested forms.

Ordinarily we'd need to write quite a bit of javascript to make this work but with live view we get to do things differently. And in an easier way.

## The tool for the job
You can do an online search on how to do this and certainly you might find a few tutorials. However, documentation in the Elixir world is truly wonderful.

It just so happens Live View already has the thing we need: the [inputs_for component](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#inputs_for/1).

As the docs say, it renders nested form inputs for associations or embeds.

In our specific case, we have a clinic that can have many offices and we want our users to be able to create or edit those offices from the same form they create or edit their clinic's info.

That is where `inputs_for` comes in.

Say we have this simple form:

```html
<.simple_form
  for={@form}
  id="clinic-form"
  phx-target={@myself}
  phx-change="validate"
  phx-submit="save"
>
  <.input field={@form[:name]} type="text" label="Name" />
  <.input field={@form[:address]} type="text" label="Address" />
  <.input field={@form[:postal_code]} type="text" label="Postal code" />
  <.input field={@form[:city]} type="text" label="City" />
  <.input field={@form[:state]} type="text" label="State" />
  <.input field={@form[:country]} type="text" label="Country" />
  <:actions>
    <.button phx-disable-with="Saving...">Save Clinic</.button>
  </:actions>
</.simple_form>
```

It's pretty standard. In fact, it's the one Phoenix made for me when I created the models. Following the docs for input for, we just need to add this to it:

```html
  <label class="block cursor-pointer">
    <input type="checkbox" name="clinic[offices_sort][]" class="hidden" />
    add more
  </label>
  <.inputs_for :let={office} field={@form[:offices]}>
    <input type="hidden" name="clinic[offices_sort][]" value={office.index} />
    <.input field={office[:identifier]} type="text" label="Name" />
    <.input field={office[:amenities]} type="text" label="Amenities" />
    <label>
      <input type="checkbox" name="clinic[offices_drop][]" value={office.index} class="hidden" />
      <.icon name="hero-x-mark" class="w-6 h-6 relative top-2" />
    </label>
  </.inputs_for>
  <input type="hidden" name="clinic[offices_drop][]" />
```

I'm going to try to explain what this is doing.

First item we have is a `label` tag with a hidden `input` and a text saying "add more". This is just the button that will trigger the addition of the form inputs inside the `inputs_for` component. The way it works is that we take advantage of Ecto's `sort_param` option on the `cast` function to add empty `Office` changesets whenever the `offices_sort` request parameter isn't recognized (more on this later).

Next we have the actual `inputs_for` tag. Here we need to give it the parameter to which it will insert whatever input fields we have inside. In our example, we're saying we want the `identifier` and `amenities` keys to be sent over as one of the offices under the `offices` key. The way `inputs_for` does this is that it assigns a numeric string to each nested form we add, so, supposing we had 2 office inputs, once we submitted our form, this is what our params would look like:

```elixir
%{
  "clinic" => %{
    "address" => "Rua A, 134",
    "city" => "Fortaleza",
    "country" => "Brasil",
    "name" => "Clínica 1",
    "offices" => %{
      "0" => %{
        "_persistent_id" => "0",
        "amenities" => "sjdfjhsghf jshgdf jsdfhgs",
        "identifier" => "123"
      },
      "1" => %{
        "_persistent_id" => "1",
        "amenities" => "lkajsdflakjshdf laksjdhf asjdhf",
        "identifier" => "736"
      }
    }
  }
}
```

Also, notice the hidden input field called `clinic[offices_sort][]`. This parameter is used in combination with the `sort_param` option to order how associated elements' inputs will appear.

Then we have a hidden checkbox input, this time inside a `label`, whose purpose is to inform Ecto which child data should be removed.

Finally, outside the `inputs_for` tag we have another hidden input named `clinic[offices_drop][]` which is responsible for informing ecto that it should delete all the child objects, when the user clears all previous associations in the form.

Now we need to make some changes to the backend.
## On the schema side
First, you need to make sure that your schemas have the proper declarations. Since we have a has many/belongs to relation in our example, in our clinic schema that's the first thing to add:

```elixir
schema "clinics" do
  field :name, :string
  field :state, :string
  field :address, :string
  field :postal_code, :string
  field :city, :string
  field :country, :string
  field :number_of_offices, :integer
  field :user_id, :id
+ has_many :offices, Office, on_replace: :delete

  timestamps()
end
```

Furthermore, in order to properly handle the addition of new associated offices changesets so that `inputs_for` renders them correctly, we need to modify our `changeset` function:

```elixir
  def changeset(clinic, attrs) do
    clinic
    |> cast(attrs, [:name, :address, :postal_code, :city, :state, :country])
-   |> cast_assoc(:offices)
+   |> cast_assoc(
+     :offices,
+     with: &Office.changeset/2,
+     sort_param: :offices_sort,
+     drop_param: :offices_drop
+   )
    |> validate_required([:name, :address, :postal_code, :city, :state, :country])
  end
```

Here you can see the `sort_param` and `drop_param` options we were talking about. By telling the `cast_assoc` function to look at those parameters for sorting and dropping associated changesets, we get the behavior we wanted initially. If there is an unrecognized value in `offices_sort` (Any value not equal to one of the ids already associated), ecto interprets that as a new entry to be added. Naturally, if any ids are in `offices_drop`, they get dropped.

Finally, for this to work we also need to make sure the associated model has the proper function calls. Namely, in this example, a call to `belongs_to`. Otherwise, once it receives the parameters to create an `Office` changeset, because we don't explicitly set a `clinic_id`, it would fail validations. By calling `has_many :offices, Office` in the clinic schema and calling `belongs_to :clinic, Clinic` in the office schema, ecto will know, through `cast_assoc`, how to fill in `clinic_id`.

```elixir
  schema "offices" do
    field :identifier, :string
    field :amenities, :string
-   field :clinic_id, :id
+   belongs_to :clinic, Clinic

    timestamps()
  end
```

And we're done. Check it out:
![[Screen Recording 2023-11-30 at 6.42.48 AM.mov]]
## Conclusion

My experience with nested forms in Rails never was the best one. But I don't want to say it's bad because I used it when I was pretty new to the job and haven't touched it since.

But because of that experience, it does feel pretty good to have a ready solution in the framework that's so easy to implement.
