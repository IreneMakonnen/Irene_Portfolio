# Snort: Basic Alerting Ruleset

**Objective**:

1. Install Snort 3 on Ubuntu 22.04.

2. Create basic Snort rules to detect common network activities: ICMP pings and Nmap scans.
Identify key characteristics of these activities and translate them into Snort syntax.

3. Create a Snort ruleset that alerts whenever a user attempts to:

    - Perform a standard ICMP ping.
    - Initiate a basic TCP SYN (stealth) scan using Nmap.

## Creating the Ruleset

### Rule 1: Detecting ICMP Ping

Created a Snort rule that triggers an alert whenever an ICMP Echo Request (ping) is detected.

```shell
vim /usr/local/etc/rules/local.rules

alert icmp any any -> any any ( msg:"ICMP Traffic Detected"; sid:10000001; metadata:policy security-ips alert; )
```

Breakdown of the alert rule:

- `alert` - It tells snort what to do when the rule matches, generate an alert when matched
- `icmp` - It only looks at ICMP traffic
- `any any` - Match from any source IP and any source port
- `->` -  A traffic direction indicator. It means from the source to destination
- `any any` - Match to any destination IP and any destination port
- `msg:"ICMP Traffic Detected";` - Text is shown in the alert when the rule is triggered
- `sid:10000001;` -  Unique identifier for the rule
- `metadata:policy security-ips alert;` - Tags the rule for policy classification

### Rule 2: Detecting Nmap TCP SYN Scan

Created a Snort rule that triggers an alert when a basic Nmap TCP SYN scan is performed. Focus on the initial SYN packet with no payload.

```shell
# Check for a community rule is available
sudo less /usr/local/etc/rules/snort3-community-rules/snort3-community.rules | grep 'syn'

# Edit the rule to fit the needs
alert tcp $EXTERNAL_NET any -> $HOME_NET any ( msg:"NMAP SYN Scan Detected"; flow:stateless; flags:S; dsize:0; detection_filter:track by_src, count 5, seconds 60; classtype:attempted-recon; sid:10000005; rev:1; metadata:policy security-ips alert; )
```

Breakdown of the alert rule:

- `alert` - It tells snort what to do when the rule matches, generate an alert when matched
- `tcp` - It only looks at TCP traffic
- `$EXTERNAL_NET` - Source is any IP outside the defined home network (defined in `snort.lua`)
- `any` - Any source port
- `->` -  A traffic direction indicator. It means from the source to destination
- `$HOME_NET` - Destination is the internal/protected network
- `any` - Any destination port
- `msg:"NMAP SYN Scan Detected";` - Text is shown in the alert when the rule is triggered
- `flow:stateless;` - Don't require an established session — inspect every packet independently (important for detecting scans, which never complete the handshake)
- `flags:S;` - Only match packets with only the SYN flag set (classic SYN scan signature)
- `dsize:0;` - Payload must be 0 bytes — SYN probe packets carry no data
- `detection_filter:track by_src, count 5, seconds 60;` - Only alert after the same source IP sends 5+ matching packets within 60 seconds — reduces false positives
- `classtype:attempted-recon;` - Categorizes the alert type using Snort's built-in classification system
- `sid:10000005;` -  Unique identifier for the rule
- `rev:1;` - Rule revision number 
- `metadata:policy security-ips alert;` - Tags the rule for policy classification

### Part 3: Testing the Rules

#### Testing Ping Rule

From a different machine on the same network, ran a simple ping command to the Snort machine's IP address.

Run Snort with the `local.rules` file and observed if the alert is triggered.

```shell
# Verify Snort loaded the rule and config with 0 warnings
sudo snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -T

# Observe if alert is triggered
sudo snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -i ens33 -A alert_fast -s 65535 -k none

# In another computer
ping 192.168.13.133 

# Check the log file for alerts
cat alert_fast.txt

09/08-19:56:52.229002 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 172.16.97.128 -> 172.16.97.130
10/21-13:54:21.900995 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 192.168.13.130 -> 192.168.13.133
```

Breakdown of the snort command:

- `sudo`: Run as root
- `snort`: Invokes the Snort 3 binary
- `-c /usr/local/etc/snort/snort.lua`: Points to the snort.lua configuration file.
- `-R /usr/local/etc/rules/local.rules`: Loads custom local rules on top of the config containing ICMP and NMAP TCP rule.
- `T`: Test mode — validates config and rules, then exits without capturing traffic. 

Breakdown of the snort command:

- `sudo`: Run as root
- `snort`: Invokes the Snort 3 binary
- `-c /usr/local/etc/snort/snort.lua`: Points to the snort.lua configuration file.
- `-R /usr/local/etc/rules/local.rules`: Loads custom local rules on top of the config containing ICMP and NMAP TCP rule.
- `-i ens33`: The interface to listen on.
- `-A alert_fast`: Alert output format — writes compact single-line alerts to /var/log/snort/alert_fast.txt.
- `-s 65535`: Set the snaplen (snapshot length), maximum bytes captured per packet. 65535 captures the full packet so Snort doesn't truncate and drop oversized packets.
- `-k none`: Ignore bad checksums, otherwise, snort will drop packets with bad checksums

#### Testing Nmap SYN Scan Rule

From a different machine on the same network, perform a basic Nmap SYN scan against the Snort machine's IP address.

Run Snort with the `local.rules` file and observed if the alert is triggered.

```shell
# Verify Snort loaded the rule and config with 0 warnings
sudo snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -T

sudo snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -i ens33 -A alert_fast -s 65535 -k none

# In another computer
nmap -sS 192.168.13.133 

# Check the log file for alerts
cat alert_fast.txt

10/21-14:06:48.502304 [**] [1:10000005:1] "NMAP SYN Scan Detected" [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.13.133:40952 -> 192.168.13.130:2401
```

## Reference Guides

- [How to Write Custom Snort Rules](https://www.zenarmor.com/docs/linux-tutorials/how-to-install-and-configure-snort-on-ubuntu-linux#how-to-write-custom-snort-rules)
- [Snort 3 Rule Writing Guide](https://docs.snort.org/start/running)
