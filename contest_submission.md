**Need to develop a hiddenprocess plug-in**

Volatility is a widely used tool among forensic investigators for performing memory forensics. There are two versions of Volatility: versions 2 and 3. However, development and support for version 2 have ceased. Consequently, Volatility 2 is not suitable for analyzing the latest operating systems, such as Windows 11, because it fails to recognize the necessary profiles for memory analysis. Therefore, forensic investigators are compelled to use Volatility 3 when examining the most recent operating systems. Despite its advancements, Volatility 3 lacks several plugins that were available and useful in version 2.
A notable example of such a plugin is 'psxview,' which is used to detect hidden and terminated processes, including rootkits. This plugin was instrumental for investigators in efficiently uncovering a wealth of information about concealed processes. However, the absence of the 'psxview' function in Volatility 3 poses a significant challenge to analysis. To address this, I have developed a method for detecting hidden processes in a Volatility 3 environment. I believe this advancement will greatly assist investigators in efficiently conducting memory forensics to identify malicious processes. With these capabilities, I am confident that my project would make a substantial contribution to the field, which is why I have entered it in the contest.

**What is hiddenprocess plug-in and how do use it?**

To develop a hiddenprocess, existing plug-ins had to be modified. <Table 1> below compares the results of the hidden process developed this time with the existing psxview.

![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/ee7534ef-d8f4-49aa-afb2-54e4562c409a)

Among them, the function of thrdproc was produced using the code uploaded to the volatility3 foundation pull requests. (https://github.com/volatilityfoundation/volatility3/pull/960),Other plug-in codes I modified include sessions, privileges, etc. Like the existing psxview, the hidden process plug-in has the ability to detect hidden processes by comparing various results with each other.

[Figure 1] is the screen where I executed the plug-in I developed.

[Figure 1] Hiddenprocess plug-in execution screen

![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/45893654-ac6d-4fa0-ba65-b043554f5f88)

As shown in the figure 1, pslist, psscan, Thdscan, process execution time, and process end time can be used to compare whether the process is hidden or not. My plug-in has the ability to detect existing hidden and terminated processes, as well as the name of the user account that ran the process and the permission of the executed process.
If you also want to check the permissions that the process has, you should use the --privilege option.
It is important to note that to use this option, you must first run privilegesmodify, a modified version of privileges plug-in. Running this plug-in automatically creates a text file named systemprivileges_pid and outputs the permissions the process has based on these results when the --privilege option is entered.
[Figure2] below is the result of executing the privileges modify plug-in. This plug-in was designed to select and compare three of the 35 privileges inquired by the existing privileges plug-in.

[Figure 2] privilegesmodify plug-in execution screen


![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/0f2e5d4b-4274-48ab-8c69-9fd27441b515)

Finally, using the hidden process I developed, I can detect hidden processes and processes that have taken over the rights of the system, such as Local Privilege Escalation (LPE).



**Hidden process function verification**

In order to verify the function of the hiddenprocess, I built an environment for infringement based on the scenario.(CVE 2023-21768 is an LPE vulnerability that elevates general processes to system process privileges.)

![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/e1de2899-49f7-4b17-8052-3acef2d3d6fd)

Victim Operating System : Windows 11 22H2 / OS 22621.525 , RAM : 7.8GB

Investigator Operating System : Windows 11 22H2 , Volatility 3-2.4.1 




**Scenario Description** 

The hacker accesses the victim's system through a reverse shell (user authority). After that, they use cronos-rootkit to hide their traces to hide their processes. In addition, CVE-2023-21768 is used to steal system rights from user rights and hide the traces. 
The analyst secures the memory dump file and analyzes the infringement system using the hidden process of Volatility 3. As a result of the analysis, processes hidden by root-kit and processes running with excessively high authority are detected.

![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/5bceaed8-dbe6-44a3-9077-0a82db71da7b)

Finally, the process to be detected in this scenario is shown in <table 2> The analyst executes privileges modify plug-in as in <Figure 3> to analyze memory dump files with hidden processes.



![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/c7b122aa-f393-4bbf-85fa-806461f7db06)

When you run privileges modify plug-in, which is modified to look up only three high-risk privileges in the existing privileges plug-in, the results are automatically stored in the systemprivileges_pid file, and when you use the privilege option in hidden processes, the process's privileges are output together based on the results.




![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/523dabf9-22a9-49b6-8d90-5c659ece6147)

When we output the hiddenprocess plug-in with the privilege option, we can see that the hidden process, cmd.exe (PID:11860), is not detected by pslist and thrdscan, as shown in <Figure 4>, so we can proceed with a deeper analysis. Additionally, we can see that the parent process of cmd.exe is shell.exe. Since cmd.exe's parent process, shell.exe (PID:14316), has an Exit Time of N/A, we can conclude that it is a suspicious process because it is not detected in pslist even though it is running and its privileges are shown as High. In our experiments, we found that Volatility 3 can detect both processes that are hidden using hiddenprocess and processes that have escalated privileges through local privilege escalation (LPE) vulnerabilities on the Windows 11 operating system.


**Conclusion**

With the end of development and support for volatility 2, I have noticed that many investigators are frustrated by the lack of plugins to detect hidden processes like psxview for newer operating systems like windows 11. To address this inconvenience, I developed a plugin to detect hidden processes in the Volatility 3 environment. In addition to detecting hidden processes, my plugin searches the username of the process and the privileges of the process, allowing investigators to intuitively identify hidden processes, malicious processes that have stolen system privileges by exploiting LPE vulnerabilities, and processes with excessive privileges. In addition, to verify the functionality of my plugin, I built a breach environment based on a scenario to verify the effectiveness of the tool, and as a result, I was able to detect both hidden processes and processes that exploited privilege escalation vulnerabilities normally.
I hope this plugin will help many investigators detect and view malicious processes.


**Future research**

Among the many processes inquired through future research, we plan to develop a "process dataset" that helps to distinguish between malicious and normal processes easily. This dataset was inspired by detecting malware through conventional hash values.
Among the numerous processes inquired by volatility, the characteristics of the normal process will be organized into data sets so that analysts can quickly select processes that do not require analysis, such as normal and system processes.


**Attachment**

While developing a plug-in that can detect hiddenprocesses in volarity 3, the results were summarized in a table<Table 3><Table 4> so that they could be interpreted. What is written in this table is not absolute and can be interpreted differently depending on the situation to be analyzed.

![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/a1608fd9-f823-49a6-984f-7f4678a4c45c)
![image](https://github.com/resposo/volatility-3-contest-2023/assets/86822730/893a38cb-88c5-450a-b259-fe7ec6641ccf)

