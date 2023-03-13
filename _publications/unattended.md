---
title: "unattended"
collection: publications
permalink: /publication/unattended
excerpt: 'This is a free room from Tryhackme'
date: 2023-13-03
venue: '13-03'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/unattended.png)

**Link:** [Unattended](https://tryhackme.com/room/unattended)

## Description : 

* Investigate a user activity between 12:05 PM to 12:45 PM on the 19th of November 2022.

* Figure out what files were accessed and exfiltrated externally.
---

<span style="color: red;">TASK3: Snooping around  </span>

>What file type was searched for using the search bar in Windows Explorer?

>What top-secret keyword was searched for using the search bar in Windows Explorer?

* You can use RegistryExplorer tool to check the **Windows Explorer Search bars**

* You can find this information in :
**NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths**
OR
**NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery**

* Load the **NTUSER.dat** hive

![unattended4](/images/unattended4.png)

![unattended01](/images/unattended01.png)

---

<span style="color: red;">TASK4: Can't simply open it  </span>

>What is the name of the downloaded file to the Downloads folder?

>When was the file from the previous question downloaded? (YYYY-MM-DD HH:MM:SS UTC)

* Open Autopsy and select Logical Files as data source type

![unattended1](/images/unattended1.png)

* Select 'C' file :



![unattended2](/images/unattended2.png)

* Now you can check the web downloads 
![unattended3](/images/unattended3.png)

>Thanks to the previously downloaded file, a PNG file was opened. When was this file opened? (YYYY-MM-DD HH:MM:SS)

* Switching back to Registry Explorer, you can search for **.png**

![unattended5](/images/unattended5.png)

<span style="color: red;">TASK5: Sending it out  </span>

>A text file was created in the Desktop folder. How many times was this file opened?

>When was the text file from the previous question last modified? (MM/DD/YYYY HH:MM)

* You can use **JLECmd** tool 

```bash
JLECmd.exe -d c:\Users\THM-RFedora\Desktop\kape-results\C\Users\THM-RFedora
```

![unattended6](/images/unattended6.png)


>The contents of the file were exfiltrated to pastebin.com. What is the generated URL of the exfiltrated data?

* Switching back to Autopsy, you can find the answer under *Web History* section

![unattended7](/images/unattended7.png)

>What is the string that was copied to the pastebin URL?

![unattended8](/images/unattended8.png)

