Pi-hole Ad Blocker Home Lab Project (Revised & Rebuilt)
Bryan Sykes | Alexandria, VA | May 2026


Overview
This project is a revised and fully rebuilt deployment of Pi-hole, a network-wide DNS-based ad blocker, running in Docker Desktop on Windows 11. Pi-hole intercepts DNS queries at the network level and blocks requests to known ad and tracker domains before they ever reach your devices with no browser extensions required.

This is a revisit of a previous failed deployment. The original container ran for months with zero DNS queries processed. This rebuild documents the full diagnostic process, mistakes made along the way, and the corrected approach that resulted in a fully working deployment.


Environment
Component
Detail
OS
Windows 11
Platform
Docker Desktop for Windows
Pi-hole Version
v4.61.0
Host IP
192.168.1.182 (DHCP reservation via router)
RAM
16GB DDR4
Connection
Ethernet only



What Went Wrong (Original Deployment)
The original container (unruffled_gould) was created via the Docker Desktop GUI while following a video tutorial. Two issues prevented it from ever working:

Wrong UDP port — Port was mapped as 52:53/UDP instead of 53:53/UDP. Since most DNS traffic uses UDP, queries physically could not reach the Pi-hole.
No DNS routing — Devices on the network were never actually pointed at Pi-hole as their DNS server.

Container logs confirmed it immediately:

INFO: -> Total DNS queries: 0

INFO: -> Blocked DNS queries: 0

INFO: -> Unique clients: 0


The Fix
Switched to Docker Compose
Replaced GUI setup with a docker-compose.yml file to eliminate manual entry errors and create a repeatable, auditable configuration.
IP-Bound Port Mapping
Instead of binding to 0.0.0.0 (all interfaces), ports are bound to the specific LAN IP. This avoids Windows port 53 conflicts without touching system services:

ports:

  - "192.168.1.182:53:53/tcp"

  - "192.168.1.182:53:53/udp"

  - "192.168.1.182:80:80/tcp"
Full docker-compose.yml
version: "3"

services:

  pihole:

    container_name: pihole

    image: pihole/pihole:latest

    ports:

      - "192.168.1.182:53:53/tcp"

      - "192.168.1.182:53:53/udp"

      - "192.168.1.182:80:80/tcp"

    environment:

      TZ: 'America/New_York'

      WEBPASSWORD: 'yourpasswordhere'

    volumes:

      - './etc-pihole:/etc/pihole'

      - './etc-dnsmasq.d:/etc/dnsmasq.d'

    restart: unless-stopped


Deployment Steps
# 1. Create project folder

mkdir C:\homelab\pihole

# 2. Navigate to it

cd C:\homelab\pihole

# 3. Create docker-compose.yml (paste content above, set your password)

# 4. Start the container

docker compose up -d

# 5. Verify Pi-hole is responding to DNS queries

nslookup google.com 192.168.1.182


Verification
A successful nslookup response from 192.168.1.182 confirms Pi-hole is live and resolving DNS before any router changes are made. Then point your router's primary DNS to 192.168.1.182 and devices will automatically route through Pi-hole on their next DHCP renewal.


Reset Password
docker exec pihole pihole setpassword yournewpassword


Blocklists Used
List
Source
StevenBlack Hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
Hagezi Pro
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt


After adding lists, go to Tools → Update Gravity in the Pi-hole web UI to apply them.


Results
Metric
Value
Total queries (first session)
517
Queries blocked
23 (4.4%)
Active clients
2
Domains on blocklists
82,208
Pi-hole status
Active



Lessons Learned
Use docker-compose over GUI — eliminates typos in port mappings and creates a repeatable config
Bind to a specific LAN IP — avoids Windows port 53 conflicts without disabling system services
Do not disable Dnscache on Windows — it causes complete network outage; IP-bound ports are the correct solution
Verify with nslookup first — always confirm Pi-hole responds before touching router DNS settings
DHCP reservation in router is the right way to maintain a stable IP, not a static IP set in Windows adapter settings


Dashboard Access
http://192.168.1.182/admin


Related Portfolio Projects
Jellyfin Home Media Server (replaced Plex)
Pelican Game Server - Minecraft + Palworld (in progress)
CTDX Threat Detection Project
AI Job Application Automation



Part of an ongoing home lab portfolio — Alexandria, VA

