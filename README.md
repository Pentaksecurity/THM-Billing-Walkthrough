# THM-Billing-Walkthrough

<img width="1672" alt="Screen Shot 2025-03-13 at 6 14 09 PM" src="https://github.com/user-attachments/assets/2e383778-8651-4630-a822-7752f0bff170" />

# Recon using Nmap

First lets start with some Recon using Nmap.

nmap -A -T4 -p 10.10.66.150 -v 

![Nmap](https://github.com/user-attachments/assets/2151283b-8611-4fcf-a0fd-012293e82adf)

As you can see we have a few ports open.

Port 3306/tcp SQL

Port 80/tcp http

Port 22/tcp ssh

Port 5038/tcp Asterisk Call Manager

Also since we used the -A which a pretty aggressive scan and -v for verbosity we got some more info on Service and Version of each port.


![nmap 2](https://github.com/user-attachments/assets/ffb8c447-4fcc-443b-b5a7-06b2b3d1d52e)

And I can see Asterisk Manager 2.10.6 and MariaDB. 

After a bit of digging I found there is a known vulnerability for Magnus Billing Asterisk Manager.

# Metasploit

I decided to check Metasploit.

After launching Metasploit I searched for "asterisk"


![Aterisk Metsploit](https://github.com/user-attachments/assets/75eca361-d0e1-4300-aec9-dd4b56d0a07e)

And long behold I have a Magnus Billing RCE CVE-2023-30258.

Lets use it!

![MSF Meterpreter](https://github.com/user-attachments/assets/71d3d942-e4e9-469d-a224-d3403931a9dc)


So after setting my RHOSTS to the Target IP and LHOST to my Kali Machine.

I got a Meterpreter Session open!

Now lets transition to a shell!

P.S. can't see it my screenshot but when you get the Meterpreter session open type in "shell"

And you don't have to but if you want a more stable shell you can run:

python3 -c 'import pty;pty.spawn("/bin/bash");'

After that you can navigate to the home directory > magnus > find the user.txt > and you got the first flag!!!!!!


![userflag1](https://github.com/user-attachments/assets/3d0f08d7-f415-41ed-a9f7-a0255efb2deb)

# Privelage Escalation

So let's run 'sudo -l' to see what kind of sudo permissions we have.

![sudol](https://github.com/user-attachments/assets/ce133a05-4edb-4521-a23d-bc15483d1315)

As you can see we have /usr/bin/fail2ban-client

Honestly I had to do a bunch of digging and I found a few websites that go through Fail2ban priveleage escalation.

What I found was that fail2ban monitors system logs for malicious activity, such as repeated failed login attempts, and automatically blocks the offending IP addresses by updating firewall rules.

So I found some blogs that led me to checking the status.

Jails are basically configs that difine what to monitor for and pattern to look for.

![status](https://github.com/user-attachments/assets/63139fec-6917-4faa-ad69-70d988d332a2)

To execute commands as root, we can modify an action defined for the jail like what action to perform when blocking an IP.

sudo /usr/bin/fail2ban-client get asterisk-iptables actions

![priv esc](https://github.com/user-attachments/assets/ca53f28d-21ae-477d-b85a-83d4c5f31dd5)

Modify the command for the actionban which is executed when banning an IP and set it to run our command that sets the setuid bit on /bin/bash 



<img width="1085" alt="Screen Shot 2025-03-13 at 7 01 11 PM" src="https://github.com/user-attachments/assets/3835bfa1-637f-45ac-9bcb-3bc9adc92c62" />


Now we can ban an IP for the asterisk-iptables jail!

With setuid bit set on the /bin/bash binary, we can use it to obtain a shell 

python3 -c 'import os;import pty;os.setuid(0);os.setgid(0);pty.spawn("/bin/bash");'

<img width="793" alt="Screen Shot 2025-03-13 at 7 03 25 PM" src="https://github.com/user-attachments/assets/ab65a307-e00b-4193-8544-f15e05eb41a1" />

Now that we got the root shell lets navigate to the root.txt file and get the final flag!

![privesc fklag2](https://github.com/user-attachments/assets/edd96186-6b52-4a11-be09-75fc2a74e681)

