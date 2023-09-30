# Windows-Internals
In this repo, I'll talk about windows internals &lt;3

## Part 1 :
### 01. Introduction
### Contents
1. Course Objectives
2. Windows Versions
3. Tools
4. Summary

---

#### Course Objectives
- Understand windows features and architecture
- Uncover internal mechanisms relevant for developers
- Enhance ability to write better software for Windows

---
  
#### Windows Versions
- Windows NT 3.1 (July 1993)
- Windows NT 3.5 (September 1994)
- Windows NT 3.51 (May 1995)
- Windows NT 4.0 (July 1996)
- Windows 2000 (December 1999)
- Windows XP (August 2001)
- Windows Server 2003 (March 2003) Windows Vista (January 2007)
- Windows Server 2008 (February 2008)
- Windows 7 & 2008 R2 (October 2009)
- Windows 8 & Windows Server 2012 (October 2012)
- Windows 8.1 ("Blue") (expected August 2013)

---

#### Tools
1. Windows built in Tools :
- Task manager, resource monitor, performance monitor, others.
- Windows built in comes with the windows and installed with it.

2. Sysinternals :
- Obtained from http://www.sysinternals.com (which is redirected to
- http://microsoft.technet.com/sysinternals)
- Most written by Mark Russinovich
- No installation needed
- Free
- Sysinternals -> A software suit that was built by Mark Russinovich and some other developers, 
Sysinternals are free to download, And no installation is needed.

3. Debugging tools for Windows
- Now part of the Windows SDK
- No installation needed
- Free
- Debugging tools for windows part of windows sdk and it follows the same pattern as Sysinternals.

---

## I'll try to explain below how to get tools:
- Windows 8 SDK -> https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/
- ![Windows 8 SDK](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/61740c16-d4b7-4e2d-9551-c619372ff73a)

- Sysinternals -> https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
![Sysinternals](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/4ed26130-e1c1-4058-8de5-7a04a0989c2b)

---

#### Summary
- Windows has maintained roughly the same architecture since the first Windows NT version
- Various tools will be used throughout the course to demonstrate Windows features and behaviors

---

### 02. Basic Concepts
### Contents
1. User mode vs. Kernel mode
2. Processes
3. Threads
4. Virtual memory
5. Objects and handles
6. Summary

---

#### User mode vs. Kernel mode
- Whenever code executes, OS has one mode associated with it which may be user mode or kernel mode, so that associated with a thread. 
##### User mode vs. kernel mode
1. User mode
- Less powerful mode that doens't allow access to operating system code or data.
- No access to the hardware.
- Any exception that is unhandled causes the only running process to crash.
- Protects user applications from crashing the system(It will crash only the running process, nothing else).

---

2. Kernel mode
- Privileged mode for use by the kernel and device drivers only
- Allows access to all system resources(Memory location, HW, anything...).
- Any exception that is unhandled causes the system to crash, that's known as Blue Screen Of Death(BSOD).
![BSOD](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/1925ee17-2c39-4685-b09b-c17197f14a85)

---

#### Processes
- ##### Process(Management obj.) :
- A set of resources used to execute a program, process doesn't run, threads run.
- A process is simply a manager.
- ##### A process consists of :
- A private virtual address space(where memory is allocated :")
- An executable program, referring to an image file on disk which contains the initial code and data to be executed.
- A table of handles to various kernel objects, for example -> If i'm openning a file, I'm use API such as a CreateFile function, if this CreateFile function is successful it returns a handle, that handle is simply a number that stored in a private table of that particuler process, that means that these handles cannot be used by another process.
- A security context (access token), used for security checks when accessing shared resources.
- One or more threads that execute code.

---

#### Threads
- #### Thread :
- Entity that is scheduled by the kernel to execute code.
#### A thread contains :
- The state of CPU registers(that are saved in thread's stack).
- Current access mode (user mode or kernel mode).
- Two stacks, one in user space and one in kernel space.
- A private storage area, called Thread Local Storage (TLS). I'll talk a little bit about TLS, U have to take a look <3
#### TLS is useful in multi-threaded programming for several reasons:
1. Isolation: Each thread can have its own independent variables or data without the need for explicit synchronization mechanisms like mutexes. This can lead to improved performance and reduced contention for shared resources.
   
2. Thread Safety: TLS can help ensure thread safety by eliminating the need for locks when accessing thread-specific data. This can simplify the code and reduce the risk of deadlocks and other synchronization-related issues.

3. Efficiency: Accessing thread-local storage is typically faster than accessing shared data, as it doesn't involve inter-thread communication or locking mechanisms.

- TLS is commonly used in programming languages and libraries that support multi-threading, such as C/C++ with the thread_local keyword or the pthread library for POSIX threads. In Java, thread-local storage can be implemented using the ThreadLocal class. Each thread can allocate and access its own instance of a variable stored in thread-local storage.
Here's a simple example in C++:
![TLS](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/10230b27-fa9d-4c5f-8a25-91dcabaeea7d)
- In this example, each thread has its own copy of thread_specific_variable, and changes made to it in one thread do not affect the value in the other thread.
- Thread-local storage is a valuable tool in multi-threaded programming for managing thread-specific data and can help improve the performance and reliability of concurrent applications.
#### Getting back to what thread contains :
- Optional security token.
- Optional message queue and Windows the thread creates(In this case it will be UI thread).
- A priority, used in thread scheduling. from 0 to 31 (31 -> the highest priority).
- A state: running, ready, waiting. Explaination below take a look :"
- Ready State-> When threads wants to run but currently can't run because old cores (old logical processors) are currently executing code for other threads, So if i have 8 logical cores then 8 threads can run immediately but the 9th thread has to wait, So it will be in the ready state(wants to run).
- Waiting State -> The thread doesn't want to run at all because it's waiting for something, this something can be I/O operation needs to return, it may be some kernel object the thread is waiting on before it can continue doing some work, In this case the thread doesn't consume any CPU cycles, When the thing that thread waiting upon finally arrives it moves to the ready state and hopefully to the running state as soon as possible depending on the other threads and the number of available logical processors(Cores).

#### Let's dig deeper into threads
- We can see thread information in Task Manager here is the performance tab, there are 204 processes in my system and 2776 threads, because my processor doesn't really do significant work, that means that most of these threads are in the waiting state.
![threadinfo](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/f245ec6b-823b-4565-b92d-dfa65d06b798)

- The details tab in Windows 11 task manager can show us the number of threads for each process. but nothing really more than that.
![no threads](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/8342d0fd-e96a-4e04-a398-56c6fd583b24)

#### To show more information we have to go to process explorer
- In process explorer we can view each process's threads using the properties window, let's select svchost.exe for example and right click and choose properties.
![px](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/a73cd422-c114-4f66-aacb-8e8435f1921d)

- And here can see the properties window of this particuler process(svchost.exe).
![prp](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/5217a05b-a93c-4d65-af90-e7656e3247a7)

- The image tab shows some general information about the process such as its path, command line that started with, parent process, user that running this process and some other information, But there is a threads tab here that show all the threads that are currently in this particuler process.
- For each thread we cann see several pieces of information the ThreadID, CPU consumption of this particuler thread and the number of cycles it accumelated since the last time this cycles were calculated and finally there is a start address.
- The start address shows what is the address that the thread starts running with.

>[!NOTE]
>The ThreadIDs(TIDs) and ProcessIDs(PIDs) are coming from the same pool

![thr](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/42e421f6-80c0-4deb-b610-436c36879228)

- If we select a particuler thread we can even look at it's call stack, the call stack is so much interesting because it shows all the call stack from the user mode into kernel mode
![utok](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/09b2f640-fe73-47ee-9c0e-b5c18cafe381)

- At the bottom here we can see more attributes of the thread such as its state, the amount of time it spent in kernel mode and user mode, the number of context switches that exists for this particuler thread, number of cycles it totally ran that CPU cycles, some information about priority(Base abd Dynamic) i'll talk about the difference in process and thread modules a bit later <3
![thr](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/42e421f6-80c0-4deb-b610-436c36879228)

![threadinf0](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/0f77b994-345a-49c5-aa32-e74c709b0278)

- U can also kill or suspend a thread which is usually a bad thing to do or perhaps a bit dangerous but if we have a thread that consumes a lot of CPU cycles we can suspend that.
![sus](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/82bf417d-e664-4a66-b273-af2dbc87dffd)

---

