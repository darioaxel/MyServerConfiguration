# MyServerConfiguration
Documentation about how i configured my own server

## Setting up DDClient with Google Host 

### [Installation](https://samhobbs.co.uk/2015/01/dynamic-dns-ddclient-raspberry-pi-and-ubuntu)

Use this command to install ddclient:
```
sudo apt-get install ddclient
```
### Enable dynamic DNS with Google Host

Under the DNS section of Google Domains, go to the “Synthetic Records” section.

Select “Dynamic DNS” from the left menu, enter the subdomain of your domain you want to configure for DyDNS, and press “Add.” In this example, I entered “@” because I want the top-level of my domain (rainvilletech.net itself) to be configured for DyDNS. If you wanted “random.mysite.com” to be dynamically updated, you’d enter “random” instead of “@.”

### DDClient Configuration

The main configuration file for ddclient is at /etc/ddclient.conf, you can open this file to edit it with a text editor of your choice - this command will open it in vi:

```
sudo vi /etc/ddclient.conf
```
Here is a sample "basic" configuration file for ddclient:

```
# /etc/ddclient.conf
if=enxb827eb06c7a8

ssl=yes
protocol=googledomains
login=username
password=’passwordhere’
daemon=300
use=web, web=checkip.dyndns.com
mywebsite.com
```
**if**: Enter the network interface that should be used for making connections. This is easily obtainable via “ifconfig.”

**protocol=googledomains**: DDclient supports this, although not listed as an option in the setup we went through at install, which is a bit misleading.

**login**: Enter your username, found from the Dynamic DNS synthetic record you made at the beginning.

**password**: Make sure this has single-quotes around it.

**daemon**: This is the interval of seconds between checks the server will perform to see if its IP needs updating.

**use, web**: This tells DDclient to obtain its own external IP via dydns.org. Remember, if you’re running NAT (which you probably are), the IP in your server’s interface is not the same as your public IP! This takes care of that.

**mywebsite.com**: Enter your domain here. If you’re updating the top-level of the domain like I am, omit the “@.”
The web method involves ddclient querying one of the many "what is my ip" type web services on the internet, and extracting your IP address from the page returned. You can tell ddclient to use this method by using this line:

Sending your password via http (not https) is a bad idea. This parameter will force https:
```
ssl=yes
```

This needs to go above the protocol parameter in your config file.

For this to work, you need a perl library that can use SSL. Install it with this command:
```
sudo apt-get install libio-socket-ssl-perl
```
### [Testing your configuration](http://www.netinstructions.com/how-to-setup-dynamic-dns-for-home-computer-or-server/)

You can check if the pre-defined use values can detect your WAN IP by running this command:
```
sudo ddclient -query
```

If your server is connected with an ethernet cable, the output should look something like this:
```
use=if, if=lo address is 127.0.0.1
use=if, if=p2p1 address is 192.168.1.119
use=if, if=wlan0 address is NOT FOUND
use=web, web=dnspark address is 1.2.3.4
use=web, web=dyndns address is 1.2.3.4
use=web, web=loopia address is 1.2.3.4
```
To test your ddclient configuration with really verbose output, printing all possible configuration parameters and their values, you can use this command:
```
sudo ddclient -debug -verbose -noquiet
```
I won't print a sample output because it's too long, but somewhere near the bottom you should see a line like this:

SUCCESS:  updating backup: good: IP address set to 1.2.3.4

While we've got all this information, It's worth checking to make sure you are actually using SSL to connect to your dynamic DNS provider. Look for lines like this:

CONNECT:  dynamicdns.park-your-domain.com
CONNECTED:  using SSL

### Run ddclient as a daemon

Since we don't just want the IP address to update once, we still need to set up ddclient to run as a daemon so it can check for a change of IP address periodically and notify the dynamic DNS provider if necessary.

To start the daemon we need to open another configuration file, `/etc/default/ddclient` and set:
```
run_daemon="true"
```
You will notice there is a daemon_interval parameter there too, I think the default value of 300 seconds (5 minutes) is reasonable, so I didn't change it.

Save and close the file, and then run:
```
sudo service ddclient start
```
to start the daemon, and:
```
sudo service ddclient status
```
to check its status.

ddclient keeps a cache of your IP address, and it will only update the record with your dynamic DNS provider if your IP address hasn't changed. Since some ISPs seem to only allocate new IP addresses when the modem is power cycled, and some dynamic DNS providers will time out if you don't update the record in a while, there is one thing left to do - we need to add a cron job to force an update weekly, just in case.

Choose whether you want to force an update daily or weekly, and then create a file called ddclient in the relevant directory, e.g. `etc/cron.daily` or `/etc/cron.weekly`:
```
sudo nano /etc/cron.daily/ddclient
```
Fill in this information:
```
#!/bin/sh
/usr/sbin/ddclient -force
```
Then make the script executable:
```
sudo chmod +x /etc/cron.daily/ddclient
```
## G-NAT
Don't forget to ask your service provider to take you off the [G-NAT](https://en.wikipedia.org/wiki/Carrier-grade_NAT)
## Install OpenSSH Server

Within the terminal, run the following command as root to install the OpenSSH server package.
```
apt install openssh-server -y
```

To configure the server use 
```
sudo xed /etc/ssh/sshd_config
```
