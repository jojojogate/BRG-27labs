# BRG-27labs

# Simple FTP Server Setup on Ubuntu (vsftpd)
This setup demonstrates a basic FTP server hosted on a cloud-based Ubuntu instance (e.g., AWS EC2) using the **vsftpd** service.

1. Launch an EC2 Instance (Ubuntu) <br>
AMI: Ubuntu Server 22.04 LTS <br>
Instance Type: t2.micro (Free Tier) <br>
Security Group Settings: <br>
Allow SSH (22) – Source 0.0.0.0/0 <br>
Allow FTP (21) – Source 0.0.0.0/0 <br>
Elastic IP (recommended): Allocate and associate to your instance for a fixed public IP <br>

2. Install vsftpd on the EC2 Instance <br>
SSH into your EC2 server: ssh -i your-key.pem ubuntu@<public-ip> <br>
Update packages and install vsftpd: sudo apt update <br>
Install vsftpd: sudo apt install vsftpd -y <br>

3. Configure vsftpd <br>
Edit the config file: sudo nano /etc/vsftpd.conf <br>
Make sure these settings are configured: <br>
listen=YES <br>
anonymous_enable=NO <br>
local_enable=YES <br>
write_enable=YES <br>
chroot_local_user=YES <br>

 #Optional custom welcome message <br>
ftpd_banner=Welcome to My Briding FTP service. <br>

4. Create a Local FTP User <br>
sudo adduser ftpuser <br>
Set a password <br>

Give the user a home directory: <br>
sudo mkdir /home/ftpuser <br>
sudo chown ftpuser:ftpuser /home/ftpuser <br>
Fix permission issue (vsftpd will refuse login if chroot dir is writable): <br>
sudo chmod a-w /home/ftpuser <br>

5. Open Required Ports in UFW or AWS <br>
If using UFW on the instance: <br>
sudo ufw allow 21/tcp <br>
sudo ufw reload <br>
If relying on AWS Security Groups, update your inbound rules to include: <br>

Type	Protocol	Port Range	Source <br>
SSH	TCP	22	0.0.0.0/0 <br>
FTP	TCP	21	0.0.0.0/0 <br>

6. Test From Your Kali Linux Machine <br>
Open a terminal in Kali:ftp <your-ec2-public-ip> <br>
Example session: <br>
```
┌──(kali㉿kali)-[~]
└─$ ftp 13.213.78.181
Connected to 13.213.78.181.
220 Welcome to My Briding FTP service.
Name (13.213.78.181:kali): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
#If you see this you have accessed your FTP server

Restart the vsftpd service after changes

---

## What Works

- The FTP service is live and reachable on port **21**
- Custom welcome banner is displayed on login
- A test user account (`ftpuser`) has been created and can successfully authenticate
- Basic FTP commands like `pwd` work once logged in

---

## What Doesn't Work (Yet): `ls` and Passive Mode

After login, attempting to run the `ls` command (which lists files on the server) hangs or fails.

```
ftp> ls
229 Entering Extended Passive Mode (|||10750|)
^C
ftp>
```

Why This Happens
FTP operates using two types of connections:
Control connection (port 21) – used to send commands like USER, PASS, etc.
Data connection – used for file transfers and commands like ls, get, put.
In Passive Mode, the server tells the client to connect to a random high-numbered port (like 10750) to receive data. If that port is not open, the connection will hang or timeout when trying to list files or transfer data.

How to Fix Passive Mode
For you guys out there that actually want to use the FTP service you will need to add afew other configurations

1. Configure vsftpd for Passive Mode
Edit /etc/vsftpd.conf: nano /etc/vsftpd.conf

pasv_enable=YES
pasv_min_port=10090
pasv_max_port=10100
This restricts the data ports to a known range you can open on your firewall.

Then restart vsftpd: sudo systemctl restart vsftpd

2. Open Passive Ports in the Firewall
For UFW (Ubuntu): sudo ufw allow 10090:10100/tcp
For AWS EC2: Go to Security Groups for your instance

Edit Inbound Rules
Type: Custom TCP
Port Range: 10090–10100
Source: Anywhere (0.0.0.0/0) or your IP for tighter security

3. Test Again
Reconnect from your FTP client and try to list: ls
This should now succeed without hanging.

Additional Notes
vsftpd will refuse login if the FTP root directory is writable when chrooted. This is a security feature.

You'll see:
500 OOPS: vsftpd: refusing to run with writable root inside chroot()
To fix:
chmod a-w /home/ftpuser
FTP sends credentials in plain text. Avoid using it over the public internet unless tunneling through SSH or using FTPS (not covered here).

Consider using SFTP for a more secure alternative (built into SSH).
