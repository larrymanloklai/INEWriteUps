# Title: Network Penetration Testing CTF 1 (eCPPTv3)

### 1) Overview:
This lab focuses on using SNMP to extract sensitive data and establish an initial foothold on a target system. You will then escalate privileges and pivot to compromise a vulnerable application on a machine that is not directly accessible from Kali. Through uncovering sensitive data, gaining shell access, exploiting vulnerabilities, and elevating privileges, you will progress through a multi-stage attack scenario.

### 2) Task 1: Leverage SNMP to uncover a user with access to a sensitive share on server.prod.local 
(1) find out DNS mapping by cat /etc/hosts.
(2) find out which IP we have connection with.
(3) nmap TCP SYN scan for target Windows system. then go for service, standard script and OS scan to understand what we are dealing with.
- <img width="1917" height="929" alt="Screenshot from 2026-01-04 20-36-11" src="https://github.com/user-attachments/assets/53a410cc-3b31-4e9b-8a4d-c3f050782917" />
(4) we know the SNMP from the task so another namp scan was done. The standard -sU scan came back with filtered | open state. Since it was UDP scan port 161, without a community string there was no response.
(5) nmap snmp brute force also didn't yield any outcome, meaning the community string was not in nmap's default list.
(6) metasploit framework's list was used and the community string was found.
- <img width="1917" height="662" alt="image" src="https://github.com/user-attachments/assets/a40a73f9-8b41-4bb7-a3d7-61e4d4ec3fb5" />
(7) snmpwalk was also used `snmpwalk -v 1 -c blue server.prod.local > snmpoutput.txt` for further enumeration.
(8) Then msfconsole was used to further enumerate users and shares.
- <img width="1917" height="930" alt="image" src="https://github.com/user-attachments/assets/7f048263-3cb8-45a0-8550-2ce5494144e6" />
(9) A long report was given. the most important parts were:
- <img width="1917" height="420" alt="Screenshot from 2026-01-04 20-53-14" src="https://github.com/user-attachments/assets/6501b521-8c99-49f0-8706-d52a6ec1e9d2" />
- <img width="1917" height="323" alt="Screenshot from 2026-01-04 20-54-32" src="https://github.com/user-attachments/assets/2602480c-15ca-4291-8e57-6c30f722370e" />
(10) Obviously in the context of CTF, timothy is the account we need to gain access to. nevertheless I will brute force all the account names found - just in case. After trying both password lists given, timothy's password was found.
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/54d15168-26dd-484a-b2ae-2685c84585f4" />
(11) I continued to let the smb login brute force running when I got the flag1 and additional information for flag2.

### 2) Task 2: Obtain a shell on server.prod.local &  Task 3: Gain elevated privileges on server.prod.local
(1) With the credentials obtained in task 1. The first thing came to mind was psexec to get a shell. However there was not writable share to enable that.
(2) Then the next obvious channel is to exploit mssql with the credential found in task 1.
(3) mssqlclient was found and used to login. username was timmy, and it was found that timmy could be impersonated as sa. It was done and xp_cmdshell was enabled in order to execute a reverse shell back to the msfconsole in the attack box.
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/c87df517-580b-4f28-9deb-ab267f076103" />
(4) Session was elevated and process migrated and found all flag 1 2 3.
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/abd772ee-0f96-4f6d-b5fe-f2fa726320f5" />

### 3) Task 4: Exploit a vulnerable application on web.prod.local
(1) with autoroute running a TCP port scan was done. Since the lab was indicating the tool finger might be required, I included port 79 in the scan. Ports 22, 79 and 80 were opened.
- <img width="1917" height="923" alt="Screenshot from 2026-01-04 21-40-30" src="https://github.com/user-attachments/assets/e526788e-d11b-454c-aaa1-d1ddb65912f3" />
(2) Then I was using nmap for -sV and it was probably too aggressive. At one point I thought the target was down.
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/0b723156-d3bd-420d-bcda-1c395a34872e" />
(3) Then let's setup RDP to machine 1 and enumerate machine 2. We are dealing with SPIP v.4.3.1. However, the exploitation code is not available in searchsploit or msfconsole. For SPIP v.4.3.1 exploitation code is available online.
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/2ca34d9b-3bb7-41f0-9631-e4bfc2080fb8" />
- <img width="1917" height="923" alt="image" src="https://github.com/user-attachments/assets/4c8c0e1e-8d67-4b89-b0bc-3c4df78fd47e" />
(4) Then my lab was expired - great! I got time to research how to get the code. First I got the exploit.py code however it was so difficult to copy and paste to the Guacamole environment. After it was done, the script was lacking the library required. Then finally I found this scirpt not depending on other library.
(5) https://github.com/saadhassan77/SPIP-BigUp-Unauthenticated-RCE-Exploit-CVE-2024-8517
(6) Here was how flag4.txt was found.
- <img width="1917" height="796" alt="image" src="https://github.com/user-attachments/assets/f49e6308-071b-445b-a596-ec58aaba222e" />

### 4) Task 5: Examine a user's .plan file on web.prod.local for a privilege escalation opportunity
(1) to examine user's .plan first we need to find the usernames from /etc/passwd, then we can enumerate by `finger` one by one. Soemthing interesting was found from auditor's.
- <img width="1917" height="920" alt="image" src="https://github.com/user-attachments/assets/aa35bddc-3535-4b54-b703-8b669ac3935f" />
(2) further enumerate as per the hint. Here was how the ssh_key was found.
- <img width="1917" height="920" alt="image" src="https://github.com/user-attachments/assets/b376b70a-8b0e-4d1a-a506-630cda3f7a17" />
(3) The priv key was saved, followed by chmod 400, then login. Here was how the list flag, flag5 was found.
- <img width="1917" height="920" alt="image" src="https://github.com/user-attachments/assets/a99e19da-201b-42ca-9589-b63bc63a85a1" />








