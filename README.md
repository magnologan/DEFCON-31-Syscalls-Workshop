<img width="705" alt="Intro_ws_logo" src="https://user-images.githubusercontent.com/50073731/235339663-9c59e27f-57ea-4bbd-8188-e9e2849990f3.png">

# Direct Syscalls: A Journey from High to Low

## Disclaimer 
The content and all code examples int this repository are for teaching and research purpose only and must not be used in an unethical context! The code samples are not new and I make no claim to it. 
- Most of the code comes, as so often, from [**ired.team**](https://www.ired.team/), thank you [**@spotheplanet**](https://twitter.com/spotheplanet) for your brilliant work and sharing it with us all!. 
- For the syscall POCs, [**SysWhispers3**](https://github.com/klezVirus/SysWhispers3) was used, also thanks to [**@KlezVirus**](https://twitter.com/KlezVirus) for providing this awesome code. 

I would also like to thank all those members of the infosec community who have researched, shaped, and continue to research the topic of direct system calls. Without all of you, this workshop would never have been possible! Please forgive me if I have forgotten anyone.
- [**@Cneelis**](https://twitter.com/Cneelis) from [**@OutflankNL**](https://twitter.com/OutflankNL) and his research and article [**Red Team Tactics: Combining Direct System Calls and sRDI to bypass AV/EDR**](https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/)
- [**@j00ru**](https://twitter.com/j00ru) for his research in [**syscall tables**](https://j00ru.vexillium.org/syscalls/nt/64/)
- [**@Jackson_T**](https://twitter.com/Jackson_T) for his research and creation of [**SysWhispers**](https://github.com/jthuraisamy/SysWhispers) and [**SysWhispers2**](https://github.com/jthuraisamy/SysWhispers2)
- [**@AliceCliment**](https://twitter.com/AliceCliment) for her research and article [**A Syscall Journey in the Windows Kernel**](https://alice.climent-pommeret.red/posts/a-syscall-journey-in-the-windows-kernel/) and her other research and [**articles**](https://alice.climent-pommeret.red/) in area of syscalls 
- [**@modexpblog**](https://twitter.com/modexpblog) from [**@MDSecLabs**](https://twitter.com/MDSecLabs) and his research and article [**Bypassing User-Mode Hooks and Direct Invocation of System Calls for Red Teams**](https://www.mdsec.co.uk/2020/12/bypassing-user-mode-hooks-and-direct-invocation-of-system-calls-for-red-teams/)
- [**@CaptMeelo**](https://captmeelo.com/redteam/maldev/2021/11/18/av-evasion-syswhisper.html) for his research and article [**When You sysWhisper Loud Enough for AV to Hear You**]  (https://captmeelo.com/redteam/maldev/2021/11/18/av-evasion-syswhisper.html)
- [**@0xBoku**](https://twitter.com/0xBoku) for his overall great research, his [**blog**](https://0xboku.com/) and contributions to infosec, helping new community members, and the continued advancement of infosec 

## Abstract 
A system call is a technical instruction in the Windows operating system that allows a temporary transition from user mode to kernel mode. This is necessary, for example, when a user-mode application such as Notepad wants to save a document. Each system call has a specific syscall ID, which can vary from one version of Windows to another. Direct system calls are a technique for attackers (red team) to execute code in the context of Windows APIs via system calls without the targeted application (malware) obtaining Windows APIs from Kernel32.dll or native APIs from Ntdll.dll. The assembly instructions required to switch from user mode to kernel mode are built directly into the malware.

In recent years, more and more vendors have implemented the technique of user-mode hooking, which, simply put, allows an EDR to redirect code executed in the context of Windows APIs to its own hooking.dll for analysis. If the code executed does not appear to be malicious to the EDR, the affected system call will be executed correctly, otherwise the EDR will prevent execution. Usermode hooking makes malware execution more difficult, so attackers (red teams) use various techniques such as API unhooking, direct system calls or indirect system calls to bypass EDRs.

In this article, I will focus on the Direct System Call technique and show you how to create a Direct System Call shellcode dropper step-by-step using Visual Studio in C++. I will start with a dropper that only uses the Windows APIs (High Level APIs). In the second step, the dropper undergoes its first development and the Windows APIs are replaced by Native APIs (Medium Level APIs). And in the last step, the Native APIs are replaced by Direct System Calls (Low Level APIs).

 
