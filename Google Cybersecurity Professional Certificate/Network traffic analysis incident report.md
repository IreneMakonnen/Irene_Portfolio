# Analyzing network layer communication
## Scenario
You are a cybersecurity analyst working at a company that specializes in providing IT services for clients. Several customers of clients reported that they were not able to access the client company website www.yummyrecipesforme.com, and saw the error “destination port unreachable” after waiting for the page to load. 

You are tasked with analyzing the situation and determining which network protocol was affected during this incident. To start, you attempt to visit the website and you also receive the error “destination port unreachable.” To troubleshoot the issue, you load your network analyzer tool, tcpdump, and attempt to load the webpage again. To load the webpage, your browser sends a query to a DNS server via the UDP protocol to retrieve the IP address for the website's domain name; this is part of the DNS protocol. Your browser then uses this IP address as the destination IP for sending an HTTPS request to the web server to display the webpage  The analyzer shows that when you send UDP packets to the DNS server, you receive ICMP packets containing the error message: “udp port 53 unreachable.” 



In the tcpdump log, you find the following information:

1. The first two lines of the log file show the initial outgoing request from your computer to the DNS server requesting the IP address of yummyrecipesforme.com. This request is sent in a UDP packet.

2. The third and fourth lines of the log show the response to your UDP packet. In this case, the ICMP 203.0.113.2 line is the start of the error message indicating that the UDP packet was undeliverable to port 53 of the DNS server.

3. In front of each request and response, you find timestamps that indicate when the incident happened. In the log, this is the first sequence of numbers displayed: 13:24:32.192571. This means the time is 1:24 p.m., 32.192571 seconds.

4. After the timestamps, you will find the source and destination IP addresses. In the first line, where the UDP packet travels from your browser to the DNS server, this information is displayed as: 192.51.100.15 > 203.0.113.2.domain. The IP address to the left of the greater than (>) symbol is the source address, which in this example is your computer’s IP address. The IP address to the right of the greater than (>) symbol is the destination IP address. In this case, it is the IP address for the DNS server: 203.0.113.2.domain. For the ICMP error response, the source address is 203.0.113.2 and the destination is your computers IP address 192.51.100.15.

5. After the source and destination IP addresses, there can be a number of additional details like the protocol, port number of the source, and flags. In the first line of the error log, the query identification number appears as: 35084. The plus sign after the query identification number indicates there are flags associated with the UDP message. The "A?" indicates a flag associated with the DNS request for an A record, where an A record maps a domain name to an IP address. The third line displays the protocol of the response message to the browser: "ICMP," which is followed by an ICMP error message.

6. The error message, "udp port 53 unreachable" is mentioned in the last line. Port 53 is a port for DNS service. The word "unreachable" in the message indicates the UDP message requesting an IP address for the domain "www.yummyrecipesforme.com" did not go through to the DNS server because no service was listening on the receiving DNS port.

7. The remaining lines in the log indicate that ICMP packets were sent two more times, but the same delivery error was received both times.  

### Cybersecurity Incident Report: Network Traffic Analysis
#### Part 1: Provide a summary of the problem found in the DNS and ICMP  traffic log.  
The UDP protocol reveals that: the server was unable to acknowledge the reception of the data packet.  

This is based on the results of the network analysis, which show that the ICMP echo reply returned the error message: udp port 53 unreachable   

The port noted in the error message is used for: DNS  

The most likely issue is: the DNS server is offline or overwhelmed with malicious attacks.

#### Part 2: Explain your analysis of the data and provide at least one cause of the incident. 
Time incident occurred: 1:24 PM  

Explain how the IT team became aware of the incident: Upon complaints of several customers of clients unable to access the website www.yummyrecipesforme.com, and saw the error “destination port unreachable” after waiting for the page to load.  

Explain the actions taken by the IT department to investigate the incident: First tried to access the website and received a similar error message “destination port unreachable”. Did a troubleshoot by loading tcpdump, a network analyzer tool, and attempted to load the webpage again. The analyzer showed that when you send UDP packets to the DNS server, you receive ICMP packets containing the error message: “udp port 53 unreachable”.  

Note key findings of the IT department's investigation (i.e., details related to the port affected, DNS server, etc.): The plus sign after the query identification number indicates there are flags associated with the UDP message. The "A?" indicates a flag associated with the DNS request for an A record, where an A record maps a domain name to an IP address. The third line displays the protocol of the response message to the browser: "ICMP," which is followed by an ICMP error message. 

Next steps in troubleshooting and resolving the issue: Restart the DNS server and cancel requests overwhelming the server.  

Note a likely cause of the incident: DoS attack.
