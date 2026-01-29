# Windows-RDP-brute-force-attack-simulation-and-defensive-hardening 
This project demonstrates a Windows RDP brute-force attack simulation and defensive hardening in a controlled lab using Kali Linux (attacker VM on Hyper-V) and a Windows 10 “server” (target). The lab workflow includes:

- Build the attacker environment: install Kali Linux in Hyper-V by downloading the ISO, creating a VM, installing, and rebooting into Kali. 

- Prepare the target for defense: enable Windows Firewall and create an inbound rule to allow RDP on a custom TCP port 45931. 

- Hardening change: update Local Security Policy to restrict NTLM by setting “Restrict NTLM: Incoming NTLM traffic” to Deny all accounts. 

- Deploy IPBan: verify PowerShell version (5.1+), install IPBan, then configure allowlisting and reduce the default “failed logins before ban” threshold from 5 to 2–3. 

- Attack validation: confirm connectivity and services, run Nmap to assess RDP exposure, then perform a Hydra dictionary attack using user.txt and pass.txt. 

- Outcome: IPBan detects repeated failures and bans the attacker IP, with evidence in IPBan logs.

---
# 🛡️ RDP Brute-Force Attack & Defense Lab (Kali + Windows + IPBan)

A hands-on cyber security lab that simulates a brute-force/dictionary attack against Windows Remote Desktop Protocol (RDP) and validates defensive controls using **IPBan** to detect repeated failures and automatically block offending IPs.

> ⚠️ Ethical Use Only  
> This project is for **authorized lab environments** and defensive learning. Do not run attacks against systems you do not own or explicitly have permission to test.

---

## 🎯 Objectives

- Build an attacker VM (Kali Linux) on Hyper-V
- Validate RDP exposure and connectivity via scanning
- Execute a controlled Hydra dictionary attack against RDP authentication
- Deploy and tune **IPBan** on Windows to automatically detect and ban brute-force attempts
- Capture evidence of detection and mitigation from logs

---

## 🧱 Lab Topology

- **Target (Windows 10 server):** `192.168.0.1`
- **Attacker (Kali Linux):** `192.168.0.3`

# (A) Install Kali Linux on Hyper-V (Attacker VM)

To install Kali Linux on Hyper-V (Draft), you can follow these steps:       
1. Download the Kali Linux ISO: 
   You can download the latest Kali Linux ISO image from the official Kali Linux website: https://www.kali.org/downloads/

   <img width="544" height="408" alt="image" src="https://github.com/user-attachments/assets/0c1ecf71-063d-4b1f-99fc-a9399cb64f4a" />

 2. Create a new virtual machine in Hyper-V: 
   - Open Hyper-V Manager. 
   - Click on "Action" in the menu bar and select "New" -> "Virtual Machine". 
   - Follow the wizard to create a new virtual machine. Make sure to allocate enough resources (CPU, RAM, disk space) for the virtual machine. 
  
<img width="531" height="414" alt="image" src="https://github.com/user-attachments/assets/ccede7a1-5fa1-4619-afd9-52bff900ef93" />
<img width="527" height="316" alt="image" src="https://github.com/user-attachments/assets/3e929316-0b85-4968-a7f1-efe6ae06e71f" />
<img width="538" height="367" alt="image" src="https://github.com/user-attachments/assets/9d2fc6cb-1a80-472f-b19d-b561a5bc1f9b" />
<img width="531" height="320" alt="image" src="https://github.com/user-attachments/assets/8c8ffdda-5e32-4f32-8f83-25ecf28675c8" />
<img width="527" height="320" alt="image" src="https://github.com/user-attachments/assets/08445e95-d331-483d-b4e7-0867f9ce4b59" />
<img width="533" height="364" alt="image" src="https://github.com/user-attachments/assets/bca450bf-1943-462a-bd62-4f3125710a90" />
<img width="556" height="334" alt="image" src="https://github.com/user-attachments/assets/6b73a714-4602-4377-b28c-2376af02211d" />


3. Install Kali Linux on the virtual machine: 

- Start the virtual machine.

<img width="634" height="466" alt="image" src="https://github.com/user-attachments/assets/dc049ced-e746-4db4-bae7-ef4aaa215b02" />

Follow the on-screen instructions to install Kali Linux. You can install it alongside Windows, erase the disk, and install Kali Linux. 

<img width="658" height="398" alt="image" src="https://github.com/user-attachments/assets/70a47a73-2a47-4215-92cc-a3c11d0f5463" />

4. Complete the installation: 
  - Once the installation is complete, reboot the virtual machine.
<img width="940" height="506" alt="image" src="https://github.com/user-attachments/assets/93aa7c68-6531-4f43-a2a9-eb82517de728" />
  
  - Log in to Kali Linux using the credentials you set up during the installation process.

<img width="940" height="507" alt="image" src="https://github.com/user-attachments/assets/df9b52f7-c90a-474c-94b5-6fc5a9277b2b" />

  - Kali Linux installation Complete

<img width="798" height="530" alt="image" src="https://github.com/user-attachments/assets/9cd3fba9-8c99-4c83-ba36-2c1b9bf4a09f" />

   - That's it! You should now have Kali Linux running on Hyper-V. Remember to keep your Kali Linux system updated with the latest security patches and updates.

--- 

# (B) Develop an attack and defence activity. 

Configure Windows RDP Access & Firewall (Target)

- Ensure Windows Firewall is ON

- Create an Inbound Rule to allow RDP on the configured port

Protocol: TCP

Port: 45931 (as per lab configuration)
    
<img width="940" height="649" alt="image" src="https://github.com/user-attachments/assets/91599f22-c2bf-4130-ab05-86b1dc532d8b" />

- Click on inbound rules, right click for a new rule, click on port and configure as shown in the screenshot above (TCP, Port 45931). 

<img width="940" height="649" alt="image" src="https://github.com/user-attachments/assets/bba6f3e4-c17c-492e-8d93-3d3a113dde0b" />

- Screenshots RDP is allowed on port 45931 in the Windows firewall settings.

<img width="940" height="651" alt="image" src="https://github.com/user-attachments/assets/e19df9ca-f710-4215-b41d-ee91b91bba62" />

- Ensure Windows firewall is turned on after the above configuration.

---

# (C) Apply Hardening Control (Target)

Update Local Security Policy:

<img width="940" height="597" alt="image" src="https://github.com/user-attachments/assets/5f8367a5-8448-45d3-b4d6-7050da830210" />

- Network security: Restrict NTLM: Incoming NTLM traffic → Deny all accounts

This reduces reliance on older authentication mechanisms and aligns with stronger hardening posture.  

---

# (D) Install and Configure IPBan (Target) 

Run PowerShell as Administrator

<img width="788" height="414" alt="image" src="https://github.com/user-attachments/assets/689e8ac5-d1f8-4a8a-90fb-e63c0568af19" />

Confirm PowerShell version:

- $PSVersionTable
 
<img width="940" height="723" alt="image" src="https://github.com/user-attachments/assets/349a3260-be8b-428b-94ac-73c0fae1e237" />

Install IPBan 

- Copy and paste this link into a webshttps://github.com/jjxtra/IPBan, on the github.com website copied clipboard this command 

<img width="940" height="561" alt="image" src="https://github.com/user-attachments/assets/e1154d62-f3cf-4910-b571-af6d7fe68591" />

sudo -i; bash <(wget -qO- https://raw.githubusercontent.com/DigitalRuby/IPBan/master/IPBanCore/Linux/Scripts/Install.sh) and paste into PowerShell and press enter to run the command. 

<img width="1007" height="570" alt="image" src="https://github.com/user-attachments/assets/761f1716-997b-4014-bd98-7096ebbcfb3e" />

- The screenshot above shows IPBan installation complete. 

<img width="789" height="319" alt="image" src="https://github.com/user-attachments/assets/be68256c-2b8e-4d38-99d8-da00d55eaf9c" />

Open IPBan config:

- Configure whitelist/allowlist for trusted admin IPs

<img width="940" height="543" alt="image" src="https://github.com/user-attachments/assets/0896f1bc-f3ac-4e96-9891-30ca30c6e777" />

Tune brute-force threshold:

- Reduce failed logins before ban from 5 to 2–3 for quicker enforcement
 
(A whitelist - allow-list is a cybersecurity strategy that approves a list of email addresses, IP addresses, domain names or applications, while denying all others. IP whitelisting is when you grant network access only to specific IP addresses)

<img width="940" height="534" alt="image" src="https://github.com/user-attachments/assets/22b5e38d-759d-4cce-807d-6958f9c28f50" />

<img width="792" height="538" alt="image" src="https://github.com/user-attachments/assets/ae92c2f7-0aee-4148-a807-ee4e8f69c9b1" />

- it is recommended to actually change it to two (2) or three (3) as shown below.

---

# (E) 🔍 Attack Simulation

The IPBan service will monitor log events on the server, detect suspicious activity, and dynamically block IP addresses that exceed the configured thresholds.

<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/6451cb47-b99a-42d4-991d-1969fc7d72b2" />

- The IPBan is running correctly (as seen in the screenshot above)

 Activity 
1.	A Remote Desktop Protocol (DRP) login will be performed to verify the functionality of the IPBan. 
- IP addresses for target machine (Windows 10 server) 192.168.0.1
- IP address for attack machine (Kali linux) 192.168.0.3 
- Note (Established communications already exist between the attack machine (Kali Linux) and the target machine (Windows 10 server) form the previous exercise)

2.  Confirm IP addressing

- Kali: ifconfig

- Windows: ipconfig

<img width="638" height="469" alt="image" src="https://github.com/user-attachments/assets/2c60c202-7684-4a55-b1c4-bdecdecb8243" />

- Ifconfig command shows IP address for the kali linux

<img width="631" height="490" alt="image" src="https://github.com/user-attachments/assets/300a65fc-8459-4b91-8449-dc39e438b527" />

- Ipconfig command shows IP address for the Windows 10 machine

3.   Scan for RDP exposure (Nmap)

<img width="788" height="656" alt="image" src="https://github.com/user-attachments/assets/974cb8e5-4b0f-4fa6-8116-123098a06974" />

- Run a port scan targeting RDP (default 3389) and validate service visibility per lab configuration.   

(The results shown after the port scan in the screenshot above suggest that port 3389 rdp is filtered means that a firewall, filter, or other network obstacle is blocking the port so that Nmap cannot tell whether it is open or closed. )

<img width="731" height="467" alt="image" src="https://github.com/user-attachments/assets/f55b5d9c-2ccb-44a1-872f-53f358ebf1c3" />

- The Remote Desktop protocol configured to allow traffic has shown in the screenshot above

<img width="708" height="586" alt="image" src="https://github.com/user-attachments/assets/64ddb63d-1f69-4846-999a-eed302058b79" />

- (Since the Remote Desktop service has been enabled on the Windows Machine, it is possible to verify the service running on the device by performing a Nmap Port Scan. By default, the port that the Remote Desktop service runs on is port 3389. It can be observed that the Windows machine with IP Address 192.168.0.1 has Remote Desktop Service successfully. It is also able to extract the System Name of the Machine, it is DESKTOP-S5EA3ND.)

4.   Brute-force / Dictionary attack (Hydra)

Prepare dictionaries:

- user.txt (candidate usernames)

- pass.txt (candidate passwords)


<img width="940" height="795" alt="image" src="https://github.com/user-attachments/assets/477a1cb8-e744-47be-ba9e-021ec33839cd" />


(In a process of performing a penetration test on the Remote Desktop service, after the Nmap scan, it is time to do a Bruteforce Attack. Hydra. Although called a Bruteforce, it is more like a dictionary attack. We need to make two dictionaries one with a list of probable usernames and another with a list of probable passwords.)


<img width="940" height="795" alt="image" src="https://github.com/user-attachments/assets/93731981-ca95-478a-b5b8-3de83e4e6a13" />


- The attack on the Windows machine is shown in the screenshot ABOVE

---

# (F) 🛡️ Defense Validation (IPBan)

Expected behavior:

- IPBan monitors log events and detects repeated login failures

- Once the failure threshold is exceeded, IPBan bans the attacker IP

<img width="940" height="607" alt="image" src="https://github.com/user-attachments/assets/3faa8a1d-d024-41f9-be49-176e2fa38105" />


Evidence appears in:

- IPBan logs showing failed attempts and ban action

---

# 📌 Results / Outcome

✅ Successful simulation of brute-force attempts against RDP
✅ IPBan detected suspicious behavior and blocked the attacker IP after multiple failed logins
✅ Defensive tuning improved responsiveness by reducing ban threshold

---
# 🔐 Security Notes & Improvements

- Prefer strong passwords + account lockout policies

- Consider MFA for RDP access

- Limit RDP exposure to VPN / jump box only

- Use allowlisting + geo/IP reputation controls where possible

- Centralize logs (SIEM) for correlation across endpoints





