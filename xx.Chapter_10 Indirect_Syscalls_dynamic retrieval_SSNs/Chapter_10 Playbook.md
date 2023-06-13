## LAB Exercise 6: Dynamic Retrieval from SSNs

In the first bonus chapter we want to further develop our indirect syscall dropper. Until now, we had the limitation that our dropper would only work in the context of the Windows version that was used to debug the system service numbers (SSNs) for the used native functions ``NtAllocateVirtualMemory``, ``NtWriteVirtualMemory``, ``NtCreateThreadEx`` and ``NtWaitForSingleObject``. Why? Because to get the basics for direct and indirect syscalls, we have implemented the SSNs as hardcoded values in our assembly resource file. But normally, when we are preparing for a red team engagement, we do not know the Windows version of our target client. So we want to make our indirect syscall dropper a bit more flexible and instead of hardcoding the SSNs, we want to retrieve them dynamically at runtime from ntdll.dll. 
  

## Exercise 6 Tasks: 
### Develop your direct or indirect syscall dropper to dynamically retrieve SSNs.
| Task Nr.   | Task Description |
| :---:      | ---              |
|  1         | Download the direct or indirect syscall POC from the code section of this chapter.                 |
|  2         | Most of the code is already implemented. However, to implement the dynamic SSN retrieval functionality, you will need to complete the following tasks: <ul><li>Complete the missing code in the main code section</li><li>Complete the missing code in the ``syscalls.asm`` file</li></ul>                  |
|  3          | Create a staged x64 meterpreter shellcode with msfvenom, copy it to the poc and compile the poc.                 |
|  4          | Create and run a staged x64 meterpreter listener using msfconsole.                  |
| 5           | un your compiled .exe and check that a stable command and control channel opens.                 |


### Analyse the Dropper
| Task Nr.   | Task Description |
| :---:      | ---              |
| 6          | Use **x64dbg** to debug or analyse the dropper. <ul><li>What differences can you see between a dropper with hardcoded SSNs and a dropper that dynamically retrieves SSNs at runtime?</li></ul>                |

## Visual Studio
You can download the direct- or indirect syscall poc from the code section of this chapter. To retrieve the SSNs dynamicalla at runtime from ntdll.dll, we have to implement the following code. 

### Start Address Native Function
In order to be able to dynamically retrieve the SSN for each of the native functions used in our code, we first need to define a pointer to a function that holds the start address of that function. If you remember, this part of the code was already implemented in the chapter where we built the indirect syscall dropper, because we used the same principle to get the address for the syscall instruction of each function. This means that part of the code in the main file is already implemented.  
<summary>Solution</summary>
    
```C
// Declare and initialize a pointer to the NtAllocateVirtualMemory function and get the address of the NtAllocateVirtualMemory function in the ntdll.dll module
    UINT_PTR pNtAllocateVirtualMemory = (UINT_PTR)GetProcAddress(hNtdll, "NtAllocateVirtualMemory");
    UINT_PTR pNtWriteVirtualMemory = (UINT_PTR)GetProcAddress(hNtdll, "NtWriteVirtualMemory");
    UINT_PTR pNtCreateThreadEx = (UINT_PTR)GetProcAddress(hNtdll, "NtCreateThreadEx");
    UINT_PTR pNtWaitForSingleObject = (UINT_PTR)GetProcAddress(hNtdll, "NtWaitForSingleObject");     
```
     
</details>