

* Configure a basic (and brand new) Windows 10 Enterprise system for Ansible access

- Requirements: 

Ansible requires PowerShell version 3.0 and .NET Framework 4.0 


- Remove auto-login (This isn't needed but is a good security practice to complete)

Set-ExecutionPolicy -ExecutionPolicy Restricted -Force

$reg_winlogon_path = "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon"
Set-ItemProperty -Path $reg_winlogon_path -Name AutoAdminLogon -Value 0
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultUserName -ErrorAction SilentlyContinue
Remove-ItemProperty -Path $reg_winlogon_path -Name DefaultPassword -ErrorAction SilentlyContinue

WinRM Setup


Once Powershell has been upgraded to at least version 3.0, the final step is for the WinRM service to be configured so that Ansible can connect to it. There are two main components of the WinRM service that governs how Ansible can interface with the Windows host: the listener and the service configuration settings.

Details about each component can be read below, but the script ConfigureRemotingForAnsible.ps1 can be used to set up the basics. This script sets up both HTTP and HTTPS listeners with a self-signed certificate and enables the Basic authentication option on the service.

To use this script, run the following in PowerShell:

$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file



* Install Chocolatey Windows Package Manager using Ansible for Window Only

First, ensure that you are using an administrative shell - you can also install as a non-admin, check out Non-Administrative Installation.
Install with powershell.exe

Run Get-ExecutionPolicy. If it returns Restricted, then run Set-ExecutionPolicy AllSigned or Set-ExecutionPolicy Bypass -Scope Process.
Now run the following command:


Paste the copied text into your shell and press Enter.

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))




* Configure and enable SSH on standard port



The first step to using SSH with Windows is to install the Win32-OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH) service on the Windows host. Microsoft offers a way to install Win32-OpenSSH through a Windows capability but currently the version that is installed through this process is too old to work with Ansible. To install Win32-OpenSSH for use with Ansible, select one of these three installation options:

Manually install the service, following the install instructions from Microsoft.

Install the openssh package using Chocolatey:

choco install --package-parameters=/SSHServerFeature openssh
Use win_chocolatey to install the service:

- name: install the Win32-OpenSSH service
  win_chocolatey:
    name: openssh
    package_params: /SSHServerFeature
    state: present
Use an existing Ansible Galaxy role like jborean93.win_openssh:

# Make sure the role has been downloaded first
ansible-galaxy install jborean93.win_openssh

# main.yml
- name: install Win32-OpenSSH service
  hosts: windows
  gather_facts: no
  roles:
  - role: jborean93.win_openssh
    opt_openssh_setup_service: True


* Upload a configurable SSH key

- name: Set authorized key took from file
  authorized_key:
    user: <username>
    state: present
    key: "{{ lookup('file', '/home/<username>/.ssh/id_rsa.pub') }}"



* Install Git using Chocolatey

Assuming the above instructions have been followed to install Chocolatey, simpley run the following commands to install Git

choco install git.install



https://docs.ansible.com
https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH
https://chocolatey.org/install
