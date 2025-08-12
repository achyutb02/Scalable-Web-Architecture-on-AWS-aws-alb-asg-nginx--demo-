# AWS ALB + Auto Scaling (Nginx demo)

Application Load Balancer → Target Group → Auto Scaling Group (2–4 instances) serving Nginx.
Target-tracking scales **out** when `ALB RequestCountPerTarget > 60` and **in** when idle.

## Architecture
ALB (SG: inbound 80 from 0.0.0.0/0) → TG (HTTP:80, `/` health) → ASG across 2 AZs  
Instances SG allows 80 **only from the ALB SG**.

![diagram](screenshots/architecture.png)

## Launch template user data (Nginx)
```bash
#!/bin/bash
set -euxo pipefail
dnf -y update
dnf -y install nginx
TOKEN="$(curl -sS -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600' || true)"
md(){ local p="$1"; if [ -n "$TOKEN" ]; then curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254${p}"; else curl -sS "http://169.254.169.254${p}"; fi; }
IID="$(md /latest/meta-data/instance-id)"
AZ="$(md /latest/meta-data/placement/availability-zone)"
HOSTNAME_FQDN="$(hostname -f || hostname)"
cat >/usr/share/nginx/html/index.html <<EOF
<!doctype html><html><head><meta charset="utf-8"><title>ASG Demo</title></head>
<body style="font-family:system-ui;max-width:720px;margin:40px auto;">
<h1>Hello from $IID</h1><p>Hostname: $HOSTNAME_FQDN<br>AZ: $AZ</p></body></html>
EOF
systemctl enable --now nginx
