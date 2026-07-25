# Secure Two-Tier Network on Google Cloud

> A hands-on lab building a segmented VPC with least-privilege firewall rules on GCP.
> **Focus areas:** Cloud Networking · Network Security · GCP Fundamentals
>
> *Part 1 of a multi-phase project building toward a secure, highly-available cloud application.*

---

## Objective

Design and deploy a custom private network on Google Cloud that separates public-facing
and backend resources, and enforce traffic control with least-privilege firewall rules.

The finished environment has:

- A custom VPC (not the default one) split into a **public** and a **private** subnet
- A **web server** in the public subnet, reachable from the internet on HTTP only
- A **backend server** in the private subnet with **no public IP**, unreachable from the internet
- **Firewall rules** that allow only the traffic each tier needs — nothing more
- **Cloud NAT** so the private server can pull updates outbound without being exposed inbound

---

## Architecture

```
                          Internet
                             |
                     [ HTTP :80 / SSH :22 ]      <-- firewall: allow, restricted source
                             |
                 +-----------v-----------+
                 |   PUBLIC SUBNET       |   10.0.1.0/24
                 |   web-server (nginx)  |   external IP
                 |   tag: web-server     |
                 +-----------+-----------+
                             |
                     [ internal only ]           <-- firewall: allow from 10.0.1.0/24
                             |
                 +-----------v-----------+
                 |   PRIVATE SUBNET      |   10.0.2.0/24
                 |   db-server           |   NO external IP
                 |   tag: db-server      |
                 +-----------+-----------+
                             |
                     [ outbound only ]           <-- Cloud NAT (egress for updates)
                             |
                          Internet
```

**The security idea:** the private tier can *reach out* (for patches/updates) but nothing
on the internet can *reach in*. The public tier is the only exposed surface, and even it
only accepts the two ports it actually needs.

---

## How it was built (`gcloud` CLI)

The whole environment, provisioned from Cloud Shell:

```bash
# 1. Custom VPC
gcloud compute networks create secure-vpc --subnet-mode=custom

# 2. Subnets
gcloud compute networks subnets create public-subnet \
  --network=secure-vpc --region=us-central1 --range=10.0.1.0/24
gcloud compute networks subnets create private-subnet \
  --network=secure-vpc --region=us-central1 --range=10.0.2.0/24

# 3. Firewall rules (VPC firewall rules — these use network tags)
#    NOTE: allow-ssh is scoped to a single admin IP, never 0.0.0.0/0
gcloud compute firewall-rules create allow-http \
  --network=secure-vpc --direction=INGRESS --action=ALLOW \
  --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=web-server
gcloud compute firewall-rules create allow-ssh \
  --network=secure-vpc --direction=INGRESS --action=ALLOW \
  --rules=tcp:22 --source-ranges=YOUR_IP/32 --target-tags=web-server
gcloud compute firewall-rules create allow-internal \
  --network=secure-vpc --direction=INGRESS --action=ALLOW \
  --rules=tcp:3306,icmp --source-ranges=10.0.1.0/24 --target-tags=db-server

# 4. Public web server (with startup script installing nginx)
gcloud compute instances create web-server \
  --zone=us-central1-a --machine-type=e2-micro \
  --subnet=public-subnet --tags=web-server \
  --metadata=startup-script='#! /bin/bash
apt update && apt install -y nginx
echo "<h1>Hello from my secure VPC web tier</h1>" > /var/www/html/index.html'

# 5. Private backend server (no public IP)
gcloud compute instances create db-server \
  --zone=us-central1-a --machine-type=e2-micro \
  --subnet=private-subnet --tags=db-server --no-address

# 6. Cloud Router + NAT for private egress
gcloud compute routers create nat-router \
  --network=secure-vpc --region=us-central1
gcloud compute routers nats create nat-config \
  --router=nat-router --region=us-central1 \
  --nat-all-subnet-ip-ranges --auto-allocate-nat-external-ips
```

---

## Verification

| Test | Expected result |
|---|---|
| Browse to `web-server` external IP | nginx page loads ✅ |
| Try to reach `db-server` from internet | No external IP exists — unreachable ✅ |
| From `db-server`, run `curl -I https://www.google.com` | Response returns — outbound via NAT works ✅ |
| SSH to `db-server` directly from internet | Blocked (no public IP; access via bastion/IAP only) ✅ |

Screenshots of each step are in [`/images`](./images).

---

## Skills demonstrated

Cloud networking (VPC, subnets, CIDR) · Network segmentation · Least-privilege firewall
design · Cloud NAT / egress control · Compute Engine · Secure administrative access
(bastion / IAP concept) · Infrastructure via CLI (`gcloud`)

---

## What I learned

While building the firewall layer I ran into an instructive distinction. My first attempt
used a **network firewall policy** (a global VPC policy), and I couldn't get it to scope to
just my web server — it kept applying to all instances. The reason: **network firewall
policies don't read network tags** — they use IAM-governed *secure tags*. The simpler **VPC
firewall rules** are the ones that honor the `web-server` / `db-server` network tags I'd
placed on the VMs. Understanding when to use each — VPC firewall rules (tag-based, simple)
versus network firewall policies (secure-tag-based, IAM-controlled, hierarchical, and able
to layer in threat-intelligence rules) — was the most valuable takeaway from this lab.

The other habit I locked in was scoping SSH access to a single admin IP rather than
`0.0.0.0/0`, and setting a billing budget alert before provisioning anything.

*(Edit this into your own voice — a recruiter can tell the difference, and your real
experience is more convincing than anything generic.)*

---

## Cleanup

All resources were torn down after the lab to avoid unnecessary cost:

```bash
gcloud compute instances delete web-server db-server --zone=us-central1-a -q
gcloud compute routers delete nat-router --region=us-central1 -q
gcloud compute firewall-rules delete allow-http allow-ssh allow-internal -q
gcloud compute networks subnets delete public-subnet private-subnet --region=us-central1 -q
gcloud compute networks delete secure-vpc -q
```

---

## Next in this series

- **Phase 2** — Load-balanced, highly-available web tier (managed instance group + HTTP load balancer)
- **Phase 3** — Security hardening (IAM least privilege, Cloud Armor WAF, Secret Manager)
- **Phase 4** — Rebuild everything as Infrastructure-as-Code with Terraform
- **Phase 5** — Monitoring, logging & alerting
- **Phase 6** — Capstone: the full secure, HA cloud application

---

*Lab completed: 2026-07-25 · Cloud provider: Google Cloud Platform*
