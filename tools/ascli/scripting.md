---
title: ascli – Scripting
description: Learn how ascli provides a scripting environment to create a stack, which gets populated with commands, in order to repeat the commands and gather statistics for those commands.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Aerospike CLI (ascli)
    url: /docs/v3/ascli.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use aql instead.
{{/note}}

**ascli** provides a very simple scripting environment called `eval`, which is able to execute a series of commands. The environment creates a stack, which gets populated with commands and gives you the ability to repeat the commands and gather simple statistics for those commands.

The benefit of `eval` is that it sets up a single client instance which is shared by all commands invoked in the environment. 

`eval` should not be used for performance testing because it only makes use of a single thread and all calls are blocking. 

### Usage

`eval` reads commands from stdin. Sending commands followed by a newline will execute the command.

```bash
ascli eval
```

A common usage pattern is to create a file of commands and piping it to ascli eval:

```bash
$ cat example.ascli | grep ascli eval
```

eval will invoke any ascli command. However, eval also has its own set of special commands.

### Commands

The following are eval's special commands:

Command | Description
--- | ---
`:print` | Display statistics from last command.
`:reset` | Reset accumulated statistics.
`:repeat <n>` | Repeat all commands on the stack n times.
`:clear` | Clear the stack.
`:quit` | Quit eval.

### Stack

Every command you type in eval is pushed onto a stack.  The stack handles up to 1000 commands, and each command can be at most 1000 characters.

Once you have one or more commands on the stack, you can repeat the commands using `:repeat`

If you want to clear the stack, use `:clear`

The `:print` command will print statistics since the start of the program or the last time `:reset` command was issued. The `:reset` command will reset the statistics.

### Example Run

The following is an example session:

```bash
$ ascli eval
put test users user1 '{"name":"John"}'
:print
>>
  time:    0.42700 ms
  success: 1
  failure: 0
:reset
:print
>>
  time:    0.00000 ms
  success: 0
  failure: 0
get test users user1
{"name": "John"}
:reset
:repeat 10
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
{"name": "John"}
:print
>>
  time:    3.08900 ms
  success: 20
  failure: 0
:quit
```
