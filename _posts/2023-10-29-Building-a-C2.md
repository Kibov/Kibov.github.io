---
title: Building a C2 Part 1
categories: [Programming, C2]
tags: [programming, malware, c2, python, c/c++]     # TAG names should always be lowercase
---

## Why build a C2?
I stumbled upon a video [Building a C2 - Jim Maskelony](https://www.youtube.com/watch?v=fn6Vz0OcoK8), which led to this project. I wanted to learn more about internals of C2 and not get too bored only reading source code of other implementations. Also my Engineering thesis was coming up and I needed to select a topic, so this was a nice way to fill two needs with one deed.

**This project is not meant for real engagments.**

## Intro
This is still a work in progress, I'm learning as I do it, so if you have any suggestions or just see bugs in my code, please reach out to me.

You can see the source code here: [Fern C2](https://github.com/Kibov/C2)

## Components of the Fern C2

The complete C2 framework will have the following components:
* Server
* Agent
* Operator CLI/GUI client

### C2 server
The server and it's listeners will be built using python using the Flask framework, a simple API will be used to handle agent connections. SQLite database will be used to store data about agents, tasks and results.

Functionalities:

* Handle agents
* Receive and send tasks
* Encryption of communication
* Data storage and logging
* Stop and start HTTP and DNS(?) Listeners

### C2 agent/implant/client
C++ was chosen for the implant development for its performance, control and due to the author familiarity with it. As for the libraries I will try to only use windows api and c++ standard library.

Functionalities:
* Task download and execution
* Sending results
* Persistance
* Anti-debug
* Stageless

### Operator CLI/GUI
Allows for interaction with our server via REST API, written in python.

Functionalities:

* Add tasks for agents
* Display tasks results
* Display agents
* Stop agents


## Resources

Here are some resources that are found very helpful when I just started out and was researching. Also just check out open source code of other C2, lots of them to learn from.

- https://pre.empt.blog/2023/maelstrom-1-an-introduction
- https://0xrick.github.io/misc/c2
- https://shogunlab.gitbook.io/building-c2-implants-in-cpp-a-primer/