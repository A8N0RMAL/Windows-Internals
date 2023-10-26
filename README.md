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

#### Virtual Memory
- #### Virtual Memory :
- Each process "sees" a flat linear memory.
- Internally, virtual memory may be mapped to physical memory, but may also be stored on disk.
- Processes access memory regardless of where it actually resides.
- The memory manager handles mapping of virtual to physical pages.
- Processes cannot (and need not) know the actual physical address of a given address in virtual memory.

![vmm](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/cb927cbb-0f55-45e5-9b81-dbc209713a05)

![vml](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/9d6cfee8-0527-4d90-b5ab-bb4dda3e5990)

#### Let's dig deeper into virtual memory
- There are various tools that can show us memory information, for example task manager has a memory column here which is shown by default and that memory is a private working set(it's a term used to describe physical memory), so this column indicates the amount of physical memory that is used by the process as private memory(memory that is not shared with other processes)

![phymemory](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/298a2843-fa92-4fc4-b0d9-82111437c2a1)

- Let's look at the whole working set by right clicking and choose select columns and choose Memory(shared working set) and commit size to get how much memory a process consumes.

![wholeworkingset](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/fb4dd913-95bc-4192-95cc-373814415dd6)

![wholeworkingset0](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/2222964b-59fa-4e41-abce-35acee5796a3)

- The other tool we can use is process explorer because it has some columns we can use to gain some more information about memory uasge.
- Let's open process explorer and right clicking in processes and select columns.

![prcexp](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/141d6175-a5a9-492a-950f-68dc295da021)

- There is a process memory tab here which shows which shows various counters most of them is actually are available from performance monitor.
- We can see private bytes which is the same as commit size in task manager.
- So this may be kind of confusing, we'll take a deeper look at these counters in the memory management module.

![prcmem](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/9365da26-771f-43b5-8d1e-841eb9f9e639)

- Another tool we can use which is part of sysinternals tools is called VMMap, i'm not gonna show u all the functionality of VMMap but once we rrun that we need first to select a process.
![VMMap](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/9580cc5a-274e-4bba-8cc4-d53fdeb005c2)

- Let's select a explorer.exe process, and what this tool does it shows us the exact layout of the virtual address space of the particuler process, so we can see right here addresses which are rising up here to the 8TB limit of 64bit processes.
![VMMap0](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/97d002d3-15dd-48ac-a193-a0f00578bdce)
![VMMap1](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/957e0815-39dd-4d67-958d-1abb371c12c6)

> [!NOTE]
> Addresses are 2GB limit for 32bit processes and 8TB limit for 64bit processes.

- In process explorer we can see in Lower Pane View DLLs, Handles and Threads for each process.
![lpv](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/e97c2134-eeff-4d64-93c6-a94ebd10d7d7)
![lpv0](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/5ba8a321-e084-4de5-a53d-51b2d3be5d6e)

- We can also look at the System process which represents the kernel address space, so the modules here are actually drivers and the kernel itself.
- We can see a lot of sys files which are drivers, Hardware Abstraction Layer(HAL) and the Kernel itself(ntoskrnl.exe)
![sysspace](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/5f16bf59-a0a4-4512-b5cf-60ca3df7c197)

---

#### Objects and Handles
- Windows is an object based system, that means the kernel exposesand manages objects and provide some API to manipulate those objects, so objects can be created dynamically depending on what we wanna do.
- Objects are run time instances of static structures. Examples -> process, mutex, event, desktop, file, thread. there are some of object types that the kernel provides.
- These objects reside in system memory space(that means they can only be access directly by kernel code).
- Kernel code can obtain direct pointer to an object and manipulate that instance technically bypass the API that provided for that particuler kind of object.
- User mode can only obtain a handle to an object(cannot obtain it directly that's because system space is inaccessable in user mode, only the user address space is accessable).
- A handle is an index in some table that points to a particuler object in kernel space.
- A handle shields a user code from directly accessing an object because an object structure may change between OS versions, the API may change as well, but user code doesn't really notice because it uses a handle and indirect reference to that object.
- Objects are reference counted.
- The Object Manager is the entity resposible for creating, obtaining and otherwise manipulating objects.
#### Let's dig deeper into objects and handles
- We can see handle information in process explorer by selecting a process and looking at the lower pane with the handle's view selected.
![handle](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/f091ed12-e4c8-4777-915d-daf0860c5f27)

- Every process has its handle table(the handle table is always private to a particuler process).
- I'm gonna show handle value, access mask and object addres.
![handle0](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/17c2372a-493d-4a38-a338-da6f77662052)

- The object address is the actual address in kernel space where the real object resides.
- So this is must be a pointer to a kernel space.
![handle1](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/604e3934-d3d8-4b65-bbeb-0b2aa56543f1)

- The access mask here is a set of flags that indicates what can be done with this particuler handle.
![handle2](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/18d0c3fb-8ff1-4432-8e3b-ede1f5f2fd15)

- Let's see what we can do with this handles, first thing we can do is to let process explorer provide us some more information about the handle.
![handle3](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/ce7d3938-9e50-442c-8571-8db8aff5b4f1)

- We can see the name here more clearly, some description, the address, number of references and the handles to that particuler object.
![handle4](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/6479e9b8-38f7-4ae3-ac44-5657ecc97459)

- For the event we can see some special event information such as the type of the event which can be synchrinization or notification, these are the kernel terms for autoresetevent and manualresetevent that are perhaps more familier to user mode developers and the state for that object which currently is not signeled which means the flag that event represent is currently down.
![handle5](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/0951082f-3658-4222-a8e6-a9c7de9fe227)

- Let's get our hand dirty, Now i'll open WindowsMediaPlayer as shown here in process explorer.
![handle6](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/1d9345a1-fc87-421a-bd0d-21d4bc3ce9fc)

- Now if i try to execute another WindowsMediaPlayer it simply won't work, the reason for that is when WindowsMediaPlayer comes up it looks weather it's the first WindowsMediaPlayer or not, if it is the first it runs as usual but if it is not it simply communicates to the other existing WindowsMediaPlayer instance and simply shuts down(this process is so quick and u can't see the second WindowsMediaPlayer appearing in process explorer).
- We can trick the second WindowsMediaPlayer to think that it's the first instance even though it's not.
- If we look through the handle table of the WindowsMediaPlayer there is a handle here to a MUTEX called Mutant, that's named Microsoft_WMP_70_CheckForOtherInstanceMutex, it's kind of a hint :"
![handle7](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/b4197a24-0039-4d69-9ea8-b3453d452f90)

- So let me right click and close that handle now process explorer warn that closing a handle is very dangerous because usually this can cause the target process to lose some important information or be unable to perfrom its work, but in this case i'm gonna allow this to happen.
![handle8](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/179e33ec-3c61-46bf-927e-8f39d67830a3)
![handle9](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/ee9dcc93-dd16-4a9b-b0e7-e69ebbe76fa9)

- Now we can see the handle turning red and goes await.
![handle10](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/4a95329b-0e47-4195-bd16-d8caebe2ae86)

- Now if i try to execute another WindowsMediaPlayer, i'm able to do that <3.
![handle11](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/402e6e8e-e9b9-4480-acac-0bda3cfbfb22)

#### Summary
- A process is a management container for threads to execute code.
- A Thread executes code on a CPU.
- Multiple threads can execute concurrently on multiple CPUs.
- Per process virtual memory provides a private address space isolated from other processes.
- Kernel objects are accessed from user mode using private process handles.

---

### 03. System Architecture (Part 1)
### Contents
1. Windows design goals.
2. Windows editions.
3. General architecture overview.
4. Function call flow.
5. Summary.

---

#### Windows Design Goals
##### Separate address space per process
- One process cannot (easily) corrupt another's memory.
##### Protected kernel
- User mode applications cannot crash kernel.
##### Preemptive multitasking and multithreading Multiprocessing support
##### Internationalization support using Unicode Security throughout the system
##### Integrated networking
##### Powerful file system (NTFS)
- Supports protection, compression and encryption
##### Run most 16 bit Windows and DOS apps On 32 bit systems
##### Run POSIX 1003.1 and OS/2 applications
#####  Portable across processors and platforms Be a great client as well as server platform

---

### Windows Editions
##### Windows XP Home
- Designed as a replacement for the Windows 9x/ME family (“Consumer Windows”).
##### Windows Professional (2000, XP, Vista, 7, 8)
- Main desktop (client) OS.
##### Windows Server Standard, Advanced, Datacenter editions (Windows 2000, 2003/R2, 2008/R2, 2012)
- Server platforms.
##### Other variants
- XP starter, XP Home, Media center, Server Web Edition, Home, Premium, Ultimate, Business, Enterprise.
#### Professional vs. Server
- Same core system files
##### Differences
- Number of processors supported.
- Maximum amount of RAM than can be used.
- Maximum of concurrent network connections supported for file and print sharing.
- Some services only appear in Server versions.
- Other system policies and default settings (e.g. thread quantum).
##### OS type can be discovered by calling GetVersionEx (Win32) or RtlGetVersion(WDK)
#### Windows Numeric Versions
- Windows NT 4 (4.0)
- Windows 2000 (5.0)
- Windows XP (5.1)
- Windows Server 2003, 2003 R2 (5.2)
- Windows Vista, Server 2008 (6.0)
- Windows 7, Server 2008 R2 (6.1)
- Windows 8, Server 2012 (6.2)
- Windows 8.1, Server 2012 R2 (6.3)
##### These values can be obtained using GetVersionEx (Win32) or RtlGetVersion (WDK)

---

### General architecture overview
![GAO](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/9f8d86f4-e928-4fd9-aabd-4ab35c461ec9)

---

### Function call flow
![FCF](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/3feaf274-dff4-41ea-9166-a72e2a08e628)

---

### Breif Overview of WinDbg
##### WinDbg is part of the Debugging Tools for Windows
##### Other debuggers in the tools: NTSD, CDB, KD
##### All debuggers are based on the same engine: DbgEng.DII
##### NTSD & CDB are user mode debuggers
-  Practically identical - NTSD spawns a new console window if launched from a console window. 
##### KD is a kernel mode debugger
##### WinDbg can serve as a user mode or kernel mode debugger
##### WinDbg is the only one with a graphical user interface
##### Most important window is the Command window
- Can do anything.
- Some shortcuts available through the menu.

---

#### Summary
- Although there are many Windows editions, the kernel is basically the same.
- User mode processes use subsystem DLLs to access OS functionality.
- A system service call entails transitioning from user mode to kernel mode (and back).

---

### 04. System Architecture (Part 1)
### Contents
1. Core system files Multiprocessing.
2. Subsystems and NTDLL.
3. System processes.
4. Wow64.
5. Summary.

---

### Core system files
##### Ntoskrnl.exe
- Executive and kernel on 64 bit systems
##### NtKrnlPa.exe
- Executive and kernel on 32 bit systems
##### Hal.dll
- Hardware Abstraction Layer
##### Win32k.sys
- Kernel component of the Windows subsystem
- Handles windowing and GDI
##### NtDII.dll
- System support routines and Native API dispatcher to executive services
##### Kernel32.dll, user32.dll, gdi32.dll, advapi32.dll
- Core Windows subsystem DLLS
#### CSRSS.exe ("Client Server Runtime SubSystem")
- The Windows subsystem process, if u killed it u will get BSOD(BlueScreenOfDeath).

##### U have to take a look at demo for core system files, check link below:
[Demo-Core System Files](https://drive.google.com/file/d/1n3DMtIS3cjp9AiILOsVUgoO7oc1Sz9ui/view?usp=drive_link)

---

### Symmetric multiprocessing
#### SMP
- All CPUs are the same and share main memory and have equal access to peripheral devices (no master/slave).
#### Basic architecture supports up to 32/64 CPUs (This was managed using a bitmask, that was the size of the machine word)
- Windows 7 64 bit & 2008 R2 support up to 256 cores.
- Uses a new concept of a "processor group".
#### Actual number of CPUs determined by licensing and product type
- Multiple cores do not count towards this limit.

#### Let's dig deeper into symmetric multiprocessing
- Lets loook at some information about processors we can view with tools, so i've my task manager open here on windows 11, and i can see under the CPU there is just one CPU here.
![CPU](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/07a1f068-c49e-459e-a004-c0bd2c8c5af6)

- Down here i can see that it has a one socket that means that it has just one chip and it has 6 cores but there are 12 logical processors.
![socket](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/9fa947d8-96d8-42c9-a498-c66c1710244d)
![cores](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/05d678f3-2e4e-4499-accb-3e88e304a740)
![logicalprocessors](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/f6742295-77d9-4a75-ac1f-4d5fd7851457)

- That means that each core is actually split into 2 logical processors using hyper-threading technology.
If i right click here and change graph to logical processors.
![changegraph](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/266f3396-cd4e-497e-b781-7b407e297dd6)

- I can see the various processors right here, and each logical processor can execute one thread at a time, so technically i have 12 threads that can run at the same time.
![logicalprocessors_](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/c60b5195-d70a-426d-8410-c038256fb245)

- In previous versions of task manager these details won't shown, if u wanna se these details and perhaps a little more we can look at process explorer.
- Using CTRL+i to view system information, and this shows us various information about the system naturally.
![viewsysinfo](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/0b3c51be-181a-4cc9-b54d-86f4af695bc1)

- Taking a look at CPU tab, By default it shows just one graph that actually is average of all logical processors on my system.
![sysinfoCPU](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/cb240301-7f23-403a-bf24-e954b463cc65)

- But i can show one graph per CPU, and again we can see one socket here and six cores in the topology part.
![onegraphperCPU](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/f40edc8a-1739-4d54-889e-ec293d06c0ad)

---

### Subsystems and NTDLL
#### Subsystems
#### A subsystem is a special view of the OS
- Exposes services via subsystem DLLS.
#### Original NT shipped with Win32, OS/2 and POSIX 1003.1 (POSIX-1)
- Windows XP dropped support for OS/2
- An enhanced POSIX version is available with the "Services for UNIX" product
#### The Windows subsystem must always be running
- Owner of keyboard, mouse and display
#### Some API functions use the Advanced Local Procedure Call (ALPC) to notify CSRSS of relevant events
#### Other subsystems configured to load on demand
#### Subsystem information stored in registry: HKLM\System\CCS\Control\Session Manager\Subsystems
#### Let's dig deeper into this
- Here is a screenshot of Windows 11 registry where the subsystems are located, we can see here in HKLM system which means that we're talking about something which is machine-wide and not user-relative.
![regedit](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/5e32a209-5405-47a3-bbd7-fa3366a7bfc7)

- We can see required value here(multiple string), Dubug and Windows.
So, debug has currently nothing in it and that's because it is used internally by Microsoft.
![dbgwin](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/48708e19-5262-4da6-9a00-1f6dbf00adae)

- But Windows has something which exactly is csrss.exe, which means this will always come up when Windows starts up.
![win](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/2ac8e590-e4d7-47d9-9550-066bfe0890ad)

- The optional value here has a list of optional subsystems in this case, it is None(:
![opt](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/b5a9e8de-3d8f-4e0f-81c8-2ccf4e19dbce)

- win32k.sys path is the kernel mode component of the Windows subsystem which as u may call handles, windowing and GDI, so technically we can replace that which practically is very difficult to do because it's mostly undocumented but still it is possible.
![win32k](https://github.com/A8N0RMAL/Windows-Internals/assets/119806250/b0d67ef3-2769-4434-83d8-77971d4f3b3a)

### The Native API
#### Implemented by NTDLL.DLL
- Used by subsystem DLLs and "native" images
- Undocumented interface
- Lowest layer of user mode code
#### Contains
- Various support functions
##### Dispatcher to kernel services
- Most of them accessible using Windows API "wrappers"

### Subsystem DLLs
#### Every image belongs to exactly one subsystem
- Value stored in image PE header
- Can view with Dependency Walker (depends.exe)
- Allows the Windows Loader to make correct decisions
#### An image of a certain subsystem calls API functions exposed through the subsystem DLLs
- E.g. kernel32.dll, user32.dll, etc. for the Windows subsystem
#### Some images belong to no subsystem
- "Native" images
- Which API functions do they call? we'll discuss this just continue :"

---

