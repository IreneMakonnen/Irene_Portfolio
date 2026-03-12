# Network Discovery & Analysis

[Metasploitable](https://sourceforge.net/projects/metasploitable/) will be used in this lab.

Metasploitable is an intentionally vulnerable Linux virtual machine. This VM can be used to conduct security training, test security tools, and practice common penetration testing techniques.

This lab involves setting up a vulnerable Metasploitable Linux virtual machine as a target in a virtualized environment. It involves using fundamental Linux network tools like `ping`, `traceroute`/`tracecert`, `nmap` for network discovery, `tcpdump` and `tshark` for network communication, and `netstat` or `ss` for understanding active network connections.

## Connectivity Testing

Used `ping` from the Linux host to test basic network connectivity to the running Metasploitable VM.

```shell
ifonfig

# ping [metasploitable2 eth0 IP address]
ping -c 5 192.168.139.132

PING 192.168.139.132 (192.168.139.132) 56(84) bytes of data.
64 bytes from 192.168.139.132: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 192.168.139.132: icmp_seq=2 ttl=64 time=2.72 ms
64 bytes from 192.168.139.132: icmp_seq=3 ttl=64 time=2.33 ms
64 bytes from 192.168.139.132: icmp_seq=4 ttl=64 time=2.53 ms
64 bytes from 192.168.139.132: icmp_seq=5 ttl=64 time=2.74 ms

--- 192.168.139.132 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30096ms
rtt min/avg/max/mdev = 1.175/2.886/12.538/1.813 ms
```

## Path Discovery

Used `traceroute` to map the network path from the Linux host to Metasploitable VM.

```shell
# traceroute [metasploitable2 eth0 IP address]
traceroute 192.168.139.132

traceroute to 192.168.139.132 (192.168.139.132), 30 hops max, 60 byte packets
 1  192.168.139.132 (192.168.139.132)  1.264 ms  1.271 ms  1.185 ms
```

## Network Discovery and Port Scanning

Used `nmap` to identify the numerous open ports and services running on the deliberately vulnerable Metasploitable Linux virtual machine.

Used `netdiscover` to find its IP address on the local network.

```shell
# To get IP range of host machine
ip a

# Identify the IP address of Metasploitable VM
sudo netdiscover -r 192.168.0.0/24

# A basic TCP port scan against Metasploitable
nmap 192.168.139.132

# A service version detection scan to see the versions of the services running on the open ports
nmap -sV 192.168.139.132
```

Based on the `nmap` scan, listed are the different services running on Metasploitable and the ports they are using.

| Service                    | Port |
| -------------------------- | ---- |
| ftp                        | 21   |
| ssh                        | 22   |
| telnet                     | 23   |
| smtp                       | 25   |
| domain                     | 53   |
| http                       | 80   |
| rpcbind                    | 111  |
| netbios-ssn                | 139  |
| microsoft-ds / netbios-ssn | 445  |
| exec                       | 512  |
| login?                     | 513  |
| shell                      | 514  |
| rmiregistry / java-rmi     | 1099 |
| ingreslock / bindshell     | 1524 |
| nfs                        | 2049 |
| ccproxy-ftp                | 2121 |
| mysql                      | 3306 |
| postgresql                 | 5432 |
| vnc                        | 5900 |
| X11                        | 6000 |
| irc                        | 6667 |
| ajp13                      | 8009 |
| http                       | 8180 |


```sh
# Scanning for common vulnerabilities using nmap's NSE scripts
nmap --script vuln 192.168.139.132

| smb-vuln-cve2009-3103:
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|
|     Disclosure date: 2009-09-08
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
```

- CVE-2009-3103 - Allows remote attackers to execute arbitrary code or cause a denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location, aka "SMBv2 Negotiation Vulnerability."

## Packet Capture and Analysis

Captured network traffic involving Metasploitable VM using `tcpdump` and analyze it with `tshark` to observe the communication associated with its vulnerable services.

```shell
# Identify the network interface on host machine used for virtual network (eth0 / wlan0)
ifconfig

#Login to the Metasploitable website - http://192.168.139.132

# Capture traffic to and from Metasploitable's IP address
sudo tcpdump -i eth0 host 192.168.139.132 -w msf_capture.pcap

# Analyze the captured traffic using tcpdump to look for specific protocols (e.g., HTTP, FTP, SSH)
tcpdump -r msf_capture.pcap -n tcp port 80 or tcp port 21 or tcp port 22

# Use tshark to examine the captured packets and filter by Metasploitable's IP or specific ports of interest (identified from the `nmap` scan)
tshark -r msf_capture.pcap -Y "ip.addr == 192.168.1.1 || tcp.port == 21/22/80"
```

## Basic Network Auditing 

Used `netstat` or `ss` directly on the Metasploitable Linux VM to observe the services it is listening on.

```shell
# SSH into Metasploitable2
ssh msfadmin@192.168.139.132

# Use netstat to list listening TCP and UDP ports
sudo netstat -tulnp | less

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:512             0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 0.0.0.0:513             0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 0.0.0.0:47937           0.0.0.0:*               LISTEN      5034/rpc.mountd 
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 0.0.0.0:43461           0.0.0.0:*               LISTEN      5259/rmiregistry
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      5222/jsvc       
tcp        0      0 0.0.0.0:6697            0.0.0.0:*               LISTEN      5277/unrealircd 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      4865/mysqld     
tcp        0      0 0.0.0.0:1099            0.0.0.0:*               LISTEN      5259/rmiregistry
tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN      5277/unrealircd 
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      5109/smbd       
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      5280/Xtightvnc  
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      4301/portmap    
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      5280/Xtightvnc  
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5240/apache2    
tcp        0      0 0.0.0.0:8787            0.0.0.0:*               LISTEN      5263/ruby       
tcp        0      0 0.0.0.0:8180            0.0.0.0:*               LISTEN      5222/jsvc       
tcp        0      0 0.0.0.0:1524            0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 0.0.0.0:35476           0.0.0.0:*               LISTEN      4317/rpc.statd  
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 192.168.139.132:53      0.0.0.0:*               LISTEN      4725/named      
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      4725/named      
tcp        0      0 0.0.0.0:23              0.0.0.0:*               LISTEN      5125/xinetd     
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      4945/postgres   
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      5100/master     
tcp        0      0 127.0.0.1:953           0.0.0.0:*               LISTEN      4725/named      
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      5109/smbd       
tcp        0      0 0.0.0.0:34557           0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::2121                 :::*                    LISTEN      5166/proftpd: (acce
tcp6       0      0 :::3632                 :::*                    LISTEN      4971/distccd    
tcp6       0      0 :::53                   :::*                    LISTEN      4725/named      
tcp6       0      0 :::22                   :::*                    LISTEN      4747/sshd       
tcp6       0      0 :::5432                 :::*                    LISTEN      4945/postgres   
tcp6       0      0 ::1:953                 :::*                    LISTEN      4725/named   

# Alternatively, use ss to list listening TCP and UDP ports
ss -tuln

Netid Recv-Q Send-Q                   Local Address:Port           Peer     Address:Port 
tcp   0      64                          *:512                     *:*      users:(("xinetd",5125,11))
tcp   0      64                          *:513                     *:*      users:(("xinetd",5125,10))
tcp   0      128                         *:47937                   *:*      users:(("rpc.mountd",5034,7))
tcp   0      64                          *:2049                    *:*     
tcp   0      64                          *:514                     *:*      users:(("xinetd",5125,9))
tcp   0      50                          *:43461                   *:*      users:(("rmiregistry",5259,8))
tcp   0      0                           *:8009                    *:*      users:(("jsvc",5222,64))
tcp   0      5                           *:6697                    *:*      users:(("unrealircd",5277,3))
tcp   0      5                          :::2121                   :::*      users:(("proftpd",5166,1))
tcp   0      50                          *:3306                    *:*      users:(("mysqld",4865,10))
tcp   0      50                          *:1099                    *:*      users:(("rmiregistry",5259,7))
tcp   0      5                           *:6667                    *:*      users:(("unrealircd",5277,2))
tcp   0      50                          *:139                     *:*      users:(("smbd",5109,22))
tcp   0      5                           *:5900                    *:*      users:(("Xtightvnc",5280,3))
tcp   0      128                         *:111                     *:*      users:(("portmap",4301,4))
tcp   0      128                         *:6000                    *:*      users:(("Xtightvnc",5280,0))
tcp   0      128                         *:80                      *:*      users:(("apache2",5240,3),("apache2",5241,3),("apache2",5243,3),("apache2",5246,3),("apache2",5248,3),("apache2",5252,3))
tcp   0      10                         :::3632                   :::*      users:(("distccd",4971,4),("distccd",4972,4),("distccd",5164,4),("distccd",5165,4))
tcp   0      5                           *:8787                    *:*      users:(("ruby",5263,3))
tcp   0      100                         *:8180                    *:*      users:(("jsvc",5222,49))
tcp   0      64                          *:1524                    *:*      users:(("xinetd",5125,12))
tcp   0      128                         *:35476                   *:*      users:(("rpc.statd",4317,8))
tcp   0      64                          *:21                      *:*      users:(("xinetd",5125,5))
tcp   0      3             192.168.139.132:53                      *:*      users:(("named",4725,25))
tcp   0      3                   127.0.0.1:53                      *:*      users:(("named",4725,23))
tcp   0      3                          :::53                     :::*      users:(("named",4725,21))
tcp   0      128                        :::22                     :::*      users:(("sshd",4747,3))
tcp   0      64                          *:23                      *:*      users:(("xinetd",5125,6))
tcp   0      128                         *:5432                    *:*      users:(("postgres",4945,6))
tcp   0      128                        :::5432                   :::*      users:(("postgres",4945,3))
tcp   0      100                         *:25                      *:*      users:(("master",5100,11))
tcp   0      128                       ::1:953                    :::*      users:(("named",4725,29))
tcp   0      128                 127.0.0.1:953                     *:*      users:(("named",4725,28))
tcp   0      50                          *:445                     *:*      users:(("smbd",5109,21))
tcp   0      64                          *:34557                   *:*     
```
