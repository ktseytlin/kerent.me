---
layout: post
title:  "Remote Debugging Nifi on an EC2 With IntelliJ"
date:   2016-09-22 11:30:00
categories: blog
---

Purpose: I have a Nifi instance running on an AWS EC2 instance. I want to connect a remote debugger from my local machine and walk through what is happening on the EC2.

<u>**PREP WORK**</u>:
If you want easy Nifi installation, just copy and paste the below lines into a new machine. I'm using a Linux, and then using Linux brew to brew install Nifi (just to make my life a bit easier).

```
sudo yum groupinstall 'Development Tools' -y && sudo yum install curl git m4 ruby texinfo bzip2-devel curl-devel expat-devel ncurses-devel zlib-devel -y
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"
PATH="$HOME/.linuxbrew/bin:$PATH"
echo 'export PATH="$HOME/.linuxbrew/bin:$PATH"' >> ~/.bash_profile
echo 'export MANPATH="$HOME/.linuxbrew/share/man:$MANPATH"' >> ~/.bash_profile
echo 'export INFOPATH="$HOME/.linuxbrew/share/info:$INFOPATH"' >> ~/.bash_profile
brew install nifi
JAVA_HOME=~/.linuxbrew/bin/java
```

<u>**ONE**</u>. Change bootstrap.conf file

Okay, so in order to change the bootstrap.conf file, we need to SSH into our EC2 machine. If you're not sure how to do that yet, just go to the AWS console, right click on your machine and copy the line that lets you SSH.

Let's stop Nifi before we change the configs.

```
jps -m
kill xxxx  # whatever process Nifi is running at
```

Now let's change the file:

```
cd .linuxbrew/Cellar/nifi/1.0.0/libexec/conf
vi bootstrap.conf
```

In this conf file, you should see two lines that look like this:

```
Enable Remote Debugging
java.arg.debug=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000
```

Uncomment the second line. For me, changing it to 8000 caused stuff to stop working, even if Nifi said it was up and running. To make it work I had to change the address to port 5500.

Now, let's start Nifi back up:

```
cd ~/.linuxbrew/Cellar/nifi/1.0.0/bin
./nifi start
```

Double check that Nifi is running at the IP address. The URL should look like this: http://xx.xxx.xxx.xx:8080/nifi/

<u>**TWO**</u>. Set up IntelliJ for Remote Debugging

1. Run --> Edit Configurations --> Plus (+) sign to add an additional debugger (click on remote within here)
2. This will open up a new debugger called "Unnamed".
3. Change the name to "AWS Nifi" or whatever makes you happy. Change the "Host" to whatever your EC2 IP address is and the port to whatever you set it to in bootstrap.conf. Then press OK.
4. Do Run --> Debug --> Then click on the name that you gave the Remote Debug process above.

Doing this led me to the following error message: Error running AWS Nifi: Unable to open debugger port
(xx.xxx.xxx.xxx:5500): java.net.ConnectionException "Operation timed out"

If you get a similar problem, then let's continue to the next section.

<u>**THREE**.</u> Port Forwarding

Because I work in an enterprise, it is not easy to just change the security groups to open up communication between the EC2 and my local machine. And that makes sense - I shouldn't be opening up permanent connection between my local machine and my Nifi. Due to a bunch of corporate hoops to jump through, trying to do this permanently will also take way too long. So let's do port forwarding.

So we changed the debug port to 5500 (or 8000 or any other number). Now we are going to port forward, and we are going to need to use our pem file to do this. You will do this within your local terminal, not on the terminal of the EC2.

```
ssh -i <<LOCATION OF PEM FILE>> -N -L 5500:<<IP ADDRESS OF AWS EC2 NIFI>>:5500 ec2-user@<<IP ADDRESS OF AWS EC2 NIFI>>
```

Example of Location of Pem File would be something like: ~/Documents/pemfilexample.pem

Now that we have it forwarding to localhost we need to change the configs in IntelliJ.

1. Run --> Edit Configurations --> Go to "AWS Nifi" or whatever you called it.
2. Change the "Host" to 127.0.0.1
3. Do Run --> Debug --> Then click on the name that you gave the Remote Debug process above.

Now if you go to Debugger you should see that it is Connected! VIOLA!!!

<u>Useful Links</u>:

* https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding
* http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-ssh-tunnel-local.html
