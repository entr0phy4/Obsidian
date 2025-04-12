# What's a Operative System?
An operating system is the program that, after being initially loaded into the computer by a boot program, managers all the other application programs in a computer. Users can interact directly with the operating system through a user interface (UI), such as a command-line interface (CLI) or a graphical user interface (GUI).

An operating system brings powerful benefits to computer software and software development. Without and operating system, which is system software specifically designed to run the computer, every application would need to include its own UI, as well as the comprehensive code needed to handle all low-level functionality of the underlying computer's system software, such as disk storage and network interfaces. Considering the vast array of underlying [[Hardware]] available, and the number of software routines that must be run at the system software level to support computer functions, this would bloat the size of every application and make software development impractical.

The OS provides a consistent and repeatable way for applications to interact with the hardware and other system-level functions without the applications needing to know any details about them.

At simplest level, an operating system does two things:
- It manages the hardware and software resources of the system, In computers, tables smartphones these resources include the processors, memory, disk space and more.
- It provides a stable, consistent way for applications to deal with the hardware without having to know all the details of the hardware.

## Types of operative systems.
Within the broad family of operating systems, there are several types, categorized based on the types of computers they control and the sort of applications they support. The categories are:

- **Real-time operative system (RTOS)**: Are used to control machinery, scientific instruments and industrial systems. RTOS typically has very little user-interface capability, and no end-user utilities since the system will be a "sealed box", when delivered for use. *This type of OS is a specialized in handle time-critical tasks.* In RTOS, tasks are divided into two types: 
	- *Real-time tasks*: have specified deadlines and must be completed within those deadlines to ensure the proper functioning of the system.
	- *Non-real-time tasks*: Do not have strict timing requirements and can be executed when system resources are available.
  Priority-bases scheduling is a fundamental mechanism in RTOS that determines the order in which tasks are executed. Each task is assigned a priority level, and the scheduler ensures that higher-priority tasks preempt lower-priority tasks when necessary. This allows for the precise execution of critical tasks without being delayed by less important ones.

				*expand this*

- **Single-user, single task**: As the name implies, this operating system is designed to manage the computer so that one user can effectively do one thing at a time. [[MS-DOS]] is a good example of a single-user, single-task operating system.
- **Single-user, multitasking**: this is the type of operating system most people use on their desktop and laptop computers today. Microsoft's Windows and Apple's MacOS platforms are both examples of operating of operating systems that lets a single user have several applications in operation at the same time. For example, it's entirely possible for a Windows user to be writing a note in a word processor while downloading a file from the internet and printing the text of an email message.
- **Multi-user**: Allows many different users to take advantage of a computer's resources simultaneously. The operating system must make sure that the requirement of the various users are balanced, and that each of the programs they are using has sufficient and separate resources, s that a problem with one user doesn't affect the entire community of users. Unis, VM's and mainframe operating systems, such as MVS, are examples of multi-user operating system.

				*expand this.*

### [[Kernel]]