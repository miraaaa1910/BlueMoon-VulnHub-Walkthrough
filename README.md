# BlueMoon-VulnHub-Walkthrough
Lab 2 Vulnerability Analysis

## 1. Start BlueMoon and Kali Linux
![image](https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/3136a04016e27a4e19a77acece2354ed298ddfc2/image/Screenshot%202026-03-15%20211701.png)
![image](https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/92c58b6947e48ebccecdaf3f5b45ed93a9ad0253/image/Screenshot%202026-03-15%20173735.png)

## 2. Reconnaissance
First, I use ifconfig to know my IP address

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20131842.png)

Now that I know my IP address is 192.168.56.102, I need to figure out the target’s IP address by using nmap -sN 192.168.56.0/24. This command basically scans every device on the 192.168.56.0 network to see which ports are open by sending empty TCP packets.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20131933.png)

In result, I noticed that there’s 4 IP addresses. I already verify that 192.168.56.102 is my IP. While 192.168.56.1 is my host machine and 192.168.56.100 is the DHCP server. So, 192.168.56.103 is very likely our target’s IP address.

## 2. Scanning
After getting the target’s IP address, the next step is to scan vulnerabilities by using nmap -sC -sV -Pn -vv 192.168.56.103. This command identifies the exact versions of programs running on the target and uses automated scripts to find potential weaknesses.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20132016.png) 

As we can see, the target system exposes potential vulnerabilities through an open FTP service, an accessible SSH service and a web application running on Apache HTTP which may allow unauthorized access or exploitation if misconfigured or outdated. Since it has a website, I try to search for it in my browser using http://192.168.56.103. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20132941.png)

But I didn’t find any clue or hint. So, I need to discover secret or unlisted parts of a website that aren’t visible by just browsing normally. I used gobuster dir -u hhtp://192.168.56.103 -w /usr/share/wordlists/dirbuster/directory-listl2.3-medium.txt. command. This command is systematically testing thousands of possible directory name against the target web server to discover hidden or unlinked content.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20132827.png)

There’re 2 outputs from here which are /server-status and /hidden_text. But server status will just lead me to 404 error. 
## 3. Gain Access
Now that I know vulnerabilities that can be exploit, I have to gain access. To do that I need to find a user credential. Back to the gobuster scan /hidden.txt is very likely to have information that I need. Then I search http://192.168.56.103/hidden.txt

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133002.png )  

At first glance it looks like a maintenance page with no information but when I look closely the ‘Thank You’ word is actually a clickable link. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133013.png )

QR code appear when I click the ‘Thank You’. Then I try to decode the QR using online tools.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133100.png )

This script is a shortcut to connect to an FTP server with a preset username and password, log in and then disconnect. Then, I try log in using the credentials given. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133143.png )

I’ve gained valid access to the FTP server and now I can explore its files, download them or even upload a new one depending on permission. Next, I use ls -al command. This command gives me detailed list of everything in the directory including hidden file.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133256.png )

The server responded with two files which are information.txt and p_list.txt. I use get command to download both of the to my computer and then logged out. Then I confirmed that the files I downloaded from the FTP server are now saved locally in my machine by using ls command and ready to open and inspect. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133341.png )

To show their contents I use cat command for both of the files.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133351.png )
![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133407.png )

The first file is a clue that someone named Robin has a weak password. The second file is a dictionary of possible passwords I can try against Robin’s account. To find which one is the password for Robin, I use hydra -l robin -P p_lists.txt ssh://192.168.56.103. Hydra is one of brute-force attack. Hydra will try many passwords from p_lists.txt against SSH login for user robin on 192.168.56.103. Hydra tested each password until found the correct one which is k4rv3ndh4nh4ck3r.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133430.png )

Next, I use ssh robin@192.168.56.103 command to log into the target machine using credential that I got earlier.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133546.png )

To confirm I logged in as the user robin I use whoami command. ls for listed the contents of robin’s home directory. I found 2 items which are project and user1.txt. cat user1.txt to displayed the contents of the file. And I catch the first flag. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133617.png )

sudo -l command is used to check what commands my current user is allowe to run with sudo.  It revealed that robin can run the script feedback.sh as the user jerry without password. That’s a potential way to escalate privileges.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20133900.png )

sudo -u jerry /home/robin/project/feedback.sh lets me run a command as jerry instead of robin. It’s a way to become another user from that command. The id output shows that jerry is part of the docker group. Membership in the docker group is a big privilege escalation opportunity because users in that group can often run Docker containers with root privileges.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20134819.png )

I listed contents of /home/jerry by using ls la command and found a file called user2.txt. When I displayed it with cat, I captured the second flag.
## 4. Escalate Privilege
Since we know that jerry is part of docker group, I use docker run -v /:/mnt –rm -it alpine chroot /mnt sh. Being in the docker group gave jerry the ability to run Docker containers. Docker can mount and access the host filesystem. By combining chroot with the mounted /, I passed normal restrictions and became root on the host.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20135113.png )

## 5. Maintain Access
Because I’m a root now, I can manage accounts freely. I created a new user account called sysadmin using useradd -m -s /bin/bash sysadmin command to maintain persistence. Then set the password with passwd sysadmin.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20135716.png )

To gave sysadmin administrative privilege, I add it to the sudo group by using usermod -aG sudo sysadmin. This mean I can log in as sysadmin and run the commands with sudo to become root again without needing to repeat the whole exploitation chain.

![image](https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20135746.png )

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20135810.png )

## 6. Clear Tracks
To clear track in the log I use history -c and recheck it with history command.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20140300.png )
![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20140308.png )

## 7. Final Result
Now I can login to the system using sysadmin.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/e0c344aac91a86e50e1d680d41ba90db2799e40a/image/Screenshot%202026-03-15%20140439.png )
