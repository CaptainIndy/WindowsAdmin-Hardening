# WindowsAdmin-Hardening
Windows Administration and Hardening
**Summary**

This project will go over common actions taken by a Windows Administrator to harden the system. We will be diving into creating domain-hardening Group Policy Objects as well as reviewing some PowerShell fundamentals Before we begin, keep in mind that there will be two different Windows VMs used here that are both being nestled inside an Azure Windows RDP Host Machine. The main Windows 10 Server will mimic the main server used to administer all the group policies. The other Windows 10 VM will be used to mimic a user on the server's domain. 

This activity will be split into various tasks: 
   - Task 1: Create a GPO - Disable Local Link Multicast Name Resolution (LLMNR)
   - Task 2: Create a GPO - Account Lockout
   - Task 3: Create a GPO - Enabling Verbose PowerShell Logging and Transcription
   - Task 4: Create a Script - Enumerate Access Control Lists


**Task 1: Create a GPO - Disable Local Link Multicast Name Resolution (LLMNR)**
   For this first task, I investigated and mitigated one of the attack vectors that exists within a Windows domain, LLMNR.
   Local Link Multicast Name Resolution (LLMNR) is a vulnerability, so you will disable it on your Windows 10 machine (via the GC Computers OU).

A few notes about LLMNR:
- LLMNR is a protocol used as a backup (not an alternative) for DNS in Windows.
- When Windows cannot find a local address (e.g., the location of a file server), it uses LLMNR to send out a broadcast across the network asking if any device knows the address.
- LLMNR’s vulnerability is that it accepts any response as authentic, allowing attackers to poison or spoof LLMNR responses, forcing devices to authenticate to them.
- An LLMNR-enabled Windows machine may automatically trust responses from anyone in the network.
- Turning off LLMNR for the GC Computers OU will prevent your Windows machine from trusting location responses from potential attackers.

  In my Windows Server machine, I created a Group Policy Object that prevents my domain-joined Windows machine from using LLMNR.
  <img width="510" alt="GPOs" src="https://github.com/user-attachments/assets/7bf260bc-696e-4143-9125-3bb087b8e9ea">

**Task 2: Create a GPO - Account Lockout**
For security and compliance reasons, I needed to implement an account lockout policy on my Windows workstation. An account lockout disables access to an account for a set period of time after a specific number of failed login attempts. This policy defends against brute-force attacks, in which attackers can enter a million passwords in just a few minutes.

Important to note: an overly restrictive account lockout policy (such as locking an account for 10 hours after 2 failed attempts), can potentially keep an account locked forever if an attacker repeatedly attempts to access it in an automated way.

Within my nested Windows Server machine, I created another GPO focused on an account lockout. 

<img width="526" alt="Account-Lockout-Policies" src="https://github.com/user-attachments/assets/617bb224-7a32-437d-ab7c-29d1f9c4c8f5">

** Task 3: Create a GPO - Enabling Verbose PowerShell Logging and Transcription**

In this task, I am going to use a PowerShell practice that is recommended regardless of whether PowerShell is enabled or disabled: enabling enhanced PowerShell logging and visibility through verbosity. Best practices for enabling or disabling PowerShell are debated. This often leads to the solution of allowing only certain applications to run. This is why this option is better. 

For this task, I am working in my Windows Server machine. I am creating a Group Policy Object to enable PowerShell logging and transcription. This GPO will combine multiple policies into one, although they are all under the same policy collection.

Important steps to follow in the proper Windows Powershell policy Group Policy Management Editor:
  - Enable "Turn on Module Logging" and enter an asterik (wildcard) for the Module Names field.
  - Enable the "Turn on Powershell Script Block Logging" policy, cgecking the "Log script block invocation start/stop events" setting.
  - Enable the "Turn on Script Execution" policy and set "Execution Policy to Allow all scripts".
  - Enable the "Turn on Powershell Transcription" policy and leave the Trasncript output directory blank (defaults to user's ~\Documents directory). Also check the "Include invocation headers option" (adds timestamps to command transcription.
  - Leave the "Set the default source path for Update-Help policy" as "Not configured".
  - Link this new "PowerShell Logging" GPO to the GC Computers OU.
  - Lastly, The next time I log in to your Windows 10 machine, I will want to run gpupdate. Then launch a new PowerShell window and run a script. Here we will see verbose PowerShell logs created in the Windows 10 machine directory for the user that ran the script: C:\Users&lt;user&gt;\Documents.
    
<img width="491" alt="Windows-PowerShell-Policies" src="https://github.com/user-attachments/assets/eef35276-72e1-409b-b263-998ea36da557">


**Task 4: Create a Script - Enumerate Access Control Lists**





