# Raspberry Pi SSH & Network Debugging Guide (RIS Project)

## Goal

This guide explains:

- How to connect Raspberry Pi through SSH
- How to debug SSH issues
- How to find Raspberry Pi IP
- Why SSH suddenly stopped working
- How we fixed the IP conflict issue
- How to permanently fix Raspberry Pi IP using DHCP Reservation

---

# 1. System Setup

## Hardware

- Raspberry Pi 5
- Wi-Fi connection
- Windows PC
- TP-Link Router
- Raspberry Pi running headless (no HDMI monitor)

---

## Normal Connection Method

Windows CMD:

```cmd
ssh pi@192.168.0.26
```

Example:

```cmd
ssh pi@192.168.0.26
```

After password:

```txt
pi@raspberrypi:~ $
```

This means SSH login success.

---

# 2. Problem Faced

## Symptom

Earlier:

```cmd
ssh pi@192.168.0.18
```

was working.

Suddenly:

```txt
connection refused
```

came.

VNC also stopped working.

---

## Initial Suspicion

We thought:

- Raspberry Pi IP changed
- Someone changed configuration
- SSH service stopped
- Raspberry Pi disconnected from Wi-Fi

---

# 3. Investigation Steps

## Step 1 — Check Windows IP

Command:

```cmd
ipconfig
```

Output:

```txt
IPv4 Address : 192.168.0.18
```

Observation:

Windows PC itself had:

```txt
192.168.0.18
```

This was suspicious because Raspberry Pi was also expected on `.18`.

---

## Step 2 — Check Router Connected Devices

Router:

```txt
192.168.0.1
```

Navigate:

```txt
Advanced → Network Map
```

Observed:

- Raspberry Pi not clearly visible
- Multiple Unknown devices
- One MAC looked suspicious

---

## Step 3 — Check Raspberry Pi via Hostname

Command:

```cmd
ping raspberrypi.local
```

Output:

```txt
Reply from 2406:....
```

Meaning:

- Raspberry Pi ON
- Connected to network
- Hostname resolution working
- IPv6 working

---

## Step 4 — SSH through IPv6

Command:

```cmd
ssh -6 pi@raspberrypi.local
```

Result:

```txt
pi@raspberrypi:~ $
```

SSH success.

---

# 4. Root Cause Analysis

## Why SSH stopped working?

We checked Raspberry Pi network configuration.

Command:

```bash
nmcli connection show preconfigured | grep ipv4
```

Output:

```txt
ipv4.method: manual
ipv4.addresses: 192.168.0.18/24
```

Meaning:

Raspberry Pi had:

```txt
manual static IP = 192.168.0.18
```

---

## What happened?

Router DHCP gave:

```txt
192.168.0.18
```

to Windows PC.

At the same time:

```txt
Pi also tried using 192.168.0.18
```

Result:

### IP Conflict

Two devices had same IP.

```txt
PC  = 192.168.0.18
Pi  = 192.168.0.18
```

---

## Effect of Conflict

Raspberry Pi lost IPv4.

Command:

```bash
ip addr show wlan0
```

Output:

```txt
inet6 only
```

Missing:

```txt
inet 192.168.x.x
```

Meaning:

```txt
IPv4 broken
Only IPv6 alive
```

This is why:

```cmd
ssh pi@192.168.0.18
```

failed.

---

# 5. Fix Applied

## Step 1 — Change Manual IP to Auto DHCP

Command:

```bash
sudo nmcli connection modify preconfigured ipv4.method auto
```

Reconnect Wi-Fi:

```bash
sudo nmcli connection up preconfigured
```

SSH disconnected automatically:

```txt
client_loop: send disconnect: Connection reset
```

This is NORMAL.

Reason:

Wi-Fi reconnect occurred.

---

## Step 2 — Reconnect Raspberry Pi

Command:

```cmd
ssh -6 pi@raspberrypi.local
```

---

## Step 3 — Verify New IP

Command:

```bash
hostname -I
```

Output:

```txt
192.168.0.26
2406:b400....
```

Meaning:

New IPv4 assigned successfully.

---

## Step 4 — Verify Wi-Fi Interface

Command:

```bash
ip addr show wlan0
```

Output:

```txt
inet 192.168.0.26/24
```

Meaning:

IPv4 restored successfully.

---

# 6. Permanent Fix (DHCP Reservation)

## Why?

To avoid:

```txt
IP changing every day
```

or

```txt
IP conflict again
```

---

## Raspberry Pi MAC Address

Command:

```bash
cat /sys/class/net/wlan0/address
```

Output:

```txt
88:a2:9e:4c:9a:c5
```

---

## Router Configuration

Login:

```txt
192.168.0.1
```

Navigate:

```txt
Advanced
→ Network
→ DHCP Server
→ Address Reservation
```

Add:

```txt
MAC Address:
88:A2:9E:4C:9A:C5

Reserved IP:
192.168.0.26
```

Enable:

```txt
Enable This Entry = ON
```

Save.

---

## Verification

Restart Raspberry Pi:

```bash
sudo reboot
```

Reconnect:

```cmd
ssh pi@192.168.0.26
```

If login works:

```txt
SUCCESS
```

Meaning:

Static-like permanent IP configured.

---

# 7. Final Stable Setup

## Raspberry Pi Info

```txt
Hostname:
raspberrypi

IP:
192.168.0.26

MAC:
88:A2:9E:4C:9A:C5
```

---

## Final SSH Command

```cmd
ssh pi@192.168.0.26
```

---

# 8. Future Troubleshooting Checklist

If SSH stops working:

## 1. Check PC IP

```cmd
ipconfig
```

---

## 2. Ping Raspberry Pi

```cmd
ping raspberrypi.local
```

---

## 3. Try IPv6 SSH

```cmd
ssh -6 pi@raspberrypi.local
```

---

## 4. Check Pi IP

Inside Pi:

```bash
hostname -I
```

---

## 5. Check Wi-Fi Interface

```bash
ip addr show wlan0
```

---

## 6. Check IPv4 Config

```bash
nmcli connection show preconfigured | grep ipv4
```

---

## 7. Fix DHCP Mode

```bash
sudo nmcli connection modify preconfigured ipv4.method auto
sudo nmcli connection up preconfigured
```

---

# 9. Key Learnings

1. Same IP on two devices causes IP conflict.

2. Manual static IP can break networking if router gives same IP to another device.

3. DHCP Reservation is safer than manually forcing static IP inside Raspberry Pi.

4. IPv6 can still work even if IPv4 breaks.

5. SSH disconnect during Wi-Fi reconnect is normal.

6. `raspberrypi.local` is useful for recovery.
