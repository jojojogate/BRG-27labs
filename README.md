# BRG-27labs

# Simple FTP Server Setup on Ubuntu (vsftpd)

This setup demonstrates a basic FTP server hosted on a cloud-based Ubuntu instance (e.g., AWS EC2) using the **vsftpd** service.

---

## ✅ What Works

- The FTP service is live and reachable on port **21**
- Custom welcome banner is displayed on login
- A test user account (`ftpuser`) has been created and can successfully authenticate
- Basic FTP commands like `pwd` work once logged in

---

## ⚠️ What Doesn't Work (Yet): `ls` and Passive Mode

After login, attempting to run the `ls` command (which lists files on the server) hangs or fails.

```bash
ftp> ls
229 Entering Extended Passive Mode (|||10750|)
^C
ftp> 

