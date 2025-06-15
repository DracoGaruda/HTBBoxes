# ![logo](https://github.com/user-attachments/assets/739ac72d-d6a0-4e10-abbb-31c46fccc07b) ReachKart - DFIR Analysis (HackTheBox)

##   Scenario Overview

ReachKart is an e-commerce website running on Ehtereum blockchain technology, to support rapid development without disrupting the live environment, the company implemented a "production mirror" strategy, creating an exact replica of the production environment for development and testing purposes. The mirror includes replicated databases, services and blockchain nodes, but it appears that customer data has not been anonymized. Customer support is now receiving emails from sellers complaining that their wallets are missing ether. You have to investigate and understand if and how an intrusion has occurred.

---

##  Investigation Tasks

### Section 1: Path Traversal Attack

- Lets filter out traffic to HTTP Protocols to observe the URIs targeted
- As you work throught the filtered traffic you will find below packet where the attacker is performing Path Traversal Attack.
- As seen in the responses to similar packets above, those resulted in 500 errors, whereas this one returned a 200 OK response.

  ![Q1-2](https://github.com/user-attachments/assets/01e046a2-0245-49d1-881c-6a75becefbbb)

- Just to Confirm Follow TCP Stream for the packet you will find /etc/passwd file
- So vulnerable endpoint is /user/getOrderBill and UTC time(look at payload request) is 2025-03-01 04:09:22 (Original time was in EST)
- We can confrim that the Website is vulnerable to Path Traversal Attacks

####  Task 1  
**Question:** What was the vulnerable endpoint that allowed the attacker to leak files?  
**Answer:** `/user/getOrderBill`

####  Task 2  
**Question:** When was the first successful exploitation of the vulnerable endpoint by the attacker (time in UTC)?  
**Answer:** `2025-03-01 04:09:22`


- As you review further the attacker took advantage of path traversal attack and accessed package.json,rk-server.js,rk-logging.js.

   ![Q3-4_1](https://github.com/user-attachments/assets/bc28fecd-9b34-4fd3-a4f6-b897d50030a7)

- Now follow TCP Stream for Package.json and you can see all the softwares and versions used by the website

  ![Q3-4_2](https://github.com/user-attachments/assets/e89e9f6d-d87f-4249-88c4-3a916fd865b2)

- Express verison number is 4.21.2
- Among the list Ethers,Web3 and hardhat are the only etherium related softwares where hardhat is used as Ethereum development environment and framework so Development smart network is hardhat:2.22.18
- Now lets review rk-server.js and rk-logging.js. Do the similar and follow the TCP Stream and you will find follwoing in rk-logging.js

   ![Q3-4_3](https://github.com/user-attachments/assets/8f98d09a-ce56-4818-bd04-bacea6e61b88)

- Secret key used for jwt tokens is found "SuperSecretPassword"

####  Task 3  
**Question:** Which version of Express is currently being used on the server?  
**Answer:** `4.21.2`

####  Task 4  
**Question:** Which Ethereum compatible development smart contract network is running on the server? (Format: name@version)  
**Answer:** `hardhat@2.22.18`

####  Task 5  
**Question:** What is the signing key used by the server to sign JSON Web Tokens (JWT)?  
**Answer:** `SuperSecretPassword`

---

### Section 2: JWT Token Info

- Observer packets to admin page
- Follow the TCP stream for the packer you will find the JWT token

  ![Q6-7_2](https://github.com/user-attachments/assets/1ec4eddb-13d9-4328-a843-d533ac0098d5)

- Decode JWT token using JWT decoders (I used https://www.jstoolset.com/jwt)

  ![Q6-7](https://github.com/user-attachments/assets/76e9a63b-10b7-4bfb-9476-b532c4be624e)

- As you can see the email id used for this token was darthvader@empire.com
  
####  Task 6  
**Question:** The attacker was able to generate a JWT from the signing key and log in to the admin panel. What is the JWT value?  
**Answer:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiRGFydGggVmFkZXIiLCJlbWFpbCI6ImRhcnRodmFkZXJAZW1waXJlLmNvbSIsImFjY291bnRfdHlwZSI6ImFkbWluIiwiaWF0IjoxNzQwODAyNzM0LCJleHAiOjE3NDA4ODkxMzR9.RBmMEC7IYmkzz1LsT-kP_JkLCUb7-hsJKX4IdjE91TE`

####  Task 7  
**Question:** Decode the token and find the email used by the attacker to log in to the admin panel  
**Answer:** `darthvader@empire.com`

---

### Section 3: File Download

- Filter traffic for Websocket and HTTP protocol. Look For TCP information for Port number which is 8888.

  ![Q8](https://github.com/user-attachments/assets/f1ccd525-41ad-45e2-ac6b-377db951dab7)

- Follow Websocket stream and also look for exportable objects in wireshark you will find Get request for reachkart.db file
- Reachkard.db being downloaded from 172.18.10.2:6969 with Get request at 2025-03-01 04:22:01 UTC

  ![q9](https://github.com/user-attachments/assets/c5cea690-1694-4a24-b704-da0456f04ce5)

- Download Reachkart.db from exportable objects in Wireshark. Then use SHA256 generator to generate hash of the file.
- Reachjart.db is in SQLite format. Open the file with Dbbrowser and under table you will find follwoing 8 seller wallets.

  ![q11](https://github.com/user-attachments/assets/47876c18-56e2-449e-b091-f35c8037ed41)


####  Task 8  
**Question:** The admin panel uses WebSocket to send and receive terminal input. What port is being used?  
**Answer:** `8888`

####  Task 9  
**Question:** The attacker then was able to retrieve a sensitive file. When did the attacker get the file (UTC)?  
**Answer:** `2025-03-01 04:22:01`

####  Task 10  
**Question:** What is the SHA-256 hash of the file that the attacker downloaded ?  
**Answer:** `fabe3234bb709ee5e5c5c2789c891a9a49368ffa520b23d60f6be2f2ca81bac6`

####  Task 11  
**Question:** How many sellers are there in the e-commerce website?  
**Answer:** `8`

---

### Section 4: Etherium Stolen

#### Task 12  
**Question:** The attacker started sending Ether from all identified sellers' wallets. What is the hash of the first transaction?  
**Answer:** `0x7b7ded2d51f0dcb1bf3fc5cc9598b81a7a622aac15d3841d377c548986e0a7c3`

#### Task 13  
**Question:** What was the total amount of Ether stolen by the attacker? (1 Eth = 10^18 wei)  
**Answer:** `24.4`

#### Task 14  
**Question:** What is the block number of the last transaction in which Ether was stolen? (Decimal)  
**Answer:** `18`

#### Task 15  
**Question:** After the attacker stole the Ether, what was the balance in their wallet? (Ignore the trailing zeros)  
**Answer:** `24.40023`

---

###  Summary

The attacker leveraged a **remote code execution vulnerability** (CVE-2024-28353) in the **TEW-827DRU router**, exploiting an HTTP parameter via POST requests. Through command injection (`usbapps.config.smb_admin_name`), they executed commands such as `whoami` and used `wget` to pull a reverse shell script, which ultimately connected to a **C2 server** at `35.159.25.253:41143`.

---
