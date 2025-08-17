# Guide: Server‑Wide Email Sending on Ubuntu with msmtp (for Cron Jobs, Alerts, and Scripts)

This guide configures your server to send emails through a third‑party SMTP provider (Gmail, Zoho, Outlook, custom SMTP) using msmtp. It applies system‑wide, so cron jobs, scripts, and system alerts can use the `mail` command.

## Quick summary

- Install:  `sudo apt-get install msmtp msmtp-mta mailutils`
- Configure: `/etc/msmtprc` (use app passwords; `logfile ~/.msmtp.log`)

---

## 1) Install msmtp, msmtp-mta, and mailutils
```
sudo apt-get update
sudo apt-get install msmtp msmtp-mta mailutils
```
- msmtp: lightweight SMTP client
- msmtp-mta: makes msmtp the system’s default sendmail
- mailutils: provides the `mail` command

---

## 2) Create the global msmtp config (/etc/msmtprc)
```
sudo nano /etc/msmtprc
```

**Add below detail for Gmail / Google Workspace**
```
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        default
host           smtp.gmail.com
port           587
from           youremail@gmail.com
user           youremail@gmail.com
password       your_app_password
```
Notes:
- Edit with your SMTP details:
- For Gmail use an app password (not your login password).
- `logfile ~/.msmtp.log` writes per‑user logs to each user’s home directory (avoids permissions issues).

---

## 3) Secure the msmtp config
```
sudo chmod 644 /etc/msmtprc
sudo chown root:root /etc/msmtprc
```
- World‑readable (644) so all users/cron jobs can use it, writable only by root.

---

## 4) Test sending an email

**Using msmtp directly (shows debug details)**
```
echo -e "To: you@yourdomain.com\nSubject: Test Email\n\nThis is a test from $(hostname) at $(date)" \
| msmtp --debug --from=default you@yourdomain.com
```

**Or using the simpler mail command (for scripts/cron)**
```
echo "This is a test from $(hostname) at $(date)" \
| mail -s "Server Email Test" you@yourdomain.com
```
- Change `you@yourdomain.com`
- Check your inbox (and spam). If the email didn’t arrive, see logs below.

---


## 5) Logs & troubleshooting

**View per‑user log (created by logfile ~/.msmtp.log)**
```
cat ~/.msmtp.log
```

If mail fails:
- Recheck `/etc/msmtprc` credentials, host, and port (587 or 465).
- Use app passwords for Gmail accounts.
- Ensure outbound SMTP ports are allowed by your provider.
- Test with verbose msmtp:
```
echo -e "To: you@yourdomain.com\nSubject: Debug Test\n\nBody" \
| msmtp --debug --from=default you@yourdomain.com
```

---


## Security notes

- Keep `/etc/msmtprc` readable (644) and owned by root; do not expose passwords publicly.
- Prefer app passwords over primary account passwords.
- Test:      `echo "body" | mail -s "Subject" you@yourdomain.com`
- Use:       Put `mail` calls in cron/scripts (with `||` to notify on error)
- Logs:      `~/.msmtp.log` per user

Your server is now ready to send email notifications from cron jobs, scripts, and system tasks via your third‑party SMTP provider.
