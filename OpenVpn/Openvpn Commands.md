
# OpenVPN Setup Guide
## Server Setup

### Step 1: Install OpenVPN and Easy-RSA

```bash
sudo apt install openvpn easy-rsa
```

### Step 2: Initialize Easy-RSA PKI

```bash
make-cadir ~/easy-rsa
cd ~/easy-rsa
./easyrsa init-pki
```

### Step 3: Build CA (Certificate Authority)

```bash
./easyrsa build-ca
```

### Step 4: Generate Server Key Pair

```bash
./easyrsa gen-req server nopass  
./easyrsa sign-req server server
```

### Step 5: Generate Diffie-Hellman Parameters

```bash
./easyrsa gen-dh
```

### Step 6: Copy Certificates and Keys to OpenVPN Directory

```bash
sudo cp pki/ca.crt /etc/openvpn/
sudo cp pki/issued/server.crt /etc/openvpn/
sudo cp pki/private/server.key /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/
```

### Server Configuration (server.conf)

```bash
# OpenVPN Server Configuration
port 1194
proto udp
dev tun

# Certificates and Keys settings 
ca /etc/openvpn/ca.crt 
cert /etc/openvpn/server.crt 
key /etc/openvpn/server.key 
dh /etc/openvpn/dh.pem 

# Set topology to subnet for compatibility
topology subnet

# VPN Subnet Configuration 
server 10.8.0.0 255.255.255.0 

# Push routes to clients 
push "redirect-gateway def1 bypass-dhcp" 
push "dhcp-option DNS 8.8.8.8" 
push "dhcp-option DNS 8.8.4.4" 

# Keepalive settings 
keepalive 10 120 

# Data ciphers for encryption (recommended)
data-ciphers AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305

# User and Group settings for security 
user nobody 
group nogroup 

# Persist settings 
persist-key 
persist-tun 

# Logging verbosity level 
verb 3 

# Optional: Enable client-to-client communication
client-to-client
```

## Client Setup

### Step 1: Generate Client Key Pair

```bash
./easyrsa gen-req client nopass
./easyrsa sign-req client client
```

### Step 2: Create Client Configuration Directory

```bash
mkdir ~/client-configs
```

### Step 3: Copy Certificates and Keys to Client Config Directory

```bash
cp pki/ca.crt ~/client-configs/
cp ~/easy-rsa/pki/issued/client.crt ~/client-configs/  
cp ~/easy-rsa/pki/private/client.key ~/client-configs/
```

## Server: Transfer Client Config to Remote Machine

```bash
sudo systemctl start ssh
scp ~/client-configs/* alib@192.168.1.8:/home/alib/
```

## Client: Set Up Configuration

### Step 1: Prepare Client Configuration Directory

```bash
mkdir client-configs
cp ca.crt client.crt client.key /client-configs
```

### Step 2: Create `client.ovpn`

```bash
gedit client.ovpn
```

#### client.ovpn:

```bash
client
dev tun
proto udp
remote 192.168.1.2 1194  # Use the server's public IP if connecting from outside

resolv-retry infinite
nobind
persist-key
persist-tun

ca /home/alib/client-configs/ca.crt
cert /home/alib/client-configs/client.crt
key /home/alib/client-configs/client.key

cipher AES-256-GCM
verb 3

# Optional: Enable TLS authentication (if used)
# tls-auth /home/alib/client-configs/ta.key 1  # Uncomment if using a TLS key
```

### Step 3: Install OpenVPN on the Client

```bash
sudo apt install openvpn
```

### Step 4: Connect to the VPN

```bash
sudo openvpn --config client.ovpn
```

## Clean-Up Commands

### Step 1: Stop and Disable OpenVPN Service

```bash
sudo systemctl stop openvpn@server
sudo systemctl disable openvpn@server
```

### Step 2: Remove OpenVPN Configuration Files

```bash
sudo rm -rf /etc/openvpn/*
rm -rf ~/openvpn-ca
```

### Step 3: Remove OpenVPN Port from Firewall

```bash
sudo ufw delete allow 1194/udp
sudo ufw status
```

### Step 4: Disable IP Forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=0  # or sudo nano /etc/sysctl.conf and comment the line
sudo sysctl -p
```

### Step 5: Check OpenVPN Service Status

```bash
sudo systemctl status openvpn@server
```
