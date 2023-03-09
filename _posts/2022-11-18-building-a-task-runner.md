---
title: Building a Task Runner In Python
date: 2022-11-18 12:12:12 +/-0800
categories: [Reinventing The Wheel, Python]
tags: [python] # TAG names should always be lowercase
author: adham
---

In this article, I will talk about my most recent project, "Yasta".

Yasta is a modern task runner written in Python. 🚐

Yasta makes running and managing your tasks a breeze! 🌬️

Even though it's written in Python, you can use it for any kind of project, or no projects at all!

I use it to automate some non-programming-related tasks on my machine.

For this project, I used [Typer](https://typer.tiangolo.com) to build the CLI.

Yasta has 5 commands:

- init (Initializes the projects by creating a toml file, named pyproject.toml by default, and adds an empty test command)

- show (Shows the list of tasks and their commands)

- add (Adds a task to the task list)

- run (Runs the specified task)

- delete (deletes the specified task)

### run

"run" is probably the most interesting command.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668798012296/83D2NW007.png)

#### path

The path option specifies the path for the tasks file. This is useful if your file is not named "pyproject.toml" or is in another directory.

The capture-output specifies whether you want to wait for the task to finish or not.
For example, if one of your tasks is running takes a bit of time, and logs some information, then capturing output will wait for the process to end to show, which might not be what you want, in that case, you shouldn't use this option. However, if your tasks are short, you should use this option, which will capture the output, print its status in pretty colors, and show a table of the commands and the corresponding errors (if any!).

You can see the difference in the following image.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668801040534/I-eakNAD2.png)

#### force

The force option is useful when you're running a list of tasks and you don't want a failing task to stop the rest of the tasks from running.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668801669950/bs1HYyWIo.png)

#### parallel

The parallel option runs a task that consists of several tasks, in parallel.

This is useful if you're running a task that runs 2 web servers, if you don't use the option, in this case, the second web server will be waiting until the first one ends.

That's it! The code base is quite small actually, but it serves all my needs pretty well.

Source code: https://github.com/adhamsalama/yasta

It was a delight using Typer to create this CLI. I would recommend it if you're writing a CLI in Python.
