Think of a script for a play, or a movie, or a TV show. The script tells the actors what they should say and do. A script for a computer tells the computer what it should do or say. In the context of Bash script we are telling the Bash shell what it should do.

A Bash script is a plain text file which contains a series of commands. These commands are a mixture of commands we would normally type ourselves on the command line (such as ls or cp for example) and commands we could type on the command line but generally wouldn't.

**How do they works?**
In the real of Linux (and computers in general) we have the concept of programs and processes. A program is a blob of binary data consisting of a series of instructions for the CPU and possibly other resources (images, sound files and such) organised into a package and typically stored on your hard disk. When we say we are running a program we are not really running the program but a copy of it which is called a process. What we do is copy those instructions and resources from the hard disk into working memory (or RAM). We also allocate a bit of space in RAM for the process to store variables (to hold temporary working data) and a few flags to allow the operating system (OS) to manage and tack the process during it's execution.

*Essentially a process is a running instance of a program*.

### The Shebang (**#!**)

`#!/bin/bash`

This is the first line of the script above. The hash exclamation mark ( #! ) character sequence is referred to as the Shebang. Following it is the path to the interpreter (or program) that should be used to run (or interpret) the rest of the lines in the text file. (For Bash script it will be the path to Bash, but there are many other types of scripts and they each have their own interpreter.)

Formatting is important here. The shebang must be on the very first line of the file (line 2 won't do, even if the first line is blank). There must also be no spaces before the # or between the ! and the path to the interpreter.

### Variables
Think of a variable as a temporary store for a simple piece of information. These variables can be very useful for allowing us to manage and control the actions of our Bash Scripts.

A Variable is a temporary store for a piece of information. There are two actions we may perform for variables:
- Setting a value for a variable.
- Reading the value for a variable.


### [[OverTheWire]]
### [[Command-line utilities]]