# Qakbot

If you are a cybersecurity professional looking to enhance your knowledge of Qakbot or strengthen your organization's defenses against this malware, the Qakbot Github repository is a must-visit resource. Check it out today to stay up-to-date with the latest information on this dangerous Trojan virus.

Also Known As: qakbot, QuackBot, win.qakbot, QakBot, QBot, Pinkslipbot

  * [IOCs](#iocs)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/CVE.txt">1. CVEs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-cvetxt--1-cves--a-)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/EMAIL.txt">2. EMAILs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-emailtxt--2-emails--a-)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/HASH.txt">3. HASHs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-hashtxt--3-hashs--a-)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/HOSTNAME.txt">4. HOSTNAMEs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-hostnametxt--4-hostnames--a-)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/IP.txt">5. IPs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-iptxt--5-ips--a-)
    + [<a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/URL.txt">6. URLs</a>](#-a-href--https---githubcom-ghnimiwael-a-cti-diaries-blob-main-malwares-qakbot-iocs-urltxt--6-urls--a-)
    
<p align="center" width="100%">
    <img width="100%"src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FH7rTfY2uPKTawTcEUhnW%2Fuploads%2F79ZvzPXyCZ6lfg5OWxzF%2Fimage.png?alt=media&token=cb5a1035-b984-4831-b12b-9c7cba22dde5"> 
</p>


## What is Qakbot ?

Qakbot, also known as Qbot, is a type of malware that is designed to steal sensitive information from infected computers. Qakbot is a Trojan virus, which means it is designed to hide its presence on an infected computer and operate silently in the background.

Qakbot typically spreads through spam email messages that contain a malicious attachment or link. Once the attachment or link is opened, Qakbot downloads and installs itself on the infected computer. It can also spread through network shares and by exploiting vulnerabilities in software installed on the computer.

Once Qakbot infects a computer, it can perform a variety of malicious actions. It can steal login credentials for online banking and other financial websites, record keystrokes to capture sensitive information such as credit card numbers and passwords, and download additional malware onto the infected computer.

Qakbot can also spread to other computers on the same network, allowing it to infect entire organizations. Qakbot is a persistent threat and can be difficult to remove once it has infected a computer or network. It is important to use up-to-date antivirus software and to regularly update software to patch vulnerabilities that Qakbot can exploit.

## IOCs
<p align="center" width="100%">
    <img width="100%"src="https://i.imgur.com/XQR2Zc9.png"> 
</p>

In the context of cybersecurity, IOC stands for "Indicator of Compromise". An IOC is a piece of evidence or artifact that suggests a computer system or network has been breached or compromised by an attacker.

Examples of IOCs include:
- Malicious IP addresses or domains that are associated with command-and-control (C2) servers used by malware to communicate with attackers.
- Malware signatures or hashes that are associated with known malware strains or variants.
- Anomalous network traffic or patterns of behavior, such as unusual data exfiltration or system login attempts.
- Suspicious files, such as executables or scripts, that are found on a system and have not been authorized by the organization.

By identifying IOCs, security teams can investigate and respond to security incidents more quickly and effectively. IOCs can be used to detect ongoing attacks, contain their impact, and prevent similar attacks from occurring in the future. Security professionals can collect and analyze IOCs from a variety of sources, including threat intelligence feeds, security tools, and logs from network and endpoint devices.

here's a brief description of each IOC file related to Qakbot that could be included on a Github repository:

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/CVE.txt">1. CVEs</a>
This file contains a list of Common Vulnerabilities and Exposures (CVEs) associated with Qakbot. These vulnerabilities can be used by attackers to gain access to vulnerable systems and deploy Qakbot or other malware.

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/EMAIL.txt">2. EMAILs</a>
This file contains a list of email addresses associated with Qakbot, such as those used to send spam or phishing emails to distribute the malware.

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/HASH.txt">3. HASHs</a>
This file contains a list of file hashes associated with Qakbot. These hashes can be used to identify files that are associated with the malware and to detect and remove them from infected systems.

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/HOSTNAME.txt">4. HOSTNAMEs</a>
This file contains a list of domain names and hostnames associated with Qakbot. These domain names and hostnames can be used to identify command-and-control (C2) servers used by the malware to communicate with attackers.

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/IP.txt">5. IPs</a>
This file contains a list of IP addresses associated with Qakbot. These IP addresses can be used to identify C2 servers or other systems that are part of the Qakbot infrastructure.

### <a href="https://github.com/GhnimiWael/A-CTI-diaries/blob/main/Malwares/Qakbot/IOCs/URL.txt">6. URLs</a>
This file contains a list of URLs associated with Qakbot. These URLs can be used to identify websites or web-based applications that are used to distribute the malware or to host exploit kits used to deliver Qakbot payloads.

## Reversing
### 1. <a href="https://github.com/GhnimiWael/A-CTI-diaries/tree/main/Malwares/Qakbot/Qakbot-DLL-Stager-Reverse">Qakbot DLL Stager Reversing</a>
<p align="center" width="100%">
    <img width="100%"src="https://i.imgur.com/KTrEQ46.jpg"> 
</p>

"Reverse Engineering of Qakbot DLL Stager" is a new title that focuses on analyzing the behavior and functionality of the Qakbot malware's DLL stager component. This component is used to load Qakbot into memory and execute it on an infected system. The book provides an in-depth analysis of the DLL stager, including its code structure, functionality, and methods used to evade detection. It also explores techniques for detecting and reversing Qakbot infections, as well as best practices for protecting against this persistent and dangerous malware. Whether you are a seasoned reverse engineer or a newcomer to the field, "Reverse Engineering of Qakbot DLL Stager" provides valuable insights and knowledge for anyone looking to enhance their understanding of Qakbot and the techniques used by malware authors to evade detection and compromise computer systems.
