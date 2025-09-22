# 🛡️ Snort IDS Complete Setup Guide

## 📋 Overview

Snort is a powerful open-source Network Intrusion Detection System (IDS) that can monitor network traffic in real-time and detect suspicious activities. This guide provides a comprehensive walkthrough for installing, configuring, and running Snort on Linux systems.

---

## 🎯 What You'll Learn

- ✅ Install and verify Snort installation
- ✅ Configure Snort for your network environment  
- ✅ Create custom detection rules
- ✅ Run Snort in detection mode
- ✅ Set up logging and alerting
- ✅ Test and validate your IDS setup

---

## 🔧 Prerequisites

- Linux system (Ubuntu/Debian recommended)
- Root or sudo privileges
- Basic understanding of networking concepts
- Network interface knowledge

---

## 📦 Step 1: Install Snort

### Install Snort Package
```bash
sudo apt-get install snort -y
```

**🔍 What this does:**
- Downloads and installs Snort IDS from the official repositories
- Includes all necessary dependencies
- Sets up basic directory structure in `/etc/snort/`

### Verify Installation
```bash
snort --version
```

**🔍 What this does:**
- Displays the installed Snort version
- Confirms successful installation
- Shows build information and supported features

---

## 📝 Step 2: Install Text Editor (Optional)

```bash
sudo apt-get install vim -y
```

**🔍 What this does:**
- Installs Vim text editor for configuration file editing
- Alternative: You can use `nano`, `gedit`, or any preferred editor
- Essential for editing Snort configuration files

---

## ⚙️ Step 3: Configure Snort

### 🏠 Step 3.1: Edit Main Configuration File

Open the main Snort configuration file:
```bash
sudo vim /etc/snort/snort.conf
```

#### Configure Network Variables

Find and modify these critical settings:

```bash
# Define your home network (adjust to your network)
var HOME_NET 192.168.1.0/24

# Define external networks (everything else)
var EXTERNAL_NET any
```

**🔍 What these variables do:**
- `HOME_NET`: Defines your internal/trusted network range
- `EXTERNAL_NET`: Defines external/untrusted networks
- Snort uses these to determine traffic direction and apply appropriate rules

#### Enable Local Rules

Ensure this line is present and uncommented:
```bash
include $RULE_PATH/local.rules
```

**🔍 What this does:**
- Includes your custom detection rules
- `$RULE_PATH` typically points to `/etc/snort/rules/`
- Allows you to create custom detection signatures

---

### 📋 Step 3.2: Create Custom Detection Rules

Open the local rules file:
```bash
sudo vim /etc/snort/rules/local.rules
```

Add these detection rules:

```bash
# ICMP/Ping Detection
alert icmp any any -> 192.168.1.36 any (msg:"Ping Detected"; sid:100001; rev:1;)

# SSH Authentication Attempts
alert tcp any any -> 192.168.1.36 22 (msg:"SSH authentication attempts"; sid:100002; rev:1;)

# Port Scan Detection Rules
alert tcp any any -> 192.168.1.36 20 (msg:"NMAP Detection on port 20"; sid:100003; rev:1;)
alert tcp any any -> 192.168.1.36 80 (msg:"NMAP Detection on port 80"; sid:100004; rev:1;)
alert tcp any any -> 192.168.1.36 443 (msg:"NMAP Detection on port 443"; sid:100005; rev:1;)

# Traffic Rejection Rules
reject tcp any any -> 192.168.1.36 20 (msg:"NMAP Scanning Rejected on port 20"; sid:100006; rev:1;)
reject tcp any any -> 192.168.1.36 80 (msg:"NMAP Scanning Rejected on port 80"; sid:100007; rev:1;)
reject tcp any any -> 192.168.1.36 443 (msg:"NMAP Scanning Rejected on port 443"; sid:100008; rev:1;)
reject icmp any any -> 192.168.1.36 any (msg:"ping rejected"; sid:100009; rev:1;)
```

#### 🧩 Rule Structure Breakdown

Each Snort rule follows this format:
```
[action] [protocol] [src_ip] [src_port] -> [dst_ip] [dst_port] ([rule_options])
```

**Components explained:**
- `alert/reject`: Action to take when rule matches
- `tcp/icmp/udp`: Network protocol to monitor
- `any`: Wildcard for any IP address or port
- `->`: Traffic direction (source to destination)
- `192.168.1.36`: Target IP address (adjust to your system)
- `msg`: Alert message description
- `sid`: Unique rule identifier (must be unique)
- `rev`: Rule revision number

---

### ✅ Step 3.3: Validate Configuration

Test your Snort configuration:
```bash
sudo snort -T -c /etc/snort/snort.conf
```

**🔍 What this does:**
- `-T`: Test mode (validates configuration without running)
- `-c`: Specifies configuration file path
- Checks for syntax errors and rule conflicts
- Validates file paths and permissions

**Expected output should end with:**
```
Snort successfully validated the configuration!
```

---

## 🚀 Step 4: Run Snort in Detection Mode

### Find Your Network Interface

First, identify your network interface:
```bash
ip a
```

**🔍 What this shows:**
- Lists all network interfaces
- Shows IP addresses and interface status
- Common interfaces: `eth0`, `wlan0`, `enp0s3`

### Start Snort IDS

```bash
sudo snort -A console -q -c /etc/snort/snort.conf -i <network_interface>
```

**🔍 Command parameters:**
- `-A console`: Display alerts on console in real-time
- `-q`: Quiet mode (reduces verbose output)
- `-c`: Configuration file path
- `-i`: Network interface to monitor

**Example:**
```bash
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0
```

---

## 📊 Step 5: Set Up Alert Logging

### Configure Alert File Output

Edit the Snort configuration file:
```bash
sudo gedit /etc/snort/snort.conf
```

Find and uncomment or add this line:
```bash
output alert_fast: /var/log/snort/alert
```

**🔍 What this does:**
- Enables fast alert logging format
- Saves alerts to `/var/log/snort/alert` file
- Creates persistent log records for analysis

### Run Snort with File Logging

```bash
sudo snort -A fast -q -c /etc/snort/snort.conf -i <interface>
```

**🔍 Parameters:**
- `-A fast`: Use fast alert format for file output
- Alerts are written to the configured log file

---

## 🧪 Step 6: Test and Validate

### Generate Test Traffic

From another machine or terminal, generate traffic that matches your rules:

```bash
# Test ping detection
ping 192.168.1.36

# Test port scanning detection  
nmap -p 20,80,443 192.168.1.36

# Test SSH connection
ssh user@192.168.1.36
```

### Check Log Files

Verify logs are being created:
```bash
ls /var/log/snort
```

### View Alert Details

```bash
sudo vim /var/log/snort/alert
```

**🔍 What you'll see:**
- Timestamp of detected events
- Rule messages that triggered
- Source and destination IP addresses
- Protocol and port information

---

## 📈 Understanding Snort Output

### Console Alert Format
```
[**] [1:100001:1] Ping Detected [**]
[Priority: 0] 
01/15-10:30:45.123456 192.168.1.100 -> 192.168.1.36
ICMP TTL:64 TOS:0x0 ID:12345 IpLen:20 DgmLen:84
```

### Log File Format
```
01/15-10:30:45.123456  [**] [1:100001:1] Ping Detected [**] [Priority: 0] {ICMP} 192.168.1.100 -> 192.168.1.36
```

---

## 🔧 Troubleshooting Tips

### Common Issues

1. **Permission Denied**
   ```bash
   # Ensure Snort runs with sudo privileges
   sudo snort [options]
   ```

2. **Interface Not Found**
   ```bash
   # Verify interface name with:
   ip a
   # Use exact interface name from output
   ```

3. **Configuration Errors**
   ```bash
   # Test configuration first:
   sudo snort -T -c /etc/snort/snort.conf
   ```

4. **No Alerts Generated**
   - Verify network traffic is actually occurring
   - Check rule syntax and SID uniqueness
   - Ensure HOME_NET matches your network

---

## 🎯 Best Practices

### Security Considerations
- 🔒 Run Snort with minimal necessary privileges
- 🔄 Regularly update rule signatures
- 📱 Monitor log file sizes and implement rotation
- 🛡️ Use separate interface for monitoring when possible

### Performance Optimization
- ⚡ Disable unused rule categories
- 📊 Monitor system resources during operation
- 🎛️ Tune preprocessors for your environment
- 💾 Use fast storage for log files

### Rule Management
- 📋 Document custom rules with clear descriptions
- 🔢 Use systematic SID numbering (100000+ for custom)
- 🔄 Test rules thoroughly before production deployment
- 📚 Keep rule comments updated and meaningful

---

## 📚 Additional Resources

- [Official Snort Documentation](https://snort.org/documents)
- [Snort Rule Writing Guide](https://snort.org/documents/snort-rule-infographics)
- [Community Rules Repository](https://snort.org/rules)

---

## 🏁 Summary

You've successfully:
- ✅ Installed Snort IDS on your system
- ✅ Configured network variables and custom rules
- ✅ Set up real-time monitoring and logging
- ✅ Created detection rules for common threats
- ✅ Established a functional network security monitoring system

Your Snort IDS is now ready to detect and alert on suspicious network activities! 🛡️

---

*💡 **Pro Tip**: Regularly review and analyze your Snort logs to understand normal network patterns and improve detection accuracy.*