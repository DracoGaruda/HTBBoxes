# ![Screenshot 2025-06-15 223943](https://github.com/user-attachments/assets/55577135-f9bf-4b6d-8937-87885d76a6ef)Unsupervised - DFIR Analysis (HackTheBox)

##   Scenario Overview

The incident happened around 4:30 PM on Friday, "The Last day of the week" at the Accounts/Marketing department of "Finance XYZ" company. There weren't many people in the department, and the remaining were not paying much attention to their surroundings as they were getting ready to head home. After the weekend on Monday while reviewing the security cam footage member of an IT team saw "Eddie" a new intern in the Accounts/Marketing department, plugging a USB into an unauthorized computer (containing sensitive financial and marketing documents), interacting with computer and unplugging the USB before heading out. As the incident happened 2 days ago, and not having enough knowledge of what Eddie did the security team use caution while asking around and gathering intel to avoid causing suspicion. The only information they were able to find out was that Eddie had a "Toshiba" USB. You are provided with a partial image of the “unauthorized computer" as well as a list of important documents, to investigate what he did and if he stole something sensitive or not?).

---

##  Investigation Tasks

- We have a .ad1 file provided for analysis. Open this file with FTK Imager.
- With intial analysis you will find we have access to Software, System registry hives and recent files.
- ![1](https://github.com/user-attachments/assets/2cf576bd-8a3c-403e-9069-cdfd69606a88)


### Registry and USB Analysis


- Open Software and System hives with Eric Zimmerman Registry Explorer.
- Look for ControlSet001\Control\TimeZoneInformation registry Key under System Hive

####  Task 1  
**Question:** Find out the time zone of victim PC. (UTC+xx:xx)?  
**Answer:** `UTC+05:00`


- As you can see from FTK imager the only User logged in is MrManj
- You could also verify it by going to Profilelsit registry key under System Hive

####  Task 2  
**Question:** Employees should be trained not to leave their accounts unlocked. What is the username of the logged in user?  
**Answer:** `MrManj`


- THere are three true storage devices Toshiba, Kingston and unknown product.
- Note attach and detach time stamps for Toshiba USB
  
  ![2](https://github.com/user-attachments/assets/feb475d5-9bce-4d21-8d2c-43bb9a5b0040)
  
####  Task 3  
**Question:** How many USB storage devices were attached to this host in total?  
**Answer:** `3`

####  Task 4  
**Question:** What is the attach timestamp for the USB in UTC?  
**Answer:** `2024-02-23 11:37:50`

####  Task 5  
**Question:** What is the detach timestamp for the USB in UTC?  
**Answer:** `2024-02-23 11:39:12`

---

### Shell folders

- We found bunch of LNK files and Jump lists under recent folder.
- Run Zimmerman tools LEcmd and JLECmd against LNK files nad Jumplist respectively.
- Personally ran it against entire directory and saved it in a csv files
- Note: When you review and relate Mounted Devices Registry Key with USB Registry key. E: is Toshiba USB Volume name

  ![Lnk file](https://github.com/user-attachments/assets/fb0d9bc9-6a5c-4175-aa0e-6703a3ecdab1)

####  Task 6  
**Question:** Which folder did he copy to the USB?  
**Answer:** `Documents`

####  Task 7  
**Question:** There were subfolders in the folder that was copied. What is the name of the first subfolder? (Alphabetically) 
**Answer:** `Business Proposals`

####  Task 8  
**Question:** Eddie opens some files after copying them to the USB. What is the name of the file with the .xlsx extension Eddie opens?  
**Answer:** `Business Leads.xlsx`

####  Task 9  
**Question:** Eddie opens some files after copying them to the USB. What is the name of the file with the .docx extension Eddie opens?  
**Answer:** `Proposal Brnrdr ltd.docx`


  
---

## Additional USB Analysis

- Lets go back to registry key and check VolumeinfoCache key under software hive.
  
  ![Volumeinfo](https://github.com/user-attachments/assets/11a3edee-4831-4661-85e9-c4a4a78a6364)

####  Task 10  
**Question:** What was the volume name of the USB?  
**Answer:** `RVT-9J`

####  Task 11  
**Question:** What was the drive letter of the USB?  
**Answer:** `E`

## Thumbache

- Based on the hint extract thumbcache files from Local\AppData\Microsoft\Windows\Explorer
- Now use thumcacheviewer to review thumbcache files. Upon review thumbcache_256.db is of intrest
- You will find following image file that gives eddie last name

  ![thumbcache](https://github.com/user-attachments/assets/1f0698ed-71ec-44e0-a1e5-3a0a0c929e12)

#### Task 12  
**Question:** I hope we can find some more evidence to tie this all together. What is Eddie's last name?  
**Answer:** `Homer`

- Google Vid and pid of the device
  
  ![usb](https://github.com/user-attachments/assets/af8eaf46-233a-47cb-9b44-c287e89a8ab5)

#### Task 13  
**Question:**  There was an unbranded USB in the USB list, can you identify it's manufacturer’s name?  
**Answer:** `Shenzhen SanDiYiXin Electronic Co.,LTD`


---

###  Summary

An internal data exfiltration incident occurred at Finance XYZ on a quiet Friday around 4:30 PM. Security footage later revealed Eddie, a new intern, inserting a Toshiba USB drive into an unauthorized computer and interacting with it briefly. Forensic investigation using tools like FTK Imager, Registry Explorer, and Eric Zimmerman’s suite confirmed the system timezone as UTC+05:00, with the logged-in user identified as MrManj. Three USB storage devices were recognized, with the Toshiba device connected from 2024-02-23 11:37:50 to 11:39:12 UTC. Analysis of LNK files revealed Eddie copied the “Documents” folder to the USB, including the subfolder “Business Proposals”, and accessed files like “Business Leads.xlsx” and “Proposal Brnrdr ltd.docx”. The USB volume name was RVT-9J, mounted as drive E. Thumbcache analysis uncovered an image revealing Eddie’s last name as Homer. The USB was identified via VID/PID as a generic device from Shenzhen SanDiYiXin Electronic Co., LTD, strengthening the evidence of unauthorized data access.

