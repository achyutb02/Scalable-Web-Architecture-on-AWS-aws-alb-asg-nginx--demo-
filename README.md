# AWS ALB + Auto Scaling (Nginx demo)

Application Load Balancer → Target Group → Auto Scaling Group (2–4 instances) serving Nginx.
Target-tracking scales **out** when `ALB RequestCountPerTarget > 60` and **in** when idle.

## Architecture
ALB (SG: inbound 80 from 0.0.0.0/0) → TG (HTTP:80, `/` health) → ASG across 2 AZs  
Instances SG allows 80 **only from the ALB SG**.

![diagram](screenshots/architecture.png)

## Launch template user data (Nginx)
Launch Template user data: [`scripts/user-data.sh`](scripts/user-data.sh)

