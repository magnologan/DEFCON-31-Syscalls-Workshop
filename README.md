<p align="center">
<img width="700" alt="image" src="https://github.com/VirtualAlllocEx/DEFCON-31-Workshop-Syscalls/assets/50073731/7acf6efa-da5e-47fc-83fe-50f92d18a676">
</p>

# (In)direct Syscalls: A journey from high to low  
## DEFCON 31 | Red Team Village | RedOps

## Disclaimer 
The content and all code examples in this repository are for educational and research purposes only and should only be used in an ethical context! The code examples are not new and I do not claim them to be. Most of the code or the basis  comes, as so often, from [**ired.team**](https://www.ired.team/), thank you [**@spotheplanet**](https://twitter.com/spotheplanet) for your brilliant work and sharing it with us all. Also many thanks to [**@mrexodia**](https://twitter.com/mrexodia) for your awesome tool [**x64dbg**](https://twitter.com/x64dbg).

Furthermore, and very importantly, this workshop is **not a silver bullet** in the context of EDR evasion, but it **should help to understand** the basics of ``Win32 APIs``, ``Native APIs``, ``direct syscalls`` and ``indirect syscalls`` and a bit about ``call stacks`` in context of shellcode execution and EDR evasion, no more and no less. The aim of this workshop is not to show the most stealthy options or the most complex pocs for direct syscalls and indirect syscalls, instead I will focus on teaching the basics. This means that we will use as few tools as possible and do as much work as possible by hand.  

I would like to **thank all those members** of the infosec community who have researched, shaped and continue to research the topic of syscalls, direct system calls and indirect syscalls etc. Without all of you, this workshop would not have been possible!

**Creds to:**

| Twitter Handle                             						 | Contribution and Research                                                                                                      															    																					 	  			| Company |
| :---:                                         				     | :---:                                                                                                                 																		    																					 		    |:---: | 
| [@spotheplanet](https://twitter.com/spotheplanet)             	 |   [His whole awesome blog and research](https://www.ired.team/)					   																																																						|         |
| [@NinjaParanoid](https://twitter.com/NinjaParanoid) 			     | For his great blogs, research, courses and always answering my questions. [Hiding In PlainSight - Indirect Syscall is Dead! Long Live Custom Call Stacks](https://0xdarkvortex.dev/hiding-in-plainsight/), [Hiding In PlainSight - Proxying DLL Loads To Hide From ETWTI Stack Tracing](https://0xdarkvortex.dev/proxying-dll-loads-for-hiding-etwti-stack-tracing/)   | [Dark Vortex](https://0xdarkvortex.dev/), [Brute Ratel](https://bruteratel.com/)|
| [@ShitSecure](https://twitter.com/ShitSecure) 					 | For his great research in general, his blog https://s3cur3th1ssh1t.github.io/ and for the great discussion about EDRs, syscalls, etc.     																																						|         |
| [@KlezVirus](https://twitter.com/KlezVirus)						 | For his great blogs, research and great discussions about EDRs, syscalls, etc. [SysWhispers3](https://github.com/klezVirus/SysWhispers3), [SysWhispers is dead, long live SysWhispers!](https://klezvirus.github.io/RedTeaming/AV_Evasion/NoSysWhisper/), [SilentMoonwalk](https://github.com/klezVirus/SilentMoonwalk)							|         |
| [@AliceCliment](https://twitter.com/alicecliment?lang=de)          | For her great blog, research and the discussions about EDRs, syscalls etc. https://alice.climent-pommeret.red/ https://alice.climent-pommeret.red/posts/a-syscall-journey-in-the-windows-kernel/ https://alice.climent-pommeret.red/posts/direct-syscalls-hells-halos-syswhispers2/ 																																																				| 			| 								|
| [@Cneelis](https://twitter.com/Cneelis)    						 | [Red Team Tactics: Combining Direct System Calls and sRDI to bypass AV/EDR](https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/)   		 			  																		        | [OutflankNL](https://outflank.nl/) |
| [@0xBoku](https://twitter.com/0xBoku)								 | For his overall great research, his [blog](https://0xboku.com/) and contributions to infosec, helping new community members, and the continued advancement of infosec                                                                                                                 			| [IBM X-Force Red](https://www.ibm.com/x-force/team) |
| [@Jackson_T](https://twitter.com/Jackson_T)						 | [SysWhispers](https://github.com/jthuraisamy/SysWhispers), [SysWhispers2](https://github.com/jthuraisamy/SysWhispers2)																							 															     			    |         |
| [@j00ru](https://twitter.com/j00ru)								 | [syscall tables](https://j00ru.vexillium.org/syscalls/nt/64/)                                                                                                         																												 			|         |
| [@modexpblog](https://twitter.com/modexpblog)					     | [Bypassing User-Mode Hooks and Direct Invocation of System Calls for Red Teams](https://www.mdsec.co.uk/2020/12/bypassing-user-mode-hooks-and-direct-invocation-of-system-calls-for-red-teams/)        																				  			| [@MDSecLabs](https://www.mdsec.co.uk/      )  |
| [@netero_1010](https://twitter.com/netero_1010)			         |  [Indirect Syscall in CSharp](https://www.netero1010-securitylab.com/evasion/indirect-syscall-in-csharp) 																						  																							        |         |
| [@CaptMeelo](https://captmeelo.com/redteam/maldev/2021/11/18/av-evasion-syswhisper.html) | [When You sysWhisper Loud Enough for AV to Hear You](https://captmeelo.com/redteam/maldev/2021/11/18/av-evasion-syswhisper.html) 																																	        | 	      | 


## Introduction
In recent years, more and more **EDR vendors** have implemented the technique of **user-mode hooking**, which, simply put, allows an EDR to redirect code executed in the context of Windows APIs to its own hooking.dll for analysis. If the code executed does not appear to be malicious to the EDR, the affected system call will be executed correctly, otherwise the EDR will prevent execution. User-mode hooking makes malware execution more difficult, so attackers (red teams) use various techniques such as API unhooking, direct system calls or indirect system calls to bypass EDRs.

## Abstract 
This workshop is designed for new community members or members who want to understand and learn about **system calls** in general and the Red Teaming techniques of **direct system calls** and **indirect system calls** on **Windows OS**. All the content for this workshop, slides, exercises, playbooks etc. can be found in this repository. I hope you enjoy it and it can help you get a basic understanding of syscalls and the red teaming techniques of direct syscalls and indirect syscalls. 

**Important note!** This workshop **only covers syscalls** in the **context of x64**, x86 syscalls work a bit differently and are not covered here. 



![image](https://github.com/VirtualAlllocEx/DEFCON-31-Syscalls-Workshop/assets/50073731/04c6681a-d3bf-46ba-a0a8-dcec454f1c1d)

- **Chapter 1: Windows NT Basics** 
     - We take a look at the basics of Windows NT architecture that are necessary for this workshop.

- **Chapter 2: Windows OS System Calls** 
     - What are system calls in general and why are the neccessary in Windows OS?

- **Chapter 3: Concept of Direct Syscalls**
     -  What are direct syscalls, how do they work and why have they been used by red teamers?

- **Chapter 4: Win32-APIs**
     - We will create a shellcode dropper based on the Win32 APIs (high level APIs) and we will also analyse it.

- **Chapter 5: Native-APIs**
     - We will create a shellcode dropper based on the Native APIs (medium level APIs) and we will also analyse it.

- **Chapter 6: Direct Syscalls**
     - We will create a shellcode dropper based on the direct syscalls (low Level APIs) and we will also analyse it.

- **Chapter 7: Indirect Syscalls**
     - We will create a shellcode dropper based on the indirect syscalls (low Level APIs) and we will also analyse it.

- **Chapter 8: Call Stack Analysis**
     - We will analyse and compare the call stacks of all droppers.

- **Chapter 9: Summary**
     - Workshop summary and closing

- **Bonus Chapter 1: Indirect Syscalls - dynamically retrieve SSNs**
     - We want to further develop our indirect syscall dropper so that the SSNs are no longer hardcoded, but are retrieved dynamically from ntdll.dll at runtime.  

- **Bonus Chapter 2: Indirect Syscalls and hooks by EDRs**
     - What if all or some of our native functions are hooked to an EDR and we are unable to dynamically retrieve the SSN directly? Again, we will have to develop our indirect syscall droppers and use Halos Gate or Freshy Calls to solve that problem. 


## Getting Started
All the step-by-step instructions and code samples can be found in the respective exercise folder. 

### Prerequisites LAB
- **[Windows 10 x64 DEV/LAB machine](https://go.microsoft.com/fwlink/p/?linkid=2195587&clcid=0x407&culture=de-de&country=de)**
    - AV/EPP/EDR disabled
    - [Visual Studio Free 2019](https://visualstudio.microsoft.com/de/vs/older-downloads/)
    - [x64dbg](https://x64dbg.com/)
    - [WinDbg Preview](https://www.microsoft.com/store/productId/9PGJGD53TN86)
    - [Process Hacker](https://processhacker.sourceforge.io/downloads.php)
    - [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)  
    
- [**Kali Linux**](https://www.kali.org/get-kali/#kali-platforms)
    - Metasploit to create shellcode and an MSF-Listener


I hope you will enjoy the workshop and that you can use it to gain a better understanding of sycalls, direct sycalls and indirect sycalls. Have fun with it!

**Happy Hacking!**


Daniel Feichter [**@VirtualAllocEx**](https://twitter.com/VirtualAllocEx), Founder **[@RedOps](https://redops.at/en/) Information Security** 

## Previous work and references
- https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/
- https://0xdarkvortex.dev/hiding-in-plainsight/
- https://0xdarkvortex.dev/proxying-dll-loads-for-hiding-etwti-stack-tracing/
- https://klezvirus.github.io/RedTeaming/AV_Evasion/NoSysWhisper/
- https://www.mdsec.co.uk/2020/12/bypassing-user-mode-hooks-and-direct-invocation-of-system-calls-for-red-teams/
- https://captmeelo.com/redteam/maldev/2021/11/18/av-evasion-syswhisper.html
- https://winternl.com/detecting-manual-syscalls-from-user-mode/
- https://alice.climent-pommeret.red/posts/a-syscall-journey-in-the-windows-kernel/
- https://alice.climent-pommeret.red/posts/direct-syscalls-hells-halos-syswhispers2/#with-freshycalls
- https://www.netero1010-securitylab.com/evasion/indirect-syscall-in-csharp
- https://offensivecraft.wordpress.com/2022/12/08/the-stack-series-return-address-spoofing-on-x64/
- https://offensivecraft.wordpress.com/2023/02/11/the-stack-series-the-x64-stack/
- https://j00ru.vexillium.org/syscalls/nt/64/
- https://github.com/jthuraisamy/SysWhispers
- https://github.com/jthuraisamy/SysWhispers2
- https://github.com/klezVirus/SysWhispers3
- https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-api-list
- https://www.geoffchappell.com/studies/windows/km/index.htm
- "Windows Internals, Part 1: System architecture, processes, threads, memory management, and more (7th Edition)" by Pavel Yosifovich, David A. Solomon, and Alex Ionescu
- "Windows Internals, Part 2 (7th Edition)" by Pavel Yosifovich, David A. Solomon, and Alex Ionescu
