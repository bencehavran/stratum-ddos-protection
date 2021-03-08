Welcome to my Repo

Fail2ban jail+filter to protect stratum from DDoS attack.

Edit your /etc/fail2ban/jail.conf add somewhere under # JAILS, and do not forget to edit /path/to/stratum/file.log!

[stratum]
enabled  = true
#ports to ban (numeric or text)
port     = all
# ban action
action   = iptables-allports[name=fail2ban]
# filter name
filter   = stratum
# log file, you need to edit this, point it to the file where your stratum log. 
logpath  = /path/to/stratum/file.log
# time to try
maxretry = 1
# in 10 seconds
findtime = 10
# ban time in sec
bantime = 600000000

make a file under /etc/fail2ban/filter.d/stratum.conf and insert this:

[Definition]
failregex = .*\[<HOST>] .*no data*.

now restart fail2ban
sudo systemctl restart fail2ban.service
than you will see something like this:

● fail2ban.service - Fail2Ban Service
   Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-03-05 11:00:08 UTC; 4h 11min ago
     Docs: man:fail2ban(1)
  Process: 837 ExecStartPre=/bin/mkdir -p /var/run/fail2ban (code=exited, status=0/SUCCESS)
 Main PID: 845 (fail2ban-server)
    Tasks: 3 (limit: 2308)
   CGroup: /system.slice/fail2ban.service
           └─845 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Mar 05 11:00:08 DAEMON systemd[1]: Starting Fail2Ban Service...
Mar 05 11:00:08 DAEMON systemd[1]: Started Fail2Ban Service.
Mar 05 11:00:09 DAEMON fail2ban-server[845]: Server ready

than check jail
sudo fail2ban-client status stratum

Status for the jail: stratum
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     199
|  `- File list:        /home/egyel/szart/client.log
`- Actions
   |- Currently banned: 199
   |- Total banned:     199
   `- Banned IP list:   

ready!

Another recommended jail:

insert this in /etc/fail2ban/jail.conf after [stratum]

[iptables-dropped]

enabled = true
filter = iptables-dropped
banaction = iptables-allports
port = all
logpath = /var/log/kern.log
bantime = 1800
maxretry = 3

than make a iptables-dropped.conf under /etc/fail2ban/filter.d/iptables-dropped.conf

[Definition]
failregex = IPTables Dropped: .* SRC=<HOST>
ignoreregex = 

save, restart fail2ban

copy this lines one by one to shell under root or use sudo:

iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "IPTables Dropped: " --log-level 4
iptables -A LOGGING -j DROP

ready.

