1. Prerequisites
    Target OS & Version: Domain Controller: Windows Server 2022.

    Control Machine: Ubuntu 22.04 LTS.

    Required Packages (Control Machine):
        Ansible Version: 2.16 or 2.17.
        Collections: ansible.windows, microsoft.ad.
        Python Libraries: pywinrm for Windows connectivity.

    Network Requirements:
        Bidirectional connectivity between the Ubuntu control node and Windows Server.
        Windows Firewall must be disabled during the setup phase to allow WinRM traffic.

    Manual Setup: Powershell must be configured for Ansible remoting on the Windows target.

2. Installation & Configuration
    Step 1: Prepare the Windows Target
        On the Windows Server 2022 machine, open PowerShell as Administrator and run the following commands to enable Ansible management:
        PowerShell
        # Set Security Protocol to TLS 1.2
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

        # Download and run the Ansible configuration script
        $url = "https://raw.githubusercontent.com/ansible/ansible-documentation/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
        Invoke-WebRequest -Uri $url -OutFile setup.ps1
        .\setup.ps1

        # Disable the Firewall for initial deployment
        Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled False
    
    Step 2: Configure the Ubuntu Control Node
        Execute these commands on your Ubuntu machine to install the necessary environment:

        Update and install Ansible:

        Bash
        sudo apt update && sudo add-apt-repository --yes --update ppa:ansible/ansible
        sudo apt install ansible -y

        Install Collections and Dependencies:

        Bash
        ansible-galaxy collection install ansible.windows microsoft.ad --force 
        pip install pywinrm 

        Prepare Playbook:

        Bash
        git clone https://github.com/jnl1479/Vulnerable-ActiveDirectory-Ansible.git
        cd Vulnerable-ActiveDirectory-Ansible
        # Edit inventory.ini with your Windows Server IP and credentials
        # Edit vars/variables.yml with custom user credentials
    
    Step 3: Execution
        Verify connectivity and run the deployment:

        Ping Test: ansible windows -m win_ping.

        Run Playbook: ansible-playbook -i inventory.ini playbook.yml.

        Troubleshooting: If the ping or playbook fails, append -vv to the command to see detailed error logs regarding WinRM authentication or network timeouts.

3. Verification Steps
    To confirm the Arasaka infrastructure is correctly deployed and vulnerable:

    Service Status: On the Windows Server, verify the Active Directory Domain Services are running using Get-Service adws, dns, kdc.

    Verify Vulnerable User (rbartmoss): Check that the "Runner" account exists and allows login without a password.

    Command: Get-ADUser -Identity rbartmoss -Properties PasswordNotRequired

    Expected Value: PasswordNotRequired: True.

    Verify Vulnerable Executive (sarasaka): Ensure the CEO account is set for AS-REP Roasting.

    Command: Get-ADUser -Identity sarasaka -Properties DoesNotRequirePreauth

    Expected Value: DoesNotRequirePreauth: True.

    Access Test: From the Ubuntu machine, attempt to list the sarasaka account details using crackmapexec or standard LDAP tools to confirm the managedBy hint is visible in the attributes.