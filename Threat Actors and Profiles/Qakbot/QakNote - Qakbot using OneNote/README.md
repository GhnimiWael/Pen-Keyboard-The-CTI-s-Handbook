Introduction
============


<p align="center" width="100%">
    <img width="100%"src="https://www.bleepstatic.com/content/hl-images/2020/12/09/Qbot.jpg"> 
</p>

The Qakbot malware has been a persistent threat to computer systems for
several years, with its ability to steal sensitive information and
compromise system security. Recently, however, researchers have
identified a new variant of Qakbot that uses the popular note-taking
application, Microsoft OneNote, to evade detection and infect users.

This new variant of Qakbot takes advantage of OneNote\'s ability to
create and run custom add-ins to launch its attack. Once installed, the
malware can steal user data, compromise system security, and even spread
to other devices on the network.

In this report, we will provide an in-depth analysis of the Qakbot
malware that uses OneNote to attack users. We will examine the
techniques and methods used by the malware to evade detection and infect
systems, as well as the impact of the attack on users and their data. We
will also explore techniques for detecting and mitigating this type of
attack and provide best practices for protecting against Qakbot and
other persistent malware threats.

By understanding the behavior and functionality of this new variant of
Qakbot, we can better protect ourselves and our organizations from the
devastating effects of malware attacks. So, let\'s dive in and explore
the world of Qakbot malware and its use of OneNote to compromise users.

Qakbot via OneNote
==================

The initial phase of infection process commences when the recipient
receives a spam email that contains a OneNote attachment.

Upon opening the attachment, an embedded BAT file is deposited and
triggered, initiating the launch of a PowerShell script.

This script subsequently downloads a Qakbot malware DLL, which is
ultimately executed through rundll32.exe.

![](.//media/image6.png)

The diagram below depicts the OneNote-based delivery method utilized by Qakbot via PowerShell.

Qakbot malware is distributed to users through spam emails that contain
a OneNote attachment. The email subject line read for example "Success
Story, Contract Rejected, Contract Accepted".

![](.//media/image17.png)


Once the OneNote attachment is opened by the user, a page displaying a
message is presented. The message is crafted to appear as if it contains
a cloud-based attachment, thereby tricking the user into double-clicking
on it. However, this action initiates the Qakbot infection process.

The figure below displays the OneNote page containing the fraudulent
message.

![](.//media/image8.png)


When the "**[OPEN]{.underline}**" button is clicked on a OneNote page,
it performs a convert action to visit a embedded page, most of the case
written on JavaScript, as mentioned on the Figure 4.

![](.//media/image9.png)


Therefore, the next action will be the dropping BAT file with a random
generated name, for example: "*522381.dat*" or "*aOQSyYHK.bat*"

![](.//media/image10.png)


Upon the execution of the this PowerShell script, a ".dat" file will be
download but as JPG file using *Invoke-WebRequest* command.

The file is then saved as a JPG file in the %programdata% path. However,
the downloaded file is not an actual GIF file but a DLL Qakbot
executable file, which is subsequently run using "Rundll32.exe" with the
"Wind" parameter.

![](.//media/image11.png)


Conclusion
==========

Qakbot malware represents a clear example of the constantly evolving
threat landscape, underlining the importance of remaining vigilant in
the cybersecurity domain.

Its complex structure, extensive impact, and widespread prevalence
reinforce the need for proactive and robust security measures. The
Threat Actors responsible for Qakbot remain highly active. They
consistently adapt their methods to avoid detection and maximize their
gains, using innovative attack vectors such as OneNote attachments to
display their sophistication and ingenuity.

Recommendations
===============

We have listed some essential cybersecurity best practices that create
the first line of control against attackers. We recommend that our
readers follow the best practices given below:

-   Do not open emails from unknown or unverified senders.

-   Avoid downloading pirated software from unverified sites.

-   Use strong passwords and enforce multi-factor authentication
    wherever possible. 

-   Keep updating your passwords after certain intervals.

-   Use reputed anti-virus solutions and internet security software
    packages on your connected devices, including PCs, laptops, and
    mobile devices.  

-   Avoid opening untrusted links and email attachments without first
    verifying their authenticity.   

-   Block URLs that could use to spread the malware, e.g.,
    Torrent/Warez.  

-   Monitor the beacon on the network level to block data exfiltration
    by malware or Threat actors.  

-   Enable Data Loss Prevention (DLP) Solutions on employees' systems.

MITRE ATT&CK® Techniques
========================

![](.//media/image12.png)

IOCs (Indicator of Compromise)
==============================
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
