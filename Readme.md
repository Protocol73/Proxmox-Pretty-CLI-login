## Proxmox Pretty Login Notes.md

### Intro:

Lets start with some file locations and basics,  

```
/etc/motd
/etc/issue
/usr/bin/pvebanner
```

*MOTD* -- Proxmox Message of the day  
	Far as I am aware you can just change this with your favorite text editor.  
	This gets printed to your screen/terminal after login.  
	Have fun, make it cool, or make it Corporate generic:   
 - ================================  
 - === "Unauthorized access is prohibited..." ==
 - ================================  
 
The choice is yours, I'll provide [a few examples] you can look at for ideas later.(GitHub Link) ===============


Next is the login banner via the issue file.  
Proxmox generates this file via a [Perl script](https://forum.proxmox.com/threads/etc-issue-edits.54964/)  
And the pvebanner service/command. (No docs for it, really.)
If you write directly to the issue file,  
the next time the service runs it will get overwritten. (at reboot)

To print your current issue file to the terminal:  
``` sudo getty --show-issue ```  

As the issue file is still displayed to the system using [getty](https://manpages.debian.org/bullseye/util-linux/agetty.8.en.html),  
you can also use escape codes to display system info.  

Here is the ones I am using:  
	Note: I had never written any Perl before this project. :-|

```
my $ipbridge0 = '\4{vmbr0}'; #Print IPv4 address of {Interface}.
my $ipbridge1 = '\4{vmbr1}'; #Added for servers with mutiple {interfaces}.
my $clearterm = '\ec'; #clear the terminal
my $fqdn = '\n.\O'; #What it says on the tin
my $architecture = '\m'; # Same as 'uname -m'
my $Kernel = '\r'; #PVE Kernel Version
```

Also there is a:  
``` 
	my $xline = '-' x 78;
```
That Prints the top & bottom bars, We'll come back to that.

### Ready to Make Changes? 

Now before we change the banner file lets make a copy of the original  

**NOTE: I am running as a user with sudo;**

Make a copy of the original as: pvebanner.bak0  
``` sudo cp /usr/bin/pvebanner /usr/bin/pvebanner.bak0 ```

I name copies of original as .bak0 {Back Up 0}  
That way if I want to restore or save another version of the file I can just Increment.  
And restoring to the original with this process is as simple as:  
``` cp \file\original\path.bak0 \file\original\path ```  

Makes me a little more willing to break things :-)

Now, I want colors in my output so,
I'll also be using [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)  
More info if you want to get crazy with it.

+ [Flogsoft - Colors & Formatting ](https://misc.flogisoft.com/bash/tip_colors_and_formatting)
+ [Stack Overflow](https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux#5947802)

Alright, lets open the script and start making it pretty!
```
sudo nano /usr/bin/pvebanner
```


There is a more then one way to add escape codes to texts file  
In this case, the easiest is using '\e'

```
\e[39m Default
\e[31m Red
\e[32m Green
\e[33m Yellow
\e[34m Blue
\e[35m Magenta
\e[36m Cyan
\e[37m Light gray

\e[0m  Clear/Reset
```

For Testing how it will look, you can add this above:   
```
$xline

__EOBANNER

}
```

Now Save your changes (Ctrl + S) and exit nano (Ctrl + X)

Run the script to update the /etc/issue file: ```sudo pvebanner ```

Check your new output: ```sudo getty --show-issue ```
Or you can just: ``` cat /etc/issue ```

I recommend that you check how it looks via anyway the server will be accessed. (via CLI)

In my case that is: 
+ KVM @ the Server Rack
+ SSH via network (Win & OSX)
+ Web console (only for /etc/motd)

I liked the look of an ASCII Banner, so I used [Patorjk.com](http://patorjk.com/software/taag/#p=display&f=ANSI%20Shadow&t=Proxmox)  
"ANSI shadow" works for the look I want, but there is plenty of options.  


Toss it all togeather and you should end up with something like this:
```
#!/usr/bin/perl

use strict;
use PVE::INotify;
use PVE::Cluster;

my $nodename = PVE::INotify::nodename();
my $localip = PVE::Cluster::remote_node_ip($nodename, 1);

my $xline = '\e[32m=\e[0m' x 110;
my $banner = '';

#Custom PVE Banner Data by Protocol73
my $ipbridge1 = '\4{vmbr1}'; #Print IPv4 address of {Interface}.
my $ipbridge2 = '\4{vmbr2}'; #Added for servers with mutiple {interfaces}
my $clearterm = '\ec'; #clear the terminal
my $fqdn = '\n.\O'; #What it says on the tin
my $architecture = '\m'; # Same as 'uname -m'
my $Kernel = '\r'; #PVE Kernel Version

#ASCI Banner in Green
my $BLbanner ='\e[32m
██████╗ ███████╗██╗     ██╗     ███████╗██╗   ██╗██╗   ██╗███████╗    ██╗      █████╗ ██████╗ ███████╗
██╔══██╗██╔════╝██║     ██║     ██╔════╝██║   ██║██║   ██║██╔════╝    ██║     ██╔══██╗██╔══██╗██╔════╝
██████╔╝█████╗  ██║     ██║     █████╗  ██║   ██║██║   ██║█████╗      ██║     ███████║██████╔╝███████╗
██╔══██╗██╔══╝  ██║     ██║     ██╔══╝  ╚██╗ ██╔╝██║   ██║██╔══╝      ██║     ██╔══██║██╔══██╗╚════██║
██████╔╝███████╗███████╗███████╗███████╗ ╚████╔╝ ╚██████╔╝███████╗    ███████╗██║  ██║██████╔╝███████║
╚═════╝ ╚══════╝╚══════╝╚══════╝╚══════╝  ╚═══╝   ╚═════╝ ╚══════╝    ╚══════╝╚═╝  ╚═╝╚═════╝ ╚══════╝
\e[0m';

#Custom if no IP set on Bridge1
if ($ipbridge1){
   $ipbridge1 = 'TRUNK';
}else{
        #IP is set, do nothing;
}
#END custom Banner Data

if ($localip) {
    $banner .= <<__EOBANNER;
$clearterm
$xline

Welcome to the Proxmox Virtual Environment @
$BLbanner
$xline
Please use your web browser to configure this server - connect to:

FQDN: \e[35m $fqdn \e[0m
PVE Cluster Management IP: \e[33m https://$localip:8006 \e[0m

All Interfaces:
        Bridge0:\e[34m $ipbridge1 \e[0m
                10Gig Port#01 [Trunk to Switch Stack, Failover to 1Gig Ethernet [Trunk to Stack]
        Bridge1:\e[34m $ipbridge2 \e[0m
                10Gig Port#02 [VLAN2 on X4012] with failover to 1Gig Ethernet [VLAN2 @ Stack]

$xline

__EOBANNER

}

open(ISSUE, ">/etc/issue");

print ISSUE $banner;

close(ISSUE);

exit (0);
```

![/etc/issue Banner 01](1-R720-etc-issue-Custom-Banner.png)


