# Ubuntu 22.04 Postfix-Dovecot Email Server Setup Commands

## Step 1: Create Users
```bash
sudo useradd -m user1
sudo passwd user1
sudo useradd -m user2
sudo passwd user2
```

## Step 2: Install Postfix
```bash
sudo apt-get update
sudo apt-get install postfix
```

**During Postfix installation prompts:**
- General type of mail configuration: **Internet Site**
- System mail name: **esprit.com** (not mail.esprit.com)

## Step 3: Configure Postfix
```bash
sudo gedit /etc/postfix/main.cf
```

**Add/modify these lines in /etc/postfix/main.cf:**
```
# Specify the hostname for this mail server.
myhostname = mail.esprit.com

# Define the origin of locally-posted mail
myorigin = /etc/mailname

# List the destination domains for which this mail server will receive mail
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Leave the relay host empty, indicating this server performs local mail delivery.
relayhost = 

# Specify a list of trusted networks that can relay mail through this server.
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128, 192.168.231.0/24

# Set the mailbox size limit to 0, indicating unlimited mailbox size.
mailbox_size_limit = 0

# Define the recipient delimiter used in recipient addresses.
recipient_delimiter = +

# Specify that the server should listen on all network interfaces.
inet_interfaces = all

# Allow all network protocols.
inet_protocols = all

# Define the mailbox format and location for user mailboxes.
home_mailbox = Maildir/
```

## Step 4: Create System Aliases
```bash
sudo gedit /etc/aliases
```

**Add to /etc/aliases file:**
```
user1: user1@esprit.com
user2: user2@esprit.com
```

**Update aliases database:**
```bash
sudo newaliases
```

## Step 5: Configure Firewall
```bash
sudo apt install ufw
sudo ufw allow 25/tcp
sudo ufw allow 110/tcp
sudo ufw allow 143/tcp
sudo ufw reload
sudo ufw enable
sudo ufw status
```

## Step 6: Start Postfix Service
```bash
sudo systemctl restart postfix
```

## Step 7: Test Postfix (Telnet Commands)
```bash
telnet localhost smtp
```

**In telnet session, type these commands:**
```
ehlo localhost
mail from:user1@esprit.com
rcpt to:user2@esprit.com
data
test
.
quit
```

## Step 8: Install Dovecot
```bash
sudo apt-get install dovecot-imapd dovecot-pop3d
```

## Step 9: Configure Dovecot

### Edit /etc/dovecot/dovecot.conf
```bash
sudo gedit /etc/dovecot/dovecot.conf
```

**Uncomment line 24:**
```
!include_try /usr/share/dovecot/protocols.d/*.protocol
```

**Uncomment line 30:**
```
listen = *, ::
```

### Edit /etc/dovecot/conf.d/10-mail.conf
```bash
sudo gedit /etc/dovecot/conf.d/10-mail.conf
```

**Uncomment line 24:**
```
mail_location = maildir:~/Maildir
```

### Edit /etc/dovecot/conf.d/10-auth.conf
```bash
sudo gedit /etc/dovecot/conf.d/10-auth.conf
```

**Line 10 - uncomment and change:**
```
disable_plaintext_auth = no
```

**Line 100 - add "login":**
```
auth_mechanisms = plain login
```

### Edit /etc/dovecot/conf.d/10-master.conf
```bash
sudo gedit /etc/dovecot/conf.d/10-master.conf
```

**Lines 107-109 - uncomment and add:**
```
# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
  mode = 0666
  user = postfix
  group = postfix
}
```

### Restart Dovecot
```bash
sudo systemctl restart dovecot
sudo systemctl enable dovecot
```

## Step 10: Test Dovecot (Telnet Commands)
```bash
telnet localhost pop3
```

**In telnet session, type:**
```
user user2
pass user2_pass
list
retr 1
quit
```

## Thunderbird Configuration Settings

**For each user account, use these settings:**

- **Incoming Server (IMAP):**
  - Server: [your server IP]
  - Port: 143
  - Security: None
  - Authentication: Normal password

- **Incoming Server (POP3):**
  - Server: [your server IP] 
  - Port: 110
  - Security: None
  - Authentication: Normal password

- **Outgoing Server (SMTP):**
  - Server: [your server IP]
  - Port: 25
  - Security: None
  - Authentication: Normal password

**User Accounts:**
- user1@esprit.com
- user2@esprit.com

## Notes:
- Replace `192.168.231.0/24` with your actual network range
- Replace `[your server IP]` with your actual server IP address
- Make sure to set proper passwords for user1 and user2
- The domain used is `esprit.com` throughout the configuration
