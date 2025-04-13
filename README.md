---
title: "Connexion des nœuds du cluster Proxmox au réseau WiFi."
categories:
  - Blog
tags:
  - WiFi
  - PROXMOX
---

# 5. Connexion des nœuds du cluster Proxmox au réseau WiFi.

https://debian-facile.org/doc:reseau:wpasupplicant

## Prerequisites

### Hardware Requirements
- 2x nodes with WiFi module
- WiFi “FreeWiFI”
- Password “VotrePassword”

**Pour connecter l'interface Wi-Fi sur chaque nœud et connecter les nœuds via un réseau Wi-Fi, vous devez effectuer les étapes suivantes :**

### 1. Connexion de l'interface Wi-Fi

## Installation des packages requis

**Installer des packages Wi-Fi sur tous les nœuds :**
```
apt update
apt install wireless-tools wpasupplicant net-tools
```
### 2. Recherchez l’interface appropriée :
```
root@pve1:~# dmesg | grep wlan
[    9.795917] iwlwifi 0000:00:14.3 wlo1: renamed from wlan0
root@pve1:~#

[root@pve99 ~]$ dmesg | grep wlan
[   24.504063] mt7601u 5-1.1:1.0 wlx7601dd602347: renamed from wlan0
[root@pve99 ~]$
```
### 3. Configuration d'un réseau Wi-Fi

**sur chaque nœud pour configurer le Wi-Fi Modifier le fichier**
```
 nano /etc/network/interfaces
```
**Exemple de fichier pour le nœud 1 :**
```
auto lo
iface lo inet loopback

auto wlo1
iface wlo1 inet dhcp
        wpa-ssid "FreeWiFi"
        wpa-psk "VotrePassword"
auto vmbr0
iface vmbr0 inet static
        address 10.10.3.1/24
        bridge-ports none
        bridge -stp off
        bridge-fd 0
# INTERNET
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s 10.10.3.0/24 -o wlo1 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s 10.10.3.0/24 -o wlo1 -j MASQUERADE
source /etc/network/interfaces.d/*
```
**Exemple pour le nœud 2 :**
```
auto lo
iface lo inet loopback

auto wlx7601dd602347
iface wlx7601dd602347 inet dhcp
       wpa-ssid "FreeWiFi"
       wpa-psk "VotrePassword"

auto vmbr0
iface vmbr0 inet static
        address 10.10.3.1/24
        bridge-ports none
        bridge -stp off
        bridge-fd 0
# INTERNET
   post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
   post-up   iptables -t nat -A POSTROUTING -s 10.10.3.0/24 -o wlx7601dd602347 -j MASQUERADE
   post-down iptables -t nat -D POSTROUTING -s 10.10.3.0/24 -o wlx7601dd602347 -j MASQUERADE
source /etc/network/interfaces.d/*
```
**wpa-ssid : Le nom de votre réseau Wi-Fi.**
**wpa-psk : mot de passe réseau.**

### 4. Configuration DNS
```
nano /etc/resolve.conf
```
**Exemple de fichier :**
```
nameserver 192.168.1.254
nameserver 8.8.8.8 
```
### 5. Configuration de wpa_supplicant
```
wpa_passphrase votre-ssid votre-mot-de-passe >> /etc/wpa_supplicant/wpa_supplicant.conf
```

```
nano /etc/wpa_supplicant/wpa_supplicant.conf
```
**Exemple de fichier :**
```
network={
	ssid="FreeWiFi"
	#psk="*****"
	psk=360b2c805ecd920b79a370af532d2f7636bab7049ed2dc068c2dae17f5e1c38e
}
```

#ITEnterprise #ContinuitéActivité #InfrastructureSécurisée
