---
title: MuddyWater Writeup - SwampCTF 2025
draft: false
modified: 2025-03-30T23:58:47+11:00
date: 2025-03-30T18:54:02+11:00
tags:
  - writeup
  - ctf
  - SwampCTF
author: dsgrace
---
# Challenge
Challenge description:
> We caught a threat actor, called MuddyWater, bruteforcing a login for our Domain Controller. We have a packet capture of the intrustion. Can you figure out which account they logged in to and what the password is?
> 
> Flag format is `swampCTF{<username>:<password>}`

[Writeup also available on my blog](https://blog.grace.sh/posts/swampctf2025_muddywater/)
# Filtering
We are given a .pcap file, which we can open in WireShark or other similar software.  
There are 97k packets in this file, so we need to filter to find data that's useful to us.  
When we open the file and scroll, we can see lots of SMB2 packets with "Error: STATUS_LOGON_FAILURE" and "Error: STATUS_MORE_PROCESSING_REQUIRED":  
![A packet capture within WireShark](/img/swampctf2025-muddywater01.png)  
How can we filter these out?  
If we click on one of them we can look at the SMB2 details:  
![Packet details](/img/swampctf2025-muddywater02.png)  
We can see that the one related to this error is NT Status, to copy the value in brackets right-click this field and choose Copy -> Value.  
After this we can filter it out with the display filter `smb2.nt_status != 0xc000006d`, this still leaves all the `STATUS_MORE_PROCESSING_REQURIED` errors.  
To filter both out we can use the display filter `smb2.nt_status != 0xc000006d and smb2.nt_status != 0xc0000016`, which leaves us with ~9000 packets displayed out of the 97k total packets:  
![Filtered packet list](/img/swampctf2025-muddywater03.png)  
There's still lots of `Negotiate Protocol Response` packets in the list, let's filter those too with `and smb2.cmd != 0` which leaves us with just 3 packets:  
![Display filter showing 3 packets in Wireshark](/img/swampctf2025-muddywater04.png)  
Now we can right-click packet number 72074 and choose Follow -> TCP Stream to filter to just this conversation, we can close the window that comes up.  
This updates the display filter to `tcp.stream eq 6670` where we can see an Encrypted SMB3 conversation, and the username `DESKTOP-0TNOE4V\hackbackzip`:  
![Encrypted SMB3 conversation in Wireshark](/img/swampctf2025-muddywater05.png)  
# Password - getting the hash
Now we have the username, how can we get the password?  
If we search for something like "decrypt SMB Wireshark" we can find [this article](https://malwarelab.eu/posts/tryhackme-smb-decryption/) - since we just have the traffic we'll want to use "Method 3: Decrypting SMB with the captured traffic only", which says we can use hashcat.  
To use hashcat our hash needs to be in the format `username::domain:ntlmserverchallenge:ntproofstr:rest_of_ntresponse` and we need to use NetNTLMv2 hash mode (`-m 5600`), we'll also use a straight (wordlist) attack (`-a 0`).  
First lets get the details and setup our hash.  
We know from earlier the username and domain are `hackbackzip::DESKTOP-0TNOE4V:`.  
## ntlmserverchallenge
For the `ntlmserverchallenge` double-click the "Session Setup Response" at packet number 72065:  
![Packet details with the NTLM server challenge](/img/swampctf2025-muddywater06.png)  
If we drill down into `SMB2 -> Session Setup Response -> Security Blob -> GSS-API Generic [...] -> Simple Protection Negotation -> negTokenTarg -> NTLM Secure Service Provider`, we can find the NTLM server challenge - this can be copied by right-click -> Copy -> Value on the field.  
So far our hash is `hackbackzip::DESKTOP-0TNOE4V:d102444d56e078f4:ntproofstr:rest_of_ntresponse`.  
## ntproofstr and ntresponse
Now we need to see the "Session Setup Request" in packet 72069.  
![Packet details showing the NT proof str and the NT response](/img/swampctf2025-muddywater07.png)  
We can copy the NTProofStr for `ntproofstr` and NTLMv2 Response for `rest_of_ntresponse` - which we just remove ntproofstr from:  
````
NTProofStr  : eb1b0afc1eef819c1dccd514c9623201
Full NT Reponse: eb1b0afc1eef819c1dccd514c962320101010000000000006f233d3d9f9edb01755959535466696d0000000002001e004400450053004b0054004f0050002d00300054004e004f0045003400560001001e004400450053004b0054004f0050002d00300054004e004f0045003400560004001e004400450053004b0054004f0050002d00300054004e004f0045003400560003001e004400450053004b0054004f0050002d00300054004e004f00450034005600070008006f233d3d9f9edb010900280063006900660073002f004400450053004b0054004f0050002d00300054004e004f004500340056000000000000000000
restofntresponse: 01010000000000006f233d3d9f9edb01755959535466696d0000000002001e004400450053004b0054004f0050002d00300054004e004f0045003400560001001e004400450053004b0054004f0050002d00300054004e004f0045003400560004001e004400450053004b0054004f0050002d00300054004e004f0045003400560003001e004400450053004b0054004f0050002d00300054004e004f00450034005600070008006f233d3d9f9edb010900280063006900660073002f004400450053004b0054004f0050002d00300054004e004f004500340056000000000000000000
````
So our full hash will be:  
````
hackbackzip::DESKTOP-0TNOE4V:d102444d56e078f4:eb1b0afc1eef819c1dccd514c9623201:01010000000000006f233d3d9f9edb01755959535466696d0000000002001e004400450053004b0054004f0050002d00300054004e004f0045003400560001001e004400450053004b0054004f0050002d00300054004e004f0045003400560004001e004400450053004b0054004f0050002d00300054004e004f0045003400560003001e004400450053004b0054004f0050002d00300054004e004f00450034005600070008006f233d3d9f9edb010900280063006900660073002f004400450053004b0054004f0050002d00300054004e004f004500340056000000000000000000
````
# Password - hashcat
Now we can try and crack the password with hashcat, save your full hash in a file somewhere:  
```powershell
PS .\hashcat.exe -a 0 -m 5600 .\ntlm.txt .\rockyou.txt --show
HACKBACKZIP::DESKTOP-0TNOE4V:d102444d56e078f4:eb1b0afc1eef819c1dccd514c9623201:01010000000000006f233d3d9f9edb01755959535466696d0000000002001e004400450053004b0054004f0050002d00300054004e004f0045003400560001001e004400450053004b0054004f0050002d00300054004e004f0045003400560004001e004400450053004b0054004f0050002d00300054004e004f0045003400560003001e004400450053004b0054004f0050002d00300054004e004f00450034005600070008006f233d3d9f9edb010900280063006900660073002f004400450053004b0054004f0050002d00300054004e004f004500340056000000000000000000:this-is-not-the-actual-password
```
There would normally be more output, but it just shows me the password as I previously cracked it!  
If you want to get the actual password you can download hashcat and get the rockyou.txt wordlist from [SecLists on GitHub](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz)!  
# Flag 🚩
Now that we have the username and password we can put them together to make the flag.  
`swampCTF{hackbackzip:this-is-not-the-actual-password}`!