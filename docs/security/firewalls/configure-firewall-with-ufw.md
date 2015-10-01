---
author:
  name: Elle Krout
  email: ekrout@linode.com
description: 'Use UFW (Ucomplicated Firewall) to manage your firewall on Ubuntu, Debian, or Arch Linux; this guide contains instructions for setting up default rules, adding/removing rules, setting up logging, and some advanced features.'
keywords: 'ufw,uncomplicated firewall,ubuntu ufw,linux ufw,ufw tutorial,ubuntu firewall,iptables,networking,firewalls,filtering,firewall setup,ubuntu,debian,arch'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
modified: Monday, September 25th, 2015
modified_by:
  name: Elle Krout
published: 'Monday, September 25th, 2015'
title: How to Configure a Firewall with UFW
---

UFW, or *uncomplicated firewall*, is a frontend for managing firewall rules on your Ubuntu, Debian, or Arch servers. UFW is used through the command line (although has GUIs available), and aims to make firewall configuration easy (or uncomplicated).

## Setting Up UFW

### Debian/Ubuntu

1.  Update your system:

        sudo apt-get update && sudo apt-get upgrade
        
2.  Install UFW:

        sudo apt-get install ufw
        
3.  Start and enable UFW:

    -   On Debian 8/Ubuntu 15.04:
    
            sudo systemctl start ufw
            sudo systemctl enable ufw
            
    -   On Debian 7/Ubuntu 14.04:
    
            sudo service ufw start
            sudo service ufw enable
            

### Arch Linux

1.  Update your system:

        sudo pacman -Syu
        
2.  Install UFW:

        sudo pacman -S ufw

3.  Start and enable UFW:

        sudo systemctl start ufw
        sudo systemctl enable ufw


## Using UFW

### Set Default Rules

In general, most systems will need a certain amount of ports open for connections, and a vast majority of ports closed. To start with an easy basis of rules, the `ufw default` command can be used to set the default response to incoming and outgoing connections. To deny all incoming and allow all outgoing connections, run:

    sudo ufw default allow outgoing
    sudo ufw default deny incoming
        
The `ufw default` command also allows for the use of the `reject` parameter.

{:.caution}
>Configuring a default reject or deny rule can lock you out of your Linode unless explicit allow rules are in place.  Ensure that you have configured allow rules for SSH and other critical services as per the section below before applying default deny or reject rules.

### Adding Rules

Rules can be added in two ways: By denoting the **port number** or by using the **service name**.

For example, to allow both incoming and outgoing connections on port 22 for SSH, you can run:

    sudo ufw allow ssh
    
You can also run:

    sudo ufw allow 22
    
Similarly, to **deny** traffic on a certain port (in this example, 111) you would only have to run:

    sudo ufw deny 111
    
To farther fine-tune your rules, you can also allow packets based on TCP or UDP. The following will allow TCP packets on port 80:

    sudo ufw allow 80/tcp
    sudo ufw allow http/tcp
    
Whereas this will allow UDP packets on 1725:

    sudo ufw allow 1725/udp
    
    
### Advenced Rules

Along with allowing or denying based solely on port, UFW also allows you to allow/block by IP addresses, subnets, and a IP address/subnet/port combinations.

To allow connections from an IP address:

    sudo ufw allow from 123.45.67.89
    
To allow connections from a specific subnet:

    sudo ufw allow from 123.45.67.89/24
    
To allow a specific IP address/port combination:

    sudo ufw allow from 123.45.67.89 to any port 22 proto tcp
    
`proto tcp` can be removed or switched to `proto udp` depending upon your needs, and all instances of `allow` can be changed to `deny` as needed.

    
### Removing Rules

To remove a rule, add `delete` before the rule implementation. If you no longer wished to allow HTTP traffic, you could run:

    sudo ufw delete allow 22
    
Deleting also allows the use of service names.


## Editing UFW's Configuration Files

Although simple rules can be added through the command line, there may be a time when more advanced or specific rules need to be added or removed. Prior to running the rules input through the terminal, UFW will run a file, `before.rules`, that allows loopback, ping, and DHCP. To add to alter these rules edit the `/etc/ufw/before.rules` file. A `before6.rules` file is also located in the same directory for IPv6.

An `after.rule` and an `after6.rule` file also exists to add any rules that would need to be added after UFW runs your command-line-added rules.

An additional configuration file is located at `/etc/default/ufw`. From here IPv6 can be disabled or enabled, default rules can be set, and UFW can be set to manage built-in firewall chains.


## UFW Status

You can check the status of UFW at any time with the command: `sudo ufw status`. This will show a list of all rules, and whether or not UFW is active:

    Status: active
    
    To                         Action      From
    --                         ------      ----
    22                         ALLOW       Anywhere
    80/tcp                     ALLOW       Anywhere
    443                        ALLOW       Anywhere
    22 (v6)                    ALLOW       Anywhere (v6)
    80/tcp (v6)                ALLOW       Anywhere (v6)
    443 (v6)                   ALLOW       Anywhere (v6)


### Enable the Firewall

With your chosen rules in place, your initial run of `ufw status` will probably output `Status: inactive`. To enable UFW and start your firewall, run:

    sudo ufw enable
    
Similarly, to disable the firewall, run:

    sudo ufw disable
    
## Logging

You can enable logging with the command:

    sudo ufw logging on
    
Log levels can be set by running `sudo ufw logging low|medium|high`, selecting either `low`, `medium`, or `high` from the list. The default setting is `low`.
    
A normal log entry will resemble the following, and will be located at `/var/logs/ufw`:

    Sep 16 15:08:14 <hostname> kernel: [UFW BLOCK] IN=eth0 OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:00:00 SRC=123.45.67.89 DST=987.65.43.21 LEN=40 TOS=0x00 PREC=0x00 TTL=249 ID=8475 PROTO=TCP SPT=48247 DPT=22 WINDOW=1024 RES=0x00 SYN URGP=0
    
The initial values list the date, time, and hostname of your Linode. Additional important values include:

-   **[UFW BLOCK]:** This location is where the description of the logged event will be located. In this instance, it blocked a connection.

-   **IN:** If this contains a value, then the event was incoming

-   **OUT:** If this contain a value, then the event was outgoing

-   **MAC:** A combination of the destination and source MAC addresses

-   **SRC:** The IP of the packet source

-   **DST:** The IP of the packet destination

-   **LEN:** Packet length

-   **TTL:** The packet TTL, or *time to live*. How long it will bounce between routers until it expires, if no destination is found.

-   **PROTO:** The packet's protocal

-   **SPT:** The source port of the package

-   **DPT:** The destination port of the package

-   **WINDOW:** The size of the packet the sender can receive

-   **SYN URGP:** Indicated if a three-way handshake is required. `0` means it is not.