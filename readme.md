Markdown# Linux II Lab 9 - Hardening and Maintenance

This is my lab report for system hardening and maintenance.

## 1. Before and After (Lab 9.A)

I captured the initial state of the system into `before.txt`:
```bash
{ ss -tlnp; systemctl list-units --type=service --state=running; dpkg -l | wc -l; find / -perm -4000 -type f 2>/dev/null; } > ~/before.txt
After that, I removed unnecessary packages and services:Bashsudo systemctl disable --now cups avahi-daemon
sudo apt purge telnetd vsftpd samba
sudo apt autoremove --purge
Then I created a new snapshot after.txt and compared them:Bash{ ss -tlnp; systemctl list-units --type=service --state=running; dpkg -l | wc -l; find / -perm -4000 -type f 2>/dev/null; } > ~/after.txt
diff -u ~/before.txt ~/after.txt
Justification TableRemoved ItemWhat risk did it expose?What would break without it?CUPSUnnecessary print server port and potential vulnerabilitiesNetwork printer usageAvahiLocal network device service discoveryFinding devices on the local networkTelnetdUnencrypted login (passwords leak)Legacy telnet connectionsVsftpdInsecure FTP file transferSending files via FTPSambaWindows file sharing on the networkSMB file sharingDesktop Env.Large graphical attack surface and resource usageGraphical user interface (GUI)2. SSH and Updates (Lab 9.B)I checked SSH settings with sudo sshd -t and reloaded the service using sudo systemctl reload sshd.Password authentication was disabled (only key-based login is allowed). An external test (ssh -o PreferredAuthentications=password ...) now returns Permission denied.ProxyJump ConfigurationIn my ~/.ssh/config file, the bastion/router jump is configured like this:Code snippetHost router
    HostName 172.18.16.29
    User vboxuser
    IdentityFile ~/.ssh/id_ed25519

Host bloated
    HostName 172.18.16.14
    User vboxuser
    ProxyJump router
    IdentityFile ~/.ssh/id_ed25519
Automatic UpdatesI installed unattended-upgrades, ran a test (sudo unattended-upgrade --dry-run), and checked the logs to verify that updates work properly (sudo journalctl -u unattended-upgrades).
