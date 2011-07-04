    Author: Yurii Rashkovskii <yrashk@gmail.com>
    Status: Draft
    Type: Standards Track
    Erlang-Version: R15A
    Created: 2-Jul-2011
    Post-History:
****
EEP XXX: Scoped Group Leaders
----



Abstract
========

Scoped group leaders is an extension of an existing group leader 
mechanism that allows to use group leaders beyond their 
original intent (I/O only).

Specification
=============

Every process gets a "dictionary" of group leaders, with one group 
leader `io` defined by default that represents the only one group 
leader that exists in current implementation of Erlang.  On every
new process creation the entire dictionary is copied over just
like a single group leader was copied before.

Two new functions were added to the `erlang` module.

First one, scoped group leader retrieval:

    erlang:group_leader(Scope :: atom()) -> 'undefined' | pid()

This function retrieves group leader for scope `Scope`. Existing 
argumentless function `erlang:group_leader/0` is now implemented as

    erlang:group_leader(io)

Second one, scoped group leader setup:

    erlang:group_leader(Scope :: atom(), GroupLeader :: pid(), 
                                         Proc :: pid()) -> true.

This function sets group leader for scope `Scope` to `GroupLeader` 
for a process `Proc`.  Existing function `erlang:group_leader/2` is 
now implemented as

    erlang:group_leader(io, GroupLeader, Proc)

Process information available through `erlang:process_info/1` and
`erlang:process_info/2` has been extended with a new key, `group_leaders`.
It will contain a proplist of group leaders associated with the process. 
Note that it will always contain at least one tuple like `{io, <0.24.0>}`
as this group leader is present in every single process.

Distribution mechanism gets extended to support these scoped group leaders 
as well, so processed spawned on remote nodes get the whole list of group
leaders copied.

Example
-------

In this example, we'll set a group leader for the scope of `test`
and will retrieve it from the current and child processes.  We'll 
also retrieve `io` scoped group leader using both original and new
API:

    1> erlang:group_leader(test, self(), self()).
    true
    2>  erlang:group_leader().
    <0.24.0>
    3> erlang:group_leader(io).
    <0.24.0>
    4> erlang:group_leader(test).
    <0.31.0>
    5> spawn(fun() -> io:format("~p~n",[erlang:group_leader()]) end), ok.
    <0.24.0>
    ok
    6> spawn(fun() -> io:format("~p~n",[erlang:group_leader(io)]) end), ok.
    <0.24.0>
    ok
    7> spawn(fun() -> io:format("~p~n",[erlang:group_leader(test)]) end), ok.
    <0.31.0>
    ok
    8> spawn(fun() -> io:format("~p~n",[erlang:process_info(self())]) end), ok.
    [{current_function,{erl_eval,do_apply,5}},
     {initial_call,{erlang,apply,2}},
     {status,running},
     {message_queue_len,0},
     {messages,[]},
     {links,[]},
     {dictionary,[]},
     {trap_exit,false},
     {error_handler,error_handler},
     {priority,normal},
     {group_leader,<0.24.0>},
     {group_leaders,[{test,<0.31.0>},{io,<0.24.0>}]},
     {total_heap_size,233},
     {heap_size,233},
     {stack_size,24},
     {reductions,93},
     {garbage_collection,[{min_bin_vheap_size,46368},
                          {min_heap_size,233},
                          {fullsweep_after,65535},
                          {minor_gcs,0}]},
     {suspending,[]}]
    ok


Motivation
==========

I/O system is not the only domain where a concept of group leaders
comes handy.  Implicit configurations, security groups and many other
problems would benefit from being able to use this generalized group
leader mechanism.

One of the potential uses of this technique could be an extension of
I/O leader paradigm into web development, with a `web` group leader
representing an HTTP connection, WebSocket or a session.  With this 
approach one can use the same technique used by I/O primitives to allow
transparent access to HTTP communication channels, within either local
or remote processes.


Rationale
=========

We have chosen to extend existing API instead of introducing a new one
simply because this concept is a natural evolution of group leaders with
which people are already familiar.  


Backwards Compatibility
=======================

This proposed change keeps existing API intact and only provides new
functions for this new described functionality, therefore making this change 
backwards compatible.  Existing behaviours are not altered and newly 
introduced behaviours are built to mimick existing ones.

Proplist returned by `erlang:process_info/1` has all pre-existing keys
unmodified and features a new key called `group_leaders`.  Unless some code
relied on a specific set of keys used in this proplist, no backwards compatibility
issues should exist in this regard.


Reference Implementation
========================

There is no reference implementation at this point.  However, a proof 
of concept implementation is [available][1].

[1]: https://github.com/spawngrid/otp/tree/group_leader_scope


Copyright
=========

This document has been placed in the public domain.



[EmacsVar]: <> "Local Variables:"
[EmacsVar]: <> "mode: indented-text"
[EmacsVar]: <> "indent-tabs-mode: nil"
[EmacsVar]: <> "sentence-end-double-space: t"
[EmacsVar]: <> "fill-column: 70"
[EmacsVar]: <> "coding: utf-8"
[EmacsVar]: <> "End:"
