# Packet Capture Analysis

This lab considers a scenario where an attacker has planted a keylogger on one of the systems in the network.

Packet capture file used: [Noobs Keylogger.pcap](https://github.com/nipunjaswal/networkforensics/blob/master/Ch1/Noobs%20KeyLogger/Noobs%20Keylogger.pcap)

## ​Find the infected system

![Infested system](<attachments/Packet Capture Analaysis/Infected system.png>)

The IP address of the infected system: `192.168.76.131`

## ​Trace the data to the server

Used various filters such as `http` (for web traffic), `ftp` (for file transfers) and `ftp-data`(for the actual file contents transferred over FTP).

![HTTP traffic analysis](<attachments/Packet Capture Analaysis/http traffic analysis.png>)

![FTP traffic analysis](<attachments/Packet Capture Analaysis/ftp traffic analysis.png>)

![FTP-data traffic analysis](<attachments/Packet Capture Analaysis/ftp-data traffic analysis.png>)

This reveals the server IP is `140.82.59.185`.

## ​Find the frequency of the data that is being sent

![Frequency of data being sent](<attachments/Packet Capture Analaysis/Data frequency of traffic.png>)

A packet is sent every 63 seconds.

## Find what other information is carried besides the keystrokes

**On Wireshark**
![Exported Objects](<attachments/Packet Capture Analaysis/Exported objects.png>)

![Active window](<attachments/Packet Capture Analaysis/Active window.png>)

Apart from keylogging, the attacker also fetched details such as the active window.

**On Network Miner**
![Network Miner - Files](<attachments/Packet Capture Analaysis/Network Miner.png>)
The files tab shows plenty of files.

## ​Try to uncover the attacker

```shell
whois 140.82.59.185              

NetRange:       140.82.0.0 - 140.82.63.255
CIDR:           140.82.0.0/18
NetName:        CONSTANT
NetHandle:      NET-140-82-0-0-1
Parent:         NET140 (NET-140-0-0-0-0)
NetType:        Direct Allocation
OriginAS:
Organization:   The Constant Company, LLC (CHOOP-1)
RegDate:        2018-03-15
Updated:        2022-09-20
Comment:        Geofeed https://geofeed.constant.com/
Ref:            https://rdap.arin.net/registry/ip/140.82.0.0
OrgName:        The Constant Company, LLC
OrgId:          CHOOP-1
Address:        319 Clematis St. Suite 900
City:           West Palm Beach
StateProv:      FL
PostalCode:     33401
Country:        US
RegDate:        2006-10-03
Updated:        2022-12-21
Comment:        http://www.constant.com/
Ref:            https://rdap.arin.net/registry/entity/CHOOP-1


OrgNOCHandle: NETWO1159-ARIN
OrgNOCName:   Network Operations
OrgNOCPhone:  +1-973-849-0500 
OrgNOCEmail:  network@constant.com
OrgNOCRef:    https://rdap.arin.net/registry/entity/NETWO1159-ARIN

OrgTechHandle: NETWO1159-ARIN
OrgTechName:   Network Operations
OrgTechPhone:  +1-973-849-0500 
OrgTechEmail:  network@constant.com
OrgTechRef:    https://rdap.arin.net/registry/entity/NETWO1159-ARIN

# end
 
# start

NetRange:       140.82.58.0 - 140.82.59.255
CIDR:           140.82.58.0/23
NetName:        NET-140-82-58-0-23
NetHandle:      NET-140-82-58-0-1
Parent:         CONSTANT (NET-140-82-0-0-1)
NetType:        Reassigned
OriginAS:       
Organization:   Vultr Holdings, LLC (VHL-31)
RegDate:        2018-03-22
Updated:        2018-03-22
Ref:            https://rdap.arin.net/registry/ip/140.82.58.0


OrgName:        Vultr Holdings, LLC
OrgId:          VHL-31
Address:        2031 BE, Haarlem
City:           Amsterdam
StateProv:      NOORD-HOLLAND
PostalCode:     2031
Country:        NL
RegDate:        2015-03-05
Updated:        2024-04-04
Ref:            https://rdap.arin.net/registry/entity/VHL-31

OrgTechHandle: LYNCH267-ARIN
OrgTechName:   Lynch, Tomas 
OrgTechPhone:  +1-973-849-0500 
OrgTechEmail:  tlynch@vultr.com
OrgTechRef:    https://rdap.arin.net/registry/entity/LYNCH267-ARIN

# end
```

```sh
dig -x 140.82.59.185 +short

# Output
140.82.59.185.vultrusercontent.com.
```

## ​Extract and reconstruct the files that have been sent to the attacker

![Exported Object](<attachments/Packet Capture Analaysis/Exported objects.png>)

**Keys_2018-11-28_16-04-42.html** file:
![Keys file](<attachments/Packet Capture Analaysis/Keys file.png>)

Victim used Windows File Explorer to browse in the Pictures folder.

We can see that the data being transmitted contains the word Ardamax, which is the name of a common piece of keylogger software that records keystrokes from the system it has infected and sends it back to the attacker.

**Web_2018-11-29_11-28-13.html** file:
![Web file](<attachments/Packet Capture Analaysis/Active window.png>)

The attacker fetched details such as the active window.

The victim, `nipun`, used Microsoft Edge to visit Gmail

**On Network Miner**
![Credentials shown in Network Miner](<attachments/Packet Capture Analaysis/Credentials on Network Miner.png>)
