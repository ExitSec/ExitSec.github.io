---
layout: post
title: FireEye - Commando VM Review
category: Misc
---
# Introduction
As many penetration testers know, Kali is the go to operating system of choice....unless you make your own **cough** arch **cough**, however, FireEye released the Commando VM. For me, Windows has its place within the penetration testing world as many large scale enterprises run Windows with Active Directory as its backbone. So why has it taken someone so long to make a Windows Offensive Platform. Well lets find out if the Commando is what I have been hoping for as an alternative to Kali. 

To clarify this blog post will discuss the operating system from a high-level view looking at its installation, updating and tooling. For anyone that would like a further read on FireEye's Commando VM please see the following link: <https://www.fireeye.com/blog/threat-research/2019/03/commando-vm-windows-offensive-distribution.html>. 



# Install & Updating
In this build I will be using a standard Windows 10 Pro install, with all security updates as a base, sitting on a VMWare ESXI server.
The installer requires a full up to date Windows Operating System (Windows 7 or 10) before being able to continue with the installer. Furthermore, the installer provides an extremely appreciated and commonly overlooked suggestion in taking a VM Snapshot before continuing. Once the installer is up and running you just need to sit back and wait for the tools to be installed, this can take a while to finish. The installers backbone sits with [BoxStarter](https://boxstarter.org/), a Windows environment installer that utilises the [Chocolatey](https://chocolatey.org/) Windows package manager to install all of our favourite tooling as seen in Kali.
![Installer](/images/command_vm/install.png "Installer")

You will experience one or two restarts before entering the good part of the installer, where all the tools will be installed. There is a myriad of tooling that is provided ranging from Network Recon all the way down to step 6 and 7 of the kill-chain for C2.
![Tools](/images/command_vm/tools.png "Tools")

A full list of the tooling can be found on the Command VM README page: <https://github.com/fireeye/commando-vm/blob/master/README.md>

Once the environment has been installed, your background and will have changed into the Flare Ninja along with the command prompt asking you to exit.
![Installer](/images/command_vm/complete.png "Complete Install")

Before breaking out NMAP and ensure everything is up to date by running the following command in an Administrative Terminal:
```
cup all
```
![Updating](/images/command_vm/updating.png "Updating")

One thing to note, is that most Powershell scripts will output a log file for you to keep track off. This can be found on the Desktop under Transcript.
![Powershell](/images/command_vm/ps_transcript.png "PowerShell Transcript")

# The Environment
My first instinct after install is what tooling is available. Within Commando there is a significant amount of them, including standard sysinternals and Active Directory applications.
![Tooling](/images/command_vm/tooling.png "Tooling")

On the outset, there are many similarities to that of Kali. The pinning of cmder on the taskbar already makes me feel more at home coming from Kali, as well as the dark theme outset.

The tooling that is available from the outset provides an interface for all parts of the [Cyber-Kill Chain](https://www.lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf), which is great for red teaming of for those times penetration tests that have a wider scope than most, we are looking at you Web Apps! This is with the bonus of providing all the tools required for those softer skills that are needed when carrying out any test or possibly any examination in regard to penetration testing e.g., OSCP. The integration of applications such as yed, greenshot and screen2gif allow any tester for the interface between evidence capture to report writing.

![Administrative](/images/command_vm/admin.png "Administrative tooling")

If an application is missing from the substantial arsenal that is the Commando Toolkit, then the simplicity of chocolatey is at your fingertips. Although OpenVPN is installed below is a quick example of how to install a new application through the terminal.

```
chocolatey install openvpn
```
and you will be all set to carry on your penetration testing with the added warmth of whatever application you may install.

![OpenVPN](/images/command_vm/openvpn.png "OpenVPN")

However, there is much to be desired within the Web Application tooling. The only three tools (excluding Firefox) that are provided are Fiddler, Burp Suite Free and OWASP Zap.
![WebTools](/images/command_vm/web_apps.png "Web Application Tooling")

An improvement within this area could be made by adding some further fundamental Web Applications tools such as:
* SQLMAP
* Dirbuster/GoBuster
* Nikto
* Arachni 

The justification for this is that although Web Applications do have a larger investment in its toolkit, Web Applications are in large demand within all areas of security ranging from CTFs to the full-blown red team engagements.

# Conclusion
FireEyes Commando-VM demonstrates that there is a definite niche in the community for some form of 'offensive' Windows-based OS. With the very up to date tooling, which includes a lot of Windows orientated scripts that many won't of seen, and the reliance on the Windows Powershell backbone, Commando creates an impressive footprint in the security communities OS installation of choice. Furthermore, Commando provides a bridge between the 'lets hack all the things' scenario with the 'I am still getting paid at the end of the day' report writing, which targets all scopes of security enthusiasts. Large portions of the tools revolves around  red teaming and there is much to be desired in certain areas such as Web Apps, however, Commando has set down the gauntlet for Kali to update their tooling and overall implementation to stay ahead of the curve.