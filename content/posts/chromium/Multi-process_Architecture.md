---
title: 'Chromium-1 多进程结构'
date: 2022-07-14T14:53:33+08:00
draft: false
---

# Multi-process Architecture 多进程结构

This document describes Chromium's high-level architecture and how it is divided among multiple process types.

这份文件描述了 Chromium 的高层架构，以及它是如何被划分为多个进程类型的

## Problem

It's nearly impossible to build a rendering engine that never crashes or hangs. It's also nearly impossible to build a rendering engine that is perfectly secure.

In some ways, the state of web browsers around 2006 was like that of the single-user, co-operatively multi-tasked operating systems of the past. As a misbehaving application in such an operating system could take down the entire system, so could a misbehaving web page in a web browser. All it took was one rendering engine or plug-in bug to bring down the entire browser and all of the currently running tabs.

Modern operating systems are more robust because they put applications into separate processes that are walled off from one another. A crash in one application generally does not impair other applications or the integrity of the operating system, and each user's access to other users' data is restricted. Chromium's architecture aims for this more robust design.

要建立一个永不崩溃或挂起的渲染引擎几乎是不可能的。也几乎不可能建立一个完全安全的渲染引擎。

在某些方面，2006 年左右网络浏览器的状况就像过去的单用户、合作式多任务操作系统一样。在这样的操作系统中，一个行为不端的应用程序可以使整个系统瘫痪，在网络浏览器中，一个行为不端的网页也可以使整个系统瘫痪。只需要一个渲染引擎或插件的错误，就可以使整个浏览器和当前运行的所有标签崩溃。

现代操作系统更加强大，因为它们将应用程序放入独立的进程中，相互隔离。一个应用程序的崩溃通常不会损害其他应用程序或操作系统的完整性，而且每个用户对其他用户的数据的访问也受到限制。Chromium 的架构旨在实现这种更强大的设计。

## Architectural overview

Chromium uses multiple processes to protect the overall application from bugs and glitches in the rendering engine or other components. It also restricts access from each rendering engine process to other processes and to the rest of the system. In some ways, this brings to web browsing the benefits that memory protection and access control brought to operating systems.

We refer to the main process that runs the UI and manages renderer and other processes as the "browser process" or "browser." Likewise, the processes that handle web content are called "renderer processes" or "renderers." The renderers use the Blink open-source layout engine for interpreting and laying out HTML.

Chromium 使用多个进程来保护整个应用程序免受渲染引擎或其他组件的错误和故障的影响。它还限制了每个渲染引擎进程对其他进程和系统其他部分的访问。在某些方面，这给网络浏览带来了内存保护和访问控制给操作系统带来的好处。

我们把运行用户界面、管理渲染器和其他进程的主进程称为 "浏览器进程 "或 "浏览器"。同样地，处理网页内容的进程被称为 "渲染器进程 "或 "渲染器"。呈现器使用 Blink 开源布局引擎来解释和铺设 HTML。

![arch](static/chromium/arch.png)

## Managing renderer processes

Each renderer process has a global RenderProcess object that manages communication with the parent browser process and maintains global state. The browser maintains a corresponding RenderProcessHost for each renderer process, which manages browser state and communication for the renderer. The browser and the renderers communicate using Mojo or Chromium's legacy IPC system.

每个呈现器进程都有一个全局的 RenderProcess 对象，用于管理与父浏览器进程的通信，并维护全局状态。浏览器为每个呈现器进程维护一个相应的 RenderProcessHost，它负责管理浏览器状态和呈现器的通信。浏览器和渲染器使用 Mojo 或 Chromium 的传统 IPC 系统进行通信。
