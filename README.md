# My-first-CTF-report
A complete structural of Vulnerability assessment and pentesting
INTRODUCTION
This report analyze CTF challenge involving a Packet capture (pcap) file that
document multi-stage attack chain. The attacker established ftp connection
and then exfilterate an encryption configuration file and then exfilterate the
http post request. The objective follows attack chain decryption the request
and recover the flag
SUMMARY
This challenge demonstrate real-world attack technique includes:
1. Anonymous FTP login
2. Encryption via AES
3. Multi-level encoding
4. Post-exploitation
SCOPE
The scope that the packet capture (pcap) file that send for this challenge.
In this file contains 50 packet captured over
1.FTP request
2. HTTP request
The ip address connects between
Attacker : x.x.x.x
Victim : x.x.x.x

Detailed-Findings & POC concept:
First let breakdown the protocol which is in the pcap file with the help of
wireshark. Wireshark is a packet analyser tool in. It is used for computer
network analysis and troubleshooting,
ICMP – router solicitation 1-packet
FTP(control) - Anonymous login on port 21
HTTP - POST request on port 80
1.Attack Walkthrough:
FTP Recon and File Theft
The attacker connects the ftp server with anonymous login creds:
Username : anonymous
Password : -wget@
After successful authentication, the attacker downloads agent_config.json
 via the RETR command. The file contains:
 {
 "status": "staged",
 "aes_key": "SuperSecretKey1234567890123456",
 "iv": "InitVector123456"
 }
This configuration file provides the encryption key and iv vector we need
to decrypt to data

1.2 Encryption and Data exfilteration
 The HTTP POSTs encryption data to the server endpoint / data using curl
with the content type
POST body :
payload=JRY5PSh8ZFDuulhh8+U8i1NKiRggpdLIj/oi8Q8X+5I/gqW+MOoDse6
UgTEcQK5cFWuKwa4p8eFG1ZdWfYhlcxyBEvlhZa6pox+miIs8HqQ/svhpT3jU
ujpC114Vh47ytKES12HfwyCuQu75aSj8aJWIVISvOxFI+DiYXEzu22KwW//P3g
r13fuRKBplXlHk
The server (Werkzeug 3.1.5 / Python 3.13.12) responds with "200 OK"and
message "Data received"

1.3 Decryption Process
In this part I used the tool named Cyberchef a web app for encryption,
decryption, compression and data analysis. Lets explained this into layered
Level 1 - Base64 decode:
The HTTP payload parameter is Base64-decoded to obtain 144 bytes of AES256-CBC encrypted ciphertext
Level 2 – AES-256-CBC:
Here we found the KEY and IV from the previous FTP packet from the
wireshark
KEY = SuperSecretKey123456789012345678
IV = InitVector123456
MODE = CBC 
Here the Decrypted output:
 {"instruction": "update_config", "encryption_type": "xor",
 "xor_key": 66,
 "payload_bytes":
"2730223b1f2c50272f285532503b2a57303b5654565219"}
LEVEL 3 Hex decode:
The payload byte data
“2730223b1f2c50272f285532503b2a57303b5654565219”value is hex -
decoded
From the result shows the output string is randomized but from the previous
we can get the Xor key = 66 so the next level is xor decrypt
Level 4 Xor:
The json_file indicates xor key is 66 but testing reveals the xor key 100 in
decimal and hex in 64
By changing the key value into 64 we decrypt the hidden flag in the string
Final Flag: CTF_{H4CKL1V4_N3T_2026}

Conclusion:
This CTF challenge gives the real world multi-stage attack:
1. Anonymous login creds
2. Aes-256-cbc encrypt
3. Multi-stage decryption
4. Data exfilteration
Remediation Summary:
The vulnerabilities demonstrated in the challenge can be mitigated
1. FTP Security
2. Encryption best pratices
3. Logging and monitoring
4. Network security
5. Application security
1.FTP security
Disable anonymous login FTP access
Implement IP allowlisting for FTP access
2. Encryption best pratices
Store Encryption keys in HSM module
Rotate Encryption keys on a regular schedule
3. Logging and monitoring
Monitor for successful anonymous FTP logins and file downloads
Enable detailed audit logging for file access and data transfers
4.Network security
Isolate FTP and Web server and implement network segmentation
Deploy NIDS to detect anomalous data exfiltration patterns
Monitor for large POST requests containing Base64-encoded data
Use TLS encryption for all data in transit
5.Application Security
Implement Proper authentication and authorization for all members
Use web application firewall (WAF)
Conduct regular assessment and penetration testing
