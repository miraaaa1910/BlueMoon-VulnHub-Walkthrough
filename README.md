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

There’re 2 outputs from here which are /server-status and /hidden_text. But server status will just lead me to 404 error. Then I search http://192.168.56.103/hidden.txt

![image] ( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133002.png)  

At first glance it looks like a maintenance page with no information but when I look closely the ‘Thank You’ word is actually a clickable link. 

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133013.png )

QR code appear when I click the ‘Thank You’. Then I try to decode the QR using online tools.

![image]( https://github.com/miraaaa1910/BlueMoon-VulnHub-Walkthrough/blob/4449128214bf6a81de04c2d836b4aa9a08218426/image/Screenshot%202026-03-15%20133100.png )
