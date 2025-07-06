# BRG-27labs

# Simple FTP Server Setup on Ubuntu (vsftpd)

This setup demonstrates a basic FTP server hosted on a cloud-based Ubuntu instance (e.g., AWS EC2) using the **vsftpd** service.

---

## What Works

- The FTP service is live and reachable on port **21**
- Custom welcome banner is displayed on login
- A test user account (`ftpuser`) has been created and can successfully authenticate
- Basic FTP commands like `pwd` work once logged in

---

## What Doesn't Work (Yet): `ls` and Passive Mode

After login, attempting to run the `ls` command (which lists files on the server) hangs or fails.

```bash
ftp> ls
229 Entering Extended Passive Mode (|||10750|)
^C
ftp> 

Why This Happens
FTP operates using two types of connections:

Control connection (port 21) – used to send commands like USER, PASS, etc.

Data connection – used for file transfers and commands like ls, get, put.

In Passive Mode, the server tells the client to connect to a random high-numbered port (like 10750) to receive data. If that port is not open, the connection will hang or timeout when trying to list files or transfer data.
