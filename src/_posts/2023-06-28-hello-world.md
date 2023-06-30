---
layout: post
title:  "hello, world!"
date:   2023-06-28 19:45:12 -0300
categories: updates
---

Not much to say for this one except for: Hey there!

As I've described elsewhere in this site, this blog will serve as a space for me to share my explorations and
thoughts concerning programming, both in and out of work. I might insert an article on a completely unrelated
thing every now and then, but no promises.

Of course, since this post is called `hello, world!`, I'm just going to leave an Elixir implementation of the
program:

```elixir
defmodule HelloWorld do
  def hello_world, do: IO.puts("Hello, World!")
end
```
