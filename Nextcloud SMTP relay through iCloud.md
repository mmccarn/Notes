# Nextcloud SMTP relay through iCloud

## iCloud
* App password required...

# Nextcloud
* Settings -> Administration -> Basic Settings
  * Email server<br>
    Send mode: SMTP / STARTTLS<br>
    From address: [username] @ icloud.com<br>
    Authentication method: Plain / Authentication required<br>
    Server address: smtp.mail.me.com : 587<br>
    Credentials: [username] @ icloud.com / [app-specific password generated at https://appleid.apple.com]
