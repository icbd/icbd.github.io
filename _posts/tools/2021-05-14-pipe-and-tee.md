---
layout: post
title:  pipe and tee
Author: CBD
tags:   [Tools]
---

tee command reads the standard input and writes it to both the standard output and one or more files.

```text
$ echo "Hello" | tee fileA fileB fileC

+---------+   +-----+   +--------+
| Command +-->| tee +-->| Stdout |
+---------+   +--+--+   +--------+
                 |
                 |
                 |      +-------+
                 +----->| FileA |
                 |      +-------+
                 |
                 |      +-------+
                 +----->| FileB |
                 |      +-------+
                 |
                 |      +-------+
                 +----->| FileC |
                        +-------+
```

```text
          || visible in terminal ||   visible in file   || existing
  Syntax  ||  StdOut  |  StdErr  ||  StdOut  |  StdErr  ||   file   
==========++==========+==========++==========+==========++===========
    >     ||    no    |   yes    ||   yes    |    no    || overwrite
    >>    ||    no    |   yes    ||   yes    |    no    ||  append
          ||          |          ||          |          ||
   2>     ||   yes    |    no    ||    no    |   yes    || overwrite
   2>>    ||   yes    |    no    ||    no    |   yes    ||  append
          ||          |          ||          |          ||
   &>     ||    no    |    no    ||   yes    |   yes    || overwrite
   &>>    ||    no    |    no    ||   yes    |   yes    ||  append
          ||          |          ||          |          ||
 | tee    ||   yes    |   yes    ||   yes    |    no    || overwrite
 | tee -a ||   yes    |   yes    ||   yes    |    no    ||  append
          ||          |          ||          |          ||
 n.e. (*) ||   yes    |   yes    ||    no    |   yes    || overwrite
 n.e. (*) ||   yes    |   yes    ||    no    |   yes    ||  append
          ||          |          ||          |          ||
|& tee    ||   yes    |   yes    ||   yes    |   yes    || overwrite
|& tee -a ||   yes    |   yes    ||   yes    |   yes    ||  append
```

> [https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file](https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file)
