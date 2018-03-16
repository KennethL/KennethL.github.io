---
layout: post
title: Blog post attempt
---

Every week I will try to make a new blog post about something interesting regarding Erlang/OTP.

Today the 16:th of March the CodeBeam conference in San Francisco is taking place with many interesting talks.
There is one talk about the "Latest News from OTP" and another about `gen_statem` the state machine behavior
replacing the old `gen_fsm`.

An Erlang example:
```erlang
-module(mymod).

-compile(export_all).

foo(A, B) when is_atom(A) ->
  {atom_to_list(A),B}.
```

## ToDo List
- [x] Test to make a Blog Post
- [ ] Write a real interesting Blog Post
