---
title: Introducing pdmreader
date: 2018-09-02T00:26:25+08:00
tags: ["powerdesigner", "pdmreader", "python"]
draft: false
---

[PowerDesigner](https://www.sap.com/products/powerdesigner-data-modeling-tools.html) is a powerful and popular data modeling tool, but it’s too expensive to afford. As a developer, I have to consume the PDM artifacts created by my colleagues, so switching to another tool is not a solution.

Fortunately, PowerDesigner supports plain XML file format, which allows the possibility of reading and writing without PowerDesigner. This is [officially documented](http://infocenter.sybase.com/help/topic/com.sybase.infocenter.dc38628.1650/doc/html/rad1232022084031.html).

Although you can read the PDM XML file with any text editor, it will be too annoying to work directly with XML, and text editor won’t make you any help in generating DDL.

So I create a simple command line tool: [pdmreader](https://github.com/bianjp/pdmreader). 

<!--more-->

It has the following features:

* Show tables list
* Show table definition
* Generate DDL for various databases (currently only MySQL and Oracle supported)
* Generate Java entity definition (useful as I’m a Java developer)

pdmreader is written in Python 3.7 (the exciting [dataclass](https://docs.python.org/3/library/dataclasses.html) feature added in 3.7 is used to simplify the code), and has no other dependencies.

Instead of executing a command and exit, pdmreader creates an interactive "shell". You can then type a few custom commands for different purposes. This avoids the overhead of parsing XML file every time you type a command.

This tool is quite simple and easy to customize. Tweak yourself if it fails to fit your own needs.
