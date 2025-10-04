A safe, local GoPhish lab for coursework: how to install the binary, wire up a systemd service, configure config.json, test SMTP, and safely prepare the repo for GitHub. Do not use this against real targets — only authorized testing accounts and lab VMs.

What’s in this repo

gophish.service — example systemd unit

config.json.example — example config (no secrets)

README.md — this file

.gitignore — suggested ignore rules

notes / scripts used during the lab

Requirements

Linux (Ubuntu tested)

GoPhish binary placed at /opt/gophish/gophish (owned by gophish:gophish)

systemd to run the service

swaks (optional) for SMTP testing

internet access for SMTP & package installs (apt)

Quick start (install & run)

Put the GoPhish binary in /opt/gophish/ and set ownership:

sudo mkdir -p /opt/gophish
sudo chown -R gophish:gophish /opt/gophish    # create user/group first if needed
sudo chmod 750 /opt/gophish/gophish


Example systemd unit (/etc/systemd/system/gophish.service):

[Unit]
Description=GoPhish phishing toolkit
After=network.target

[Service]
User=gophish
Group=gophish
WorkingDirectory=/opt/gophish
ExecStart=/opt/gophish/gophish
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target


After editing:

sudo systemctl daemon-reload
sudo systemctl enable --now gophish
sudo systemctl status gophish


Example config.json (use config.json.example as a template):

{
  "admin_server": {
    "listen_url": "127.0.0.1:3333",
    "use_tls": true,
    "cert_path": "gophish_admin.crt",
    "key_path": "gophish_admin.key"
  },
  "phish_server": {
    "listen_url": "0.0.0.0:8080",
    "use_tls": false,
    "cert_path": "example.crt",
    "key_path": "example.key"
  },
  "db_name": "sqlite3",
  "db_path": "gophish.db",
  "migrations_prefix": "db/db_",
  "contact_address": "",
  "logging": {
    "filename": "",
    "level": ""
  }
}


Notes:

Use port 8080 (non-privileged) unless you explicitly run as root or use a reverse proxy. Binding to port 80 requires root privileges and is more likely to conflict with other services.

Keep TLS certs and gophish.db out of the repo.

Find the generated admin password

When GoPhish runs for the first time it prints the auto-generated admin password in the logs. To show it:

# Show recent journal lines with the generated admin password
sudo journalctl -u gophish -n 200 --no-pager | grep "Please login with the username" -m1


Store or reset the password using the admin UI. (If you restart the service it may create a new admin password only the first time a fresh DB is created — avoid repeatedly deleting DB if you want the same login.)

Testing SMTP (using Gmail as example)

If sending via Gmail (G Suite / Google Workspace) you must use the appropriate authenticated account and an App Password (or OAuth). Determine MX hosts for a domain:

host -t mx hitecuni.edu.pk
# or
dig mx hitecuni.edu.pk +short


Test sending with swaks:

swaks --to recipient@example.tld \
  --from "Muneeb Internee <23-cys-004@student.hitecuni.edu.pk>" \
  --server smtp.gmail.com:587 \
  --auth LOGIN \
  --auth-user '23-cys-004@student.hitecuni.edu.pk' \
  --auth-password 'APP_PASSWORD' \
  --tls


Important: Do not use credentials you do not own or have permission to use; create a test mailbox with consent.

Safe lab / ethical rules (must-read)

Only test on lab machines and on accounts/domains you have explicit permission to use.

Do not target real users, external organizations, or university infrastructure without authorization.

Do not commit credentials, keys, or database files to GitHub. Use .gitignore to exclude them.

Suggested .gitignore:

gophish.db
*.key
*.crt
config.json
*.pem
/*.log

Prepare for GitHub (what to push)

Push: README.md, config.json.example, gophish.service.example, scripts, guides.

Do not push: config.json with secrets, gophish.db, private keys, logs.

Before pushing, run git status and verify .gitignore.

Commands:

git init
git add README.md config.json.example gophish.service.example .gitignore
git commit -m "Initial GoPhish lab files (no secrets)"
git remote add origin git@github.com:youruser/yourrepo.git
git push -u origin main

Common troubleshooting

bind: permission denied on port 80 → use 8080 or run with appropriate privileges.

Failed to locate executable /opt/gophish/gophish → check ExecStart path and file permissions/ownership.

crypto/bcrypt: hashedPassword is not the hash of the given password → wrong password entered on login.

If logs show TLS handshake errors when connecting to admin UI, ensure you accept the self-signed cert in your browser or use http://localhost:3333 if TLS disabled (not recommended).

Networking/DNS/package errors (apt) → check VM network settings, DNS resolver, or proxy.

Helpful commands:

sudo journalctl -u gophish -f          # live logs
sudo systemctl restart gophish
sudo systemctl stop gophish
sudo systemctl status gophish

Cleanup before demo / submission

Stop service: sudo systemctl stop gophish

Remove DB if you created test data and want to reset: rm /opt/gophish/gophish.db (only if appropriate)

Remove TLS keys and other secrets from the VM if handing off the machine.

Make sure repo does NOT include sensitive files.

License & Attribution

Use whatever license your course requires; add a LICENSE file. This repo is a student lab — adapt it for your university assignment.
