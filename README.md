Download Link: https://assignmentchef.com/product/solved-ecs150-project2
<br>
. For this project, you and a partner will be implementing a virtual machine threading API in either C or C++.  <strong>Your virtual machine will be tested on the CSIF 32-bit machines pc1 – pc32.</strong> You must submit your source files, readme and Makefile in a zip or tar.gz file to smartsite prior to the deadline.

You will be provided a machine abstraction upon which you will be building the thread scheduler. The virtual machine will load the “applications” from shared objects that implement the VMMain function. The virtual machine will need to support multiple user space preemptive threads. The virtual machine mutex API provides synchronization mechanism and file access is provided through the file API.

Threads have three priority levels low, medium, and high. Threads are created in the dead state and have and have state transitions as shown below.

3

<ol>

 <li>I/O, acquire mutex or timeout</li>

 <li>Scheduler selects process</li>

 <li>Process quantum up</li>

 <li>Process terminates</li>

 <li>Process activated</li>

 <li>Process blocks</li>

</ol>







A makefile has been provided that compiles the virtual machine as long as you provide your code as VirtualMachine.c or VirtualMachine.cpp. It will create the virtual machine call <strong>vm</strong>. The applications can be built by making the apps with <strong>make apps</strong>. New apps can be built by adding a C or C++ file in the apps directory and adding $(BINDIR)/<strong>filename</strong>.so to the Makefile apps line dependencies.




A working example of the vm and apps can be found in /home/cjnitta/ecs150. The vm syntax is vm [options] appname [appargs]. The possible options for vm are -t and -m; -t specifies the tick time in millisecond, and -m specifies the machine timeout/responsiveness in milliseconds. By default this is set to 10ms, for debugging purposes you can increase these values to slow the running of the vm. When specifying the application name the ./ should be prepended otherwise vm may fail to load the shared object file.




The machine layer is implemented using a cooperative process that communicates using the System V message queues. As such during your development your program may crash prior to the closing of the message queues. In order to determine the message queue id, you can use the ipcs command from the shell. You can remove the message queue with a call to ipcrm. You can read more about the System V IPC commands at: http://man7.org/linux/man-pages/man1/ipcs.1.html http://man7.org/linux/man-pages/man1/ipcrm.1.html




The function specifications for both the virtual machine and machine are provided in the subsequent pages.




You should avoid using existing source code as a primer that is currently available on the Internet. You <strong>must</strong> specify in your readme file any sources of code that you or your partner have viewed to help you complete this project. All class projects will be submitted to MOSS to determine if pairs of students have excessively collaborated with other pairs. Excessive collaboration, or failure to list external code sources will result in the matter being transferred to Student Judicial Affairs.







<strong><em>Name </em></strong>

VMStart – Start the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMStart(<strong>int</strong> tickms, <strong>int</strong> machinetickms, <strong>int</strong> argc,  <strong>char</strong> *argv[]);

<h1>Description</h1>

VMStart() starts the virtual machine by loading the module specified by <em>argv</em>[0]. The <em>argc</em> and <em>argv</em> are passed directly into the VMMain() function that exists in the loaded module. The time in milliseconds of the virtual machine tick is specified by the <em>tickms</em> parameter, the machine responsiveness is specified by the <em>machinetickms</em>.

<h1>Return Value</h1>

Upon successful loading and running of the VMMain() function, VMStart() will return VM_STATUS_SUCCESS after VMMain() returns. If the module fails to load, or the module does not contain a VMMain() function, VM_STATUS_FAILURE is returned.







VMLoadModule – Loads the module and returns a reference to VMMain function.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>typedef</strong> <strong>void</strong> (*TVMMainEntry)(<strong>int</strong>, <strong>char</strong>*[]);




TVMMainEntry VMLoadModule(<strong>const</strong> <strong>char</strong> *module);

<h1>Description</h1>

VMLoadModule() loads the shared object module (or application) specified by the <em>module</em> filename. Once the module has been loaded a reference to VMMain function obtained. The source for VMLoadModule is provided in VirtualMachineUtils.c

<h1>Return Value</h1>

Upon successful loading of the module specified by <em>module</em> filename, a reference to the VMMain function is returned, upon failure NULL is returned.




VMUnloadModule – Unloads the previously loaded module.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>void</strong> VMUnloadModule(<strong>void</strong>);

<h1>Description</h1>

VMUnloadModule() unloads the previously loaded module. The source for

VMUnloadModule is provided in VirtualMachineUtils.c

<h1>Return Value</h1>

N/A

VMThreadCreate – Creates a thread in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>typedef</strong> <strong>void</strong> (*TVMThreadEntry)(<strong>void</strong> *);




TVMStatus VMThreadCreate(TVMThreadEntry entry, <strong>void</strong> *param,

TVMMemorySize memsize, TVMThreadPriority prio, TVMThreadIDRef tid);

<h1>Description</h1>

VMThreadCreate() creates a thread in the virtual machine. Once created the thread is in the dead state VM_THREAD_STATE_DEAD. The <em>entry</em> parameter specifies the function of the thread, and <em>param</em> specifies the parameter that is passed to the function.

The size of the threads stack is specified by <em>memsize</em>, and the priority is specified by <em>prio</em>. The thread identifier is put into the location specified by the <em>tid</em> parameter.

<h1>Return Value</h1>

Upon successful creation of the thread VMThreadCreate() returns

VM_STATUS_SUCCESS. VMThreadCreate() returns

VM_STATUS_ERROR_INVALID_PARAMETER if either <em>entry</em> or <em>tid</em> is NULL.







VMThreadDelete – Deletes a dead thread from the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMThreadDelete(TVMThreadID thread);

<h1>Description</h1>

VMThreadDelete() deletes the dead thread specified by <em>thread</em> parameter from the virtual machine.

<h1>Return Value</h1>

Upon successful deletion of the thread from the virtual machine, VMThreadDelete() returns VM_STATUS_SUCCESS. If the thread specified by the thread identifier <em>thread</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the thread does

exist, but is not in the dead state VM_THREAD_STATE_DEAD, VM_STATUS_ERROR_INVALID_STATE is returned.




VMThreadActivate – Activates a dead thread in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMThreadActivate(TVMThreadID thread);

<h1>Description</h1>

VMThreadActivate() activates the dead thread specified by <em>thread</em> parameter in the virtual machine. After activation the thread enters the ready state

VM_THREAD_STATE_READY, and must begin at the <em>entry</em> function specified.

<h1>Return Value</h1>

Upon successful activation of the thread in the virtual machine, VMThreadActivate() returns VM_STATUS_SUCCESS. If the thread specified by the thread identifier <em>thread</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the thread does

exist, but is not in the dead state VM_THREAD_STATE_DEAD, VM_STATUS_ERROR_INVALID_STATE is returned.




VMThreadTerminate– Terminates a thread in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMThreadTerminate(TVMThreadID thread);

<h1>Description</h1>

VMThreadTerminate() terminates the dead thread specified by <em>thread</em> parameter in the virtual machine. After termination the thread enters the ready state

VM_THREAD_STATE_DEAD, and must release any mutexes that it currently holds. The termination of a thread can trigger another thread to be scheduled.

<h1>Return Value</h1>

Upon successful termination of the thread in the virtual machine, VMThreadTerminate() returns VM_STATUS_SUCCESS. If the thread specified by the thread identifier <em>thread</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the thread does

exist, but is in the dead state VM_THREAD_STATE_DEAD, VM_STATUS_ERROR_INVALID_STATE is returned.







VMThreadID – Retrieves thread identifier of the current operating thread.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMThreadID(TVMThreadIDRef threadref);

<h1>Description</h1>

VMThreadID() puts the thread identifier of the currently running thread in the location specified by <em>threadref</em>.

<h1>Return Value</h1>

Upon successful retrieval of the thread identifier from the virtual machine,

VMThreadID() returns VM_STATUS_SUCCESS. If the thread specified by the thread identifier <em>thread</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the parameter <em>threadref</em> is NULL, VM_STATUS_ERROR_INVALID_PARAMETER is returned.




VMThreadState – Retrieves the state of a thread in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>#define</strong> VM_THREAD_STATE_DEAD       ((TVMThreadState)0x00)

<strong>#define</strong> VM_THREAD_STATE_RUNNING    ((TVMThreadState)0x01)

<strong>#define</strong> VM_THREAD_STATE_READY      ((TVMThreadState)0x02)

<strong>#define</strong> VM_THREAD_STATE_WAITING    ((TVMThreadState)0x03)




TVMStatus VMThreadState(TVMThreadID thread, TVMThreadStateRef state);

<h1>Description</h1>

VMThreadState() retrieves the state of the thread specified by <em>thread</em> and places the state in the location specified by <em>state</em>.

<h1>Return Value</h1>

Upon successful retrieval of the thread state from the virtual machine, VMThreadState() returns VM_STATUS_SUCCESS. If the thread specified by the thread identifier <em>thread</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the parameter <em>stateref</em> is NULL, VM_STATUS_ERROR_INVALID_PARAMETER is returned.




VMThreadSleep– Puts the current thread in the virtual machine to sleep.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>#define</strong> VM_TIMEOUT_INFINITE  ((TVMTick)0)

<strong>#define</strong> VM_TIMEOUT_IMMEDIATE ((TVMTick)-1)




TVMStatus VMThreadSleep(TVMTick tick);

<h1>Description</h1>

VMThreadSleep() puts the currently running thread to sleep for <em>tick</em> ticks. If tick is specified as VM_TIMEOUT_IMMEDIATE the current process yields the remainder of its processing quantum to the next ready process of equal priority.

<h1>Return Value</h1>

Upon successful sleep of the currently running thread, VMThreadSleep() returns

VM_STATUS_SUCCESS. If the sleep duration <em>tick</em> specified is

VM_TIMEOUT_INFINITE, VM_STATUS_ERROR_INVALID_PARAMETER is returned.







VMMutexCreate– Creates a mutex in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMMutexCreate(TVMMutexIDRef mutexref);

<h1>Description</h1>

VMMutexCreate() creates a mutex in the virtual machine. Once created the mutex is in the unlocked state. The mutex identifier is put into the location specified by the <em>mutexref</em> parameter.

<h1>Return Value</h1>

Upon successful creation of the thread VMMutexCreate() returns

VM_STATUS_SUCCESS. VMMutexCreate() returns

VM_STATUS_ERROR_INVALID_PARAMETER if either <em>mutexref</em> is NULL.




VMMutexDelete – Deletes a mutex from the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMMutexDelete(TVMMutexID mutex);

<h1>Description</h1>

VMMutexDelete() deletes the unlocked mutex specified by <em>mutex</em> parameter from the virtual machine.

<h1>Return Value</h1>

Upon successful deletion of the thread from the virtual machine, VMMutexDelete() returns VM_STATUS_SUCCESS. If the mutex specified by the thread identifier <em>mutex</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the mutex does exist, but is currently held by a thread, VM_STATUS_ERROR_INVALID_STATE is returned.




VMMutexQuery– Queries the owner of a mutex in the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMMutexQuery(TVMMutexID mutex, TVMThreadIDRef ownerref);

<h1>Description</h1>

VMMutexQuery() retrieves the owner of the mutex specified by <em>mutex</em> and places the thread identifier of owner in the location specified by <em>ownerref</em>. If the mutex is currently unlocked, VM_THREAD_ID_INVALID returned as the owner.

<h1>Return Value</h1>

Upon successful querying of the mutex owner from the virtual machine, VMMutexQuery() returns VM_STATUS_SUCCESS. If the mutex specified by the mutex identifier <em>mutex</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the parameter <em>ownerref</em> is NULL, VM_STATUS_ERROR_INVALID_PARAMETER is returned.







VMMutexAcquire – Locks the mutex.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”

<strong>#define</strong> VM_TIMEOUT_INFINITE  ((TVMTick)0)

<strong>#define</strong> VM_TIMEOUT_IMMEDIATE ((TVMTick)-1)




TVMStatus VMMutexAcquire(TVMMutexID mutex, TVMTick timeout);

<h1>Description</h1>

VMMutexAcquire() attempts to lock the mutex specified by <em>mutex</em> waiting up to <em>timeout</em> ticks. If <em>timeout</em> is specified as VM_TIMEOUT_IMMEDIATE the current returns immediately if the mutex is already locked. If <em>timeout</em> is specified as

VM_TIMEOUT_INFINITE the thread will block until the <em>mutex</em> is acquired.

<h1>Return Value</h1>

Upon successful acquisition of the currently running thread, VMMutexAcquire() returns

VM_STATUS_SUCCESS. If the <em>timeout</em> expires prior to the acquisition of the mutex, VM_STATUS_FAILURE is returned. If the mutex specified by the mutex identifier <em>mutex</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned.







VMMutexRelease – Releases a mutex held by the currently running thread.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMMutexRelease(TVMMutexID mutex);

<h1>Description</h1>

VMMutexRelease() releases the mutex specified by the <em>mutex</em> parameter that is currently held by the running thread. Release of the mutex may cause another higher priority thread to be scheduled if it acquires the newly released mutex.

<h1>Return Value</h1>

Upon successful release of the mutex, VMMutexRelease() returns

VM_STATUS_SUCCESS. If the mutex specified by the mutex identifier <em>mutex</em> does not exist, VM_STATUS_ERROR_INVALID_ID is returned. If the mutex specified by the mutex identifier <em>mutex</em> does exist, but is not currently held by the running thread,

VM_STATUS_ERROR_INVALID_STATE is returned.







VMPrint, VMPrintError, and VMFilePrint – Prints out to a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




<strong>#define</strong> VMPrint(format, …)

VMFilePrint ( 1,  format, ##__VA_ARGS__)

<strong>#define</strong> VMPrintError(format, …)

VMFilePrint ( 2,  format, ##__VA_ARGS__)

TVMStatus VMFilePrint(<strong>int</strong> filedescriptor, <strong>const char</strong> *format, …);

<h1>Description</h1>

VMFilePrint() writes the C string pointed by <em>format</em> to the file specified by <em>filedescriptor</em>. If format includes format specifiers (subsequences beginning with %), the additional arguments following format are formatted and inserted in the resulting string replacing their respective specifiers. The VMPrint and VMPrintError macros have been provided as a convenience for calling VMFilePrint. The source code for VMFilePrint is provided in VirtualMachineUtils.c

<h1>Return Value</h1>

Upon successful writing out of the <em>format</em> string to the file VM_STATUS_SUCCESS is returned, upon failure VM_STATUS_FAILURE is returned.










VMFileOpen    Opens and possibly creates a file in the file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileOpen(<strong>const</strong> <strong>char</strong> *filename, <strong>int</strong> flags, <strong>int</strong> mode,  <strong>int</strong> *filedescriptor);

<h1>Description</h1>

VMFileOpen() attempts to open the file specified by <em>filename</em>, using the flags specified by <em>flags</em> parameter, and mode specified by <em>mode</em> parameter. The file descriptor of the newly opened file will be placed in the location specified by <em>filedescriptor</em>. The flags and mode values follow the same format as that of open system call. The filedescriptor returned can be used in subsequent calls to VMFileClose(), VMFileRead(),

VMFileWrite(), and VMFileSeek(). When a thread calls VMFileOpen() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful opening of the file is completed.

<h1>Return Value</h1>

Upon successful opening of the file, VMFileOpen() returns VM_STATUS_SUCCESS, upon failure VMFileOpen() returns VM_STATUS_FAILURE. If either <em>filename</em> or <em>filedescriptor</em> are NULL, VMFileOpen() returns

VM_STATUS_ERROR_INVALID_PARAMETER.




VMFileClose    Closes a file that was previously opened.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileClose(<strong>int</strong> filedescriptor);

<h1>Description</h1>

VMFileClose() closes a file previously opened with a call to VMFileOpen().When a thread calls VMFileClose() it blocks in the wait state

VM_THREAD_STATE_WAITING until the either successful or unsuccessful closing of the file is completed.

<h1>Return Value</h1>

Upon successful closing of the file VMFileClose() returns VM_STATUS_SUCCESS, upon failure VMFileClose() returns VM_STATUS_FAILURE.







VMFileRead    Reads data from a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileRead(<strong>int</strong> filedescriptor, <strong>void</strong> *data, <strong>int</strong> *length);

<h1>Description</h1>

VMFileRead() attempts to read the number of bytes specified in the integer referenced by <em>length</em> into the location specified by <em>data</em> from the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The actual number of bytes transferred by the read will be updated in the <em>length</em> location. When a thread calls VMFileRead() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful reading of the file is completed.

<h1>Return Value</h1>

Upon successful reading from the file, VMFileRead() returns VM_STATUS_SUCCESS, upon failure VMFileRead() returns VM_STATUS_FAILURE. If <em>data</em> or <em>length</em>

parameters are NULL, VMFileRead() returns

VM_STATUS_ERROR_INVALID_PARAMETER.




VMFileWrite    Writes data to a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileWrite(<strong>int</strong> filedescriptor, <strong>void</strong> *data, <strong>int</strong> *length);

<h1>Description</h1>

VMFileWrite() attempts to write the number of bytes specified in the integer referenced by <em>length</em> from the location specified by <em>data</em> to the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The actual number of bytes transferred by the write will be updated in the <em>length</em> location. When a thread calls VMFileWrite() it blocks in the wait state

VM_THREAD_STATE_WAITING until the either successful or unsuccessful writing of the file is completed.

<h1>Return Value</h1>

Upon successful writing from the file, VMFileWrite() returns VM_STATUS_SUCCESS, upon failure VMFileWrite() returns VM_STATUS_FAILURE. If <em>data</em> or <em>length</em>

parameters are NULL, VMFileWrite() returns

VM_STATUS_ERROR_INVALID_PARAMETER.







VMFileSeek    Seeks within a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileSeek(<strong>int</strong> filedescriptor, <strong>int</strong> offset, <strong>int</strong> whence,  <strong>int</strong> *newoffset);

<h1>Description</h1>

VMFileSeek() attempts to seek the number of bytes specified by <em>offset</em> from the location specified by <em>whence</em> in the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The new offset placed in the <em>newoffset</em> location if the parameter is not NULL. When a thread calls VMFileSeek() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful seeking in the file is completed.

<h1>Return Value</h1>

Upon successful seeking in the file, VMFileSeek () returns VM_STATUS_SUCCESS, upon failure VMFileSeek() returns VM_STATUS_FAILURE.







MachineContextSave – Saves a machine context.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h” <strong>typedef</strong> <strong>struct</strong>{     jmp_buf DJumpBuffer;

} SMachineContext, *SMachineContextRef;




<strong>#define</strong> MachineContextSave(mcntx) setjmp((mcntx)-&gt;DJumpBuffer)

<h1>Description</h1>

MachineContextSave() saves the machine context that is specified by the parameter <em>mcntx</em>.

<h1>Return Value</h1>

Upon successful saving of the context, MachineContextSave () returns 0.




MachineContextRestore – Restores a machine context.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h” <strong>typedef</strong> <strong>struct</strong>{     jmp_buf DJumpBuffer;

} SMachineContext, *SMachineContextRef;




<strong>#define</strong> MachineContextRestore(mcntx) longjmp((mcntx)-&gt;DJumpBuffer, 1)

<h1>Description</h1>

MachineContextRestore() restores a previously saved the machine context that is specified by the parameter <em>mcntx</em>.

<h1>Return Value</h1>

Upon successful restoring of the context, MachineContextRestore() should not return.







MachineContextSwitch – Switches machine context.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h” <strong>typedef</strong> <strong>struct</strong>{     jmp_buf DJumpBuffer;

} SMachineContext, *SMachineContextRef;




<strong>#define</strong> MachineContextSwitch (mcntxold,mcntxnew)       if(setjmp((mcntxold)-&gt;DJumpBuffer) == 0)        longjmp((mcntxnew)-&gt;DJumpBuffer, 1)

<h1>Description</h1>

MachineContextSwitch() switches context to a previously saved the machine context that is specified by the parameter <em>mcntxnew</em>, and stores the current context in the parameter specified by <em>mctxold</em>.

<h1>Return Value</h1>

Upon successful switching of the context, MachineContextRestore() should not return until the original context is restored.







MachineContextCreate – Creates a machine context.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h” <strong>typedef</strong> <strong>struct</strong>{     jmp_buf DJumpBuffer;

} SMachineContext, *SMachineContextRef;




<strong>void</strong> MachineContextCreate(SMachineContextRef mcntxref,  <strong>void</strong> (*entry)(<strong>void</strong> *), <strong>void</strong> *param, <strong>void</strong> *stackaddr, size_t stacksize);

<h1>Description</h1>

MachineContextCreate() creates a context that will enter in the function specified by <em>entry</em> and passing it the parameter <em>param</em>.  The contexts stack of size <em>stacksize</em> must be specified by the <em>stackaddr</em> parameter. The newly created context will be stored in the <em>mcntxref</em> parameter, this context can be used in subsequent calls to MachineContextRestore(), or MachineContextSwitch().

<h1>Return Value</h1>

N/A




MachineInitialize – Initializes the machine abstraction layer.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>void</strong> MachineInitialize(<strong>int</strong> timeout);

<h1>Description</h1>

MachineInitialize() initializes the machine abstraction layer. The <em>timeout</em> parameter specifies the number of milliseconds the machine will sleep between checking for requests. Reducing this number will increase its responsiveness.

<h1>Return Value</h1>

N/A




MachineTerminate – Terminates the machine abstraction layer.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>void</strong> MachineTerminate(<strong>void</strong>);

<h1>Description</h1>

MachineTerminate() terminates the machine abstraction layer. This closes down the cooperative process that is executing the machine abstraction.

<h1>Return Value</h1>

N/A




MachineEnableSignals – Enables all signals.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>void</strong> MachineEnableSignals(<strong>void</strong>);

<h1>Description</h1>

MachineEnableSignals() enables all signals so that the virtual machine may be “interrupted” asynchronously.

<h1>Return Value</h1>

N/A




MachineSuspendSignals – Suspends all signals.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>typedef</strong> sigset_t TMachineSignalState, *TMachineSignalStateRef;

<strong>void </strong>MachineSuspendSignals(TMachineSignalStateRef sigstate);

<h1>Description</h1>

MachineSuspendSignals() suspends all signals so that the virtual machine will not be “interrupted” asynchronously. The current state of the signal mask will be placed in the location specified by the parameter <em>sigstate</em>. This signal state can be restored by a call to MachineResumeSignals().

<h1>Return Value</h1>

N/A




MachineResumeSignals – Resumes signal state.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>typedef</strong> sigset_t TMachineSignalState, *TMachineSignalStateRef;

<strong>void </strong>MachineResumeSignals(TMachineSignalStateRef sigstate);

<h1>Description</h1>

MachineResumeSignals() resumes all signals that were enabled when previous call to MachineSuspendSignals() was called so that the virtual machine will my be “interrupted” asynchronously. The signal mask in the location specified by the parameter <em>sigstate</em> will be restored to the virtual machine. This signal state should have been initialized by a previous call to MachineSuspendSignals().

<h1>Return Value</h1>

N/A




MachineRequestAlarm – Requests periodic alarm callback.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>typedef</strong> <strong>void</strong> (*TMachineAlarmCallback)(<strong>void</strong> *calldata);




<strong>void </strong>MachineRequestAlarm(useconds_t usec,  TMachineAlarmCallback callback, <strong>void</strong> *calldata);

<h1>Description</h1>

MachineRequestAlarm() requests periodic alarm callback from the machine abstraction layer. The callback function specified by the <em>callback</em> parameter will be called at a period of <em>usec</em> microseconds being passed the parameter specified by <em>calldata</em>. The alarm callback can be canceled by calling MachineRequestAlarm() with a parameter of 0 <em>usec</em>.

<h1>Return Value</h1>

N/A







MachineFileOpen    Opens a file with the machine abstraction layer.

<h1>Synopsys</h1>

<strong>#include</strong> “Machine.h”

<strong>typedef</strong> <strong>void</strong> (*TMachineFileCallback)(<strong>void</strong> *calldata, <strong>int</strong>




<strong>void</strong> MachineFileOpen(<strong>const</strong> <strong>char</strong> *filename, <strong>int</strong> flags, <strong>int</strong> mode, TMachineFileCallback callback, <strong>void</strong> *calldata);

<h1>Description</h1>

MachineFileOpen() attempts to open the file specified by <em>filename</em>, using the flags specified by <em>flags</em> parameter, and mode specified by <em>mode</em> parameter. The file descriptor of the newly opened file will be passed in to the <em>callback</em> function as the <em>result</em>. The <em>calldata</em> parameter will also be passed into the <em>callback</em> function upon completion of the open file request. The flags and mode values follow the same format as that of open system call. The <em>result</em> returned can be used in subsequent calls to MachineFileClose(), MachineFileRead(), MachineFileWrite(), and MachineFileSeek(). MachineFileOpen() should return immediately, but will call the <em>callback</em> function asynchronously when completed.

<h1>Return Value</h1>

N/A







FileRead    Reads from a file in the machine abstraction.

<strong><em> </em></strong>

“Machine.h”  <strong>void</strong> (*TMachineFileCallback)(<strong>void</strong> *calldata,




<strong>void</strong> MachineFileRead(<strong>int</strong> fd, <strong>void</strong> *data, <strong>int</strong> length, TMachineFileCallback callback, <strong>void</strong> *calldata);

<h1>Description</h1>

MachineFileRead() attempts to read the number of bytes specified in by <em>length</em> into the location specified by <em>data</em> from the file specified by <em>fd</em>. The <em>fd</em> should have been obtained by a previous call to MachineFileOpen(). The actual number of bytes transferred will be returned in the <em>result</em> parameter when the <em>callback</em> function is called. Upon failure the <em>result</em> will be less than zero.The <em>calldata</em> parameter will also be passed into the <em>callback</em> function upon completion of the read file request. MachineFileRead () should return immediately, but will call the <em>callback</em> function asynchronously when completed.

<h1>Return Value</h1>

N/A




FileWrite    Writes to a file in the machine abstraction.

<strong><em> </em></strong>

“Machine.h”  <strong>void</strong> (*TMachineFileCallback)(<strong>void</strong> *calldata,




<strong>void</strong> MachineFileRead(<strong>int</strong> fd, <strong>void</strong> *data, <strong>int</strong> length, TMachineFileCallback callback, <strong>void</strong> *calldata);

<h1>Description</h1>

MachineFileWrite() attempts to write the number of bytes specified in by <em>length</em> into the location specified by <em>data</em> to the file specified by <em>fd</em>. The <em>fd</em> should have been obtained by a previous call to MachineFileOpen(). The actual number of bytes transferred will be returned in the <em>result</em> parameter when the <em>callback</em> function is called. Upon failure the <em>result</em> will be less than zero. The <em>calldata</em> parameter will also be passed into the <em>callback</em> function upon completion of the write file request. MachineFileWrite() should return immediately, but will call the <em>callback</em> function asynchronously when completed.

<h1>Return Value</h1>

N/A

FileSeek    Seeks in a file in the machine abstraction.

<strong><em> </em></strong>

“Machine.h”  <strong>void</strong> (*TMachineFileCallback)(<strong>void</strong> *calldata,




<strong>void</strong> MachineFileSeek(<strong>int</strong> fd, <strong>int</strong> offset, <strong>int</strong> whence, TMachineFileCallback callback, <strong>void</strong> *calldata);

<h1>Description</h1>

MachineFileSeek() attempts to seek the number of bytes specified in by <em>offset</em> from the location specified by <em>whence</em> in the file specified by <em>fd</em>. The <em>fd</em> should have been obtained by a previous call to MachineFileOpen(). The actual offset in the file will be returned in the <em>result</em> parameter when the <em>callback</em> function is called. Upon failure the <em>result</em> will be less than zero. The <em>calldata</em> parameter will also be passed into the <em>callback</em> function upon completion of the seek file request. MachineFileSeek() should return immediately, but will call the <em>callback</em> function asynchronously when completed.

<h1>Return Value</h1>

N/A




FileClose    Closes a file in the machine abstraction layer.

<strong><em> </em></strong>

“Machine.h”  <strong>void</strong> (*TMachineFileCallback)(<strong>void</strong> *calldata,




<strong>void</strong> MachineFileClose(<strong>int</strong> fd, TMachineFileCallback callback,  <strong>void</strong> *calldata);

<h1>Description</h1>

MachineFileClose() attempts to close the file specified by <em>fd</em>. The <em>fd</em> should have been obtained by a previous call to MachineFileOpen(). The <em>result</em> parameter when the <em>callback</em> function is called will be zero upon success; upon failure the <em>result</em> will be less than zero. The <em>calldata</em> parameter will also be passed into the <em>callback</em> function upon completion of the seek file request. MachineFileClose() should return immediately, but will call the <em>callback</em> function asynchronously when completed.

<h1>Return Value</h1>

N/A


