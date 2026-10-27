---
title: "How it Works"
linkTitle: "How it Works"
weight: 30
date: 2026-10-27
tags:
  - ai
description: >
  The technical side of Stroom's AI features: runs and the model's context, memory, workspaces, and how untrusted content is handled.
---

These pages explain how Ask Stroom AI and Stroom's MCP endpoint work underneath, for administrators and for users who want to know what the agent sees and keeps.

* [Runs and context]({{< relref "runs-and-context" >}}): how a question is worked on, what the model is sent each time, and how a long chat is made to fit.
* [Memory]({{< relref "memory" >}}): the user's and the chat's memory, how they are stored and loaded, and when a change needs approval.
* [Workspaces]({{< relref "workspaces" >}}): the files a chat or MCP session works with, where they are kept, their limits and how long they last.
* [Trust and untrusted content]({{< relref "trust" >}}): how Stroom tracks what could hold planted instructions, and what that changes.

Everything described here is held in Stroom's database, so any node can carry on with a chat.
