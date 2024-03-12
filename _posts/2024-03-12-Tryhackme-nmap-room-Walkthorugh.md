<h1>Tryhackme - nmap room</h1>

<h3>#Task1 - Deploy</h3>
- no anwser

<h3>#Task2 - Introducion</h3>

#What networking constructs are used to direct traffic ro the application on a server?
- ports

#How many of these are available on any network-enabled computer?
- 65535

#How many of these are considered "Well-known"?
- 1024

<h3>#Task3 - Nmap Switches</h3>

#What is the first switch listed in the help menu for a *"Sync Scan"*?
- -sS

#Which switch would you use for a *"UDP scan"*?
- sU

#If you wanted to detect which *operating* system the target is running on, which switch would you use?
- -O

#Nmap provides a switch to detect the *version* of the services running on the target. What is this switch?
- -sV

#The default output provided by nmap often does not provide enough information for a pentester. How would you *increase the verbosity*?
- -v

#Verbosity level one is good, but verbosity level two is better! How would you set the *verbosity level to two*?
- -vv

#What switch would you use to save the nmap results in three major formats?
- -oA

#What switch would you use to save the nmap results in a "normal" format?
- -oN

#A very useful output format: how would you save results in a "grepable" format?
- -oG

#Sometimes the results we're getting just aren't enough. If we don't care about how loud we are, we can enable "aggressive" mode. This is a shorthand switch that activates service detection, operating system detection, a traceroute and common script scanning.

How would you activate this setting?
- -A

#Nmap offers five levels of "timing" template. These are essentially used to increase the speed your scan runs at. Be careful though: higher speeds are noisier, and can incur errors!

How would you set the timing template to level 5?
- -T5

#How would you tell nmap to only scan port 80?
- -p 80

#How would you tell nmap to scan ports 1000-1500?
- -p 1000-1500

#How would you tell nmap to scan all ports?
- -p-

#How would you activate a script from the nmap scripting library?
- --script

#How would you activate all of the scripts in the "vuln" category?
- --script=vuln

<h3>#Task5 - TCP Connect Scans</h3>

#Which RFC defines the appropriate behaviour for the TCP protocol?
- RFC 9293

#If a port is closed, which flag should the server send back to indicate this?
- RST