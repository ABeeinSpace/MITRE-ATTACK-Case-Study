# Mapping an Attack Lifecycle to Lockheed's Cyber Kill Chain 
**Author:** *Aidan Border*

## Abstract

This report is a case study examining an example attack's lifecycle and how the attacker's attempts to gain entry can be mapped to Lockheed Martin's Cyber Kill Chain[^1]. I will also map relevant techniques, tactics, and procedures (TTPs) to MITRE's ATT&CK matrix[^2].

The attack I am examining was an intrusion attempt against the Metasploitable 2 virtual machine. This machine was created based on an intentionally vulnerable image released by the Metasploit team to allow researchers to practice using the Metasploit framework. 

I will examine each stage in the attack utilizing the Cyber Kill Chain and how a defender could have mitigated the tactics used at each stage.

> The headings in this document will follow the Lockheed Cyber Kill Chain as closely as possible. However, this attack does not map to the Cyber Kill Chain exactly.

## Reconnaissance
*Tactic used: Reconnaissance*

*Techniques used: Active Scanning, Gather Victim Host Information*

Cyber attacks open with the reconnaissance phase. What information can the attacker gather about the victim's network? If they're targeting a specific individual, can the attacker figure out the victim's leadership structure for later use in a spearphishing attack?

In this attack, the adversary opened with active scanning tools to try to gather some information on the Metasploitable VM. Nessus Essentials was deployed from the attacker's machine to profile services running on the machine and potential vulnerabilities. Of the services running on the machine, the majority had severe vulnerabilities.

## Weaponization and Delivery
*Tactic used: Initial Access*

*Techniques used: Exploit Public-Facing Application*

Initial access was achieved by leveraging a backdoor in UnrealIRC version 3.2.8.1. An msfvenom payload was used to pop a reverse shell on the target VM.
This intrusion maps to MITRE ATT&CK [T1190](https://attack.mitre.org/techniques/T1190/), Exploiting a Public-Facing Application. The attacker used the publicly available IRC server to break onto the VM and establish an initial foothold.

### Mitigations

This is the defenders' first opportunity to stop the attack. For this particular intrusion method, keeping UnrealIRC up to date would have stopped the attacker progressing down kill chain. The VM in question was also running a very old version of Ubuntu. Updating to a supported version of the operating system would have also made this intrusion vector harder to exploit.

## Installation
*Tactic Used: Persistence*

*Techniques used: Account Manipulation > SSH Authorized Keys*

Using the access gained from the UnrealIRC backdoor, as well as an additional backdoor discovered running on port 1524, the attacker was able to insert an RSA public key into the root user's authorized_keys file. This granted the attacker full access to SSH into the server as the root user. This phase of the attack maps to MITRE ATT&CK [T1098.004](https://attack.mitre.org/techniques/T1098/004/), Account Manipulation > SSH Authorized Keys.

### Mitigations

The defenders should have been able to stop this phase of the attack a long time ago. A running backdoor on a system should be caught using regular vulnerability scanning. Vulnerability assessment softwares include Tenable's Nessus and Greenbone's OpenVAS. Either of those products would have alerted on the backdoor being present and given the security team a wealth of vulnerability information they could use to ensure the security of their environment and the safety of their customer's data.


## Post-Exploitation
*Tactic Used: Credential Access*

*Techniques used: OS Credential Dumping > /etc/passwd and /etc/shadow*

*Brute Force > Password Cracking*

Using the Secure Copy utility, the attacker was able to grab the password database from the target machine. The attacker then used the password-cracking utility John the Ripper to brute force passwords.

### Mitigations

If the defenders notice that the password database has been leaked, through file access logs or otherwise, they should immediately invalidate passwords for highly privileged accounts (highly-privileged accounts may include the root user itself and any account that has `sudoers` access at a minimum). 

Credentials like SSH keys and other cryptographic keys should be rotated and the system checked for accounts that should not be present. If found, such accounts must be disabled or removed to prevent the attacker from being able to jump right back onto the system. 

[^1]: [Lockheed Martin Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)
[^2]: [MITRE ATT&CK](https://attack.mitre.org/)
