# Uncomplicated FireWall(UFW)

Prerequisites - Computer Networks

To work with firewalls, the user needs root/admin privileges and cannot be configured without it.

The pictures are examples and not meant to be used in practical sense

## Getting Started

### Installing UFW

To install UFW - 

`sudo apt install ufw`

To check if it is installed correctly or To check if you already have it - 

`sudo ufw version`

### Check status - 

`sudo ufw status`

OR

`sudo ufw status verbose` for detailed version.

### Reload and Reset -

UFWs are reloaded after change in configuration. Command for reload - 

`sudo ufw reload`

To reset UFW to default configuration - 

`sudo ufw reset`

### Help commands -

To get help for commands, enter this - 

`sudo ufw -h`

### Enable or Disable UFW

To enable UFW - `sudo ufw enable`

TO disable ufw - `sudo ufw disable`

## Configuration

Configuring rules is pretty easy, its similar to setting ACL on routers and should make sense in English. 

### Basic Rules - 

1) `allow` - permit communication 

2) `deny` -  silently drop the packet

3) `reject` - refuse connection and return "destination unreachable" or "TCP RST" response

4) `limit` - limits the number of connection attempts to a certain rate, which if exceeded will `deny` the packets. The default is 6 attempts per ip every 30 seconds. i have mentioned how to change the rate at the end of thhis readme file. Usually used with `SSH`, `RDP`, login APIs and attempt sensitive situations

`deny` silently drops the packet and does not return any response. The sender/client recieves no response and hence 'goes into' a timeout where it waits for response and then gives up. `deny` is more suitable when you want stealth and hide services.

`reject` refuses connection and returns a response with a "destination unreachable" message or "TCP RST" status. As such the sender knows that the connection was rejected. `reject` is not stealthy but is 'polite' to inform the user and should be used when we want to inform the client that the connection was rejected.

### Directions

There are two directions - incoming and outgoing - 

1) `from` - incoming direction, i.e. the clients from where we recieve traffic

2) `out` - outgoing direction, i.e. the server where we are setting up the firewall

## Example Uses - 

### To allow/deny/reject ports- 

`sudo ufw <rule> <port>` 

`sudo ufw <rule> <port>/<protocol>`

Example - `sudo ufw allow 22` OR
            `sudo ufw reject 80/tcp` OR
              `sudo ufw allow out 22/tcp`

### To set rules for IP /subnets

`sudo ufw <rule> <direction> <ip>` OR `sudo ufw <rule> <direction> <ip>/<subnet>`

Example - `sudo ufw allow from 192.168.0.0/24`

### To specify destination port - 

`sudo ufw <rule> <direction> <ip> to <server ip(can be 'any')> port <port number> proto <protocol>`

Example - `sudo ufw allow from 10.0.0.0/8 to any port 443 proto tcp`

All commands  can be found using the help flag.

There can be various combinations depending on what you want. Firewalls need to be specific and must consider all scenarios and not miss any blindspots for a highly secure system. 

### Delete a rule

Use this to list Rules in a numbered format - 

`sudo ufw status numbered`

Once the rules are listed with a numbered format. Use this to delete a specific numbered rule - 

`sudo ufw delete <rule number>`

### Change the rate of `limit`

Here is an example of how to change rate - 

`dport 22` is the ssh port

`--seconds 60` is change the limit to 60 seconds

`--hitcount 3` is the number of attempts allowed which is 3

Here we are limiting the connection to 3 SSH connections every 60 seconds per IP

`sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set`

`sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 3 -j DROP`

## Miscellaneous

### Logging

`sudo ufw logging on` - logs all denied/rejected packets

Logs also have differnet verbosity - 

1) `sudo ufw logging low` - default and recommended
2) `sudo ufw logging medium` 
3) `sudo ufw logging high`
4) `sudo ufw logging full` - include allowed packets 

Logs can be found in system log. We can have a dedicated file for logging. Follow these steps - 

1) create a config file - `sudo nano /etc/rsyslog.d/20-ufw.conf`

2) Add this configyration to the file - 

        :msg, contains, "UFW" /var/log/ufw.log
        & stop

3) Restart rsyslog - `sudo systemctl restart rsyslog`

### Automatically enable firewall on Start

To automatically enable the firewall on device start, enter this command 

`sudo systemctl enable --now ufw`

Check its status - 

`systemctl status ufw`

### Lazy?

If you are too lazy to type rules, Install a graphical uncomplicated firewall (GUFW). 