---
layout: post
title: OTP 23 Highlights
tags: otp 23 release
author: Kenneth Lundin
---

OTP 23 has just been released (May 13:th 2020).
It has been a long process with three release
candidates in February, March and April before the final release.
We are very thankful to the feedback we have got from the release candidates
which has revealed some bugs and flaws that our internal testing did not find.

This blog post will describe some highlights of what is released in OTP 23.

You can download the readme describing the changes here:
[OTP 23 Readme](http://erlang.org/download/otp_src_23.0.readme).
Or, as always, look at the release notes of the application you are interested in.
For instance here: [OTP 23 Erts Release Notes](http://erlang.org/doc/apps/erts/notes.html).

# Language

In OTP 23 we have added some new features to the language and compiler, one has been in the backlog since the bit syntax was introduced and the other is a suggestion from the community.

## Matching syntax improvements​

### Binary matching
In binary matching, the size of the segment to be matched is now allowed to be a guard expression. In the example below the variable Size is bound to the first 8 bits and then it is used in an expression (Size-1)8 for the size of the following binary.
```erlang
example1(<<Size:8,Payload:((Size-1)*8)/binary,Rest/binary>>) ->​
    {Payload,Rest}.
```

### Matching on maps
In the current map matching syntax, the key in a map pattern must be a single value or a literal. This leads to unnatural code if the keys in a map are complex terms.​

With OTP 23 the keys in map matching can be guard expressions as you see in example2.​

The only limitation is that all variables used in a key expression must be previously bound. ​
```erlang
example2(M, X) ->
    Key = {tag,X},
    #{Key := Value} = M,
    Value.
```
Below there is an illegal example showing that it is still not supported to use an unbound variable as part of the expression for the key-pattern. In this case Key is not bound and the requirement is that all variables used in a key expression must be previously bound. 
```erlang
illegal_example(Key, #{Key := Value}) -> Value.
```

## Numeric literals with underscore
It is now allowed to write numeric literals with underscore between the digits for the purpose of readability. But the placement of the underscores is not totally free there are som rules. See example of allowed use below:

```
305441741123_456
1_2_3_4_5
123_456.789_123
1.0e1_23
16#DEAD_BEEF
2#1100_1010_0011
```
And in the following example we have some examples of dissalowed placement of the underscore:

```
_123  % variable name
123_
123__456  % only single ‘_’
123_.456
123._456
16#_1234
16#1234_
```
# Distributed spawn and the new erpc module
## Improved spawn
We have improved the scalability and performance of the spawn operation.

Also added new features, such a distributed `spawn_monitor()` BIF. This function creates a new process and sets up a monitor atomically.

The `spawn_opt()` BIF will support the monitor option for setting up a monitor atomically while creating a process.

We’ve added new `spawn_request()` BIFs for asynchronous spawning of processes.
`spawn_request()` supports all options that `spawn_opt()` already supports.

## New module erpc
The `erpc` module implements an enhanced subset of the operations provided by the rpc module.

Enhanced in the sense that it makes it possible to distinguish between returned value, raised exceptions, and other errors.​

`erpc` also has better performance and scalability than the original rpc implementation. This by utilizing the newly introduced `spawn_request()` BIF.

# gen_tcp and the new socket module

In OTP 22 comes we introduced the new experimental [socket](http://erlang.org/doc/man/socket.html) API.
The idea behind this API is to have a stable intermediary API that users can use
to create features that are not part of the higher-level gen APIs. We have now come one step further in our plan to replace the inet driver by making it possible to use the gen_tcp API with socket as an optional backend.
```
MORE TO WRITE

```
# Help in the shell
- We have implemented EEP 48. EEP 48 proposes an official API documentation storage to be used by BEAM languages. By standardizing how API documentation is stored, it will be possible to write tools that work across languages.

- We’ve extended the ordinary doc build with the generation .chunk files for all OTP modules.

- Built on these new features we’ve added on-line help in the shell.

- We’ve added a new module shell_docs with functions for rendering documentation for a shell. This can be used for instance by LSP implementations.

- The code module also got a new function, get_doc which returns the doc chunk without loading the module.

EEP 48: Documentation storage and format

Doc build generates .chunk files for all OTP

On-line help in the shell
h(Module), h(Module,Function), h(Module,Function,Arity)

New module shell_docs in stdlib with functions `render/1,2,3`

New function code:get_doc(Module) which returns the doc chunk without loading the module.
```
4> h(lists,sort,2).

  -spec sort(Fun, List1) -> List2
                when
                    Fun :: fun((A :: T, B :: T) -> boolean()),
                    List1 :: [T],
                    List2 :: [T],
                    T :: term().

  Returns a list containing the sorted elements of List1,
  according to the ordering function Fun. Fun(A, B) is to
  return true if A compares less than or equal to B in the
  ordering, otherwise false.
ok
```
## Improved tab-completion
The tab-completion in the shell is also improved. Previously the tab-completion for modules did only work for already loaded modules now this is extended to work for all modules available in the code path. The completion is also extended to work inside the "help" functions h, ht and hc. You can for example press tab like the example below and get all modules beginning with `l`:
```
5> h(l
lcnt                      leex                      lists                     
local_tcp                 local_udp                 log_mf_h                  
logger                    logger_backend            logger_config             
logger_disk_log_h         logger_filters            logger_formatter          
logger_h_common           logger_handler_watcher    logger_olp                
logger_proxy              logger_server             logger_simple_h           
logger_std_h              
logger_sup
```
Or complete all functions beginning with `s` in the `lists`module like this:
```
5> h(lists,s
search/2     seq/2        seq/3        sort/1       sort/2       split/2      
splitwith/2  sublist/2    sublist/3    subtract/2   suffix/2     sum/1        
```

# "Container friendly" features

## Take CPU quotas into account
CPU quotas are now taken into account when deciding the default number of online schedulers.

Thus, improving performance in container environments where quotas are applied, such as docker with the `--cpus` flag.

## EPMD independence
We have improved the handshake during connection setup in the Erlang distribution protocol.
It is now possible to agree on protocol version without depending on epmd or other prior knowledge of peer node version.
(makes it independent on EPMD)
The possibility to run Erlang distribution without relying on EPMD has been extended. To achieve this a couple of new options to the inet distribution has been added.

- `-dist_listen false` Setup the distribution channel, but do not listen for incoming connection. This is useful when you want to use the current node to interact with another node on the same machine without it joining the entire cluster.

- `-erl_epmd_port Port` Configure a default port that the built-in EPMD client should return. This allows the local node to know the port to connect to for any other node in the cluster.

The `erl_epmd` callback API has also been extended to allow returning -1 as the creation which means that a random creation will be created by the node.

In addition a new callback function called
listen_port_please has been added that allows the callback to return which listen port the distribution should use. This can be used instead of `inet_dist_listen_min/max` if the listen port is to be fetched from an external service.

## New option for `erl_call`
`erl_call` got a new `address` option, that can be used to connect directly to a node  without being dependent on epmd to resolve the node name.

```
MORE TO WRITE



```
# TLS enhancements and changes
The SSL application got new TLS 1.3 features.

Also we have removed support for SSLv3!

# SSH
Two notable SSH features were provided as Pull Requests from open source users, namely support for fetching keys from ssh-agents and TCP/IP port forwarding.  Port forwarding is sometimes called tunneling or tcp-forward/direct-tcp. In the OpenSSH client, port forwarding corresponds to the options -L and -R.

Ssh agent stored keys improves the security while port forwarding is often used to get an encrypted tunnel between two hosts. In the area of key handling, the default key plugin ssh_file.erl is rewritten and extended with OpenSSH file format "openssh-key-v1".  A limitation so far is that keys in the new format can't be encrypted The default plugin now also uses port numbers which increases the security.

The SSH application can now be configured in an Erlang config-file. This gives the possibility to for example change the supported algorithm set without code change.


# CRYPTO
A new crypto API was introduced in OTP-22.0.  The main reason for a new API was to use the OpenSSL libcrypto EVP API that enables HW acceleration, if the machine supports it.  The naming of crypto algorithms is also systemized and now follows the schema in OpenSSL.

There are parts of the CRYPTO app that are using very old APIs while other parts are using the latest one.
It turned out that using the old API in the new way, and still keeping it backwards compatible, was not possible.

Therefore the old api is kept for now but it is implemented with new primitives.
The Old API is deprecated in OTP-23.0 and will be removed in OTP-24.0.
