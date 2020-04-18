
> https://www.hackingdream.net/2019/02/what-is-stealthwatch-components-uses-advantages-netflow-analysis-ipfix.html

**Flow Sensor in StealthWatch**

It **is a Hardware Device/ Appliance/ Virtual Device which Creates flow data in environments in which NetFlow is not enabled**, Flow Sensors delivers performance analysis and deep packet inspection. All the flow data collected by the Flow Sensors is sent to the flow collector.

Note: The Environments in which NetFlow is enabled by default, need not implement Flow Sensors.

Flow Sensors Connects into existing Infrastructure via one of the following
          1) Switch port Analyzer (SPAN)
          2) Mirror Port
          3) Ethernet Test Access Port (TAP)


**Flow Collector in StealthWatch:**

- Flow collector aggregates flow data from multiple networks or network components
- collects and analyses data for further retrieval and analysis
- Flow Collector of StealthWatch send and analyzes data sent from the SMC
- Send an alarm if any unusual activity occurred or detected.
- Flow collector can either be a Virtual Applicance/ Hardware Device.

**UDP Director/ Flow Replicator:**

- Simplifies the management of UDP data streams from NetFlow, sFlow, Syslog, SNMP Traffic
- Forwards data from multiple network and security locations in a single data stream to network devices including the flow collector;
- Aggregates and provides a single destination for UDP data and allows distribution of it across the organization
- High speed high performance appliance that simplifies the collection of network and security across your network
- reduces point of failure on your network
- provides a single destination for all UDP formats on network including NetFlow, SNMP, Syslog
- Reduces network congestion for optimum network performance.

**SMC Config**

![](assets/markdown-img-paste-20191021051456610.png)

**FC Config**
![](assets/markdown-img-paste-20191021051015566.png)

![](assets/markdown-img-paste-2019102105173752.png)

> Netflow Configuration Template https://configurenetflow.info/

> Stealthwatch Condifuration Guide 7.1.1 https://www.cisco.com/c/dam/en/us/td/docs/security/stealthwatch/system_installation_configuration/SW_7_1_1_Installation_and_Configuration_Guide_DV_1_0.pdf


***Very interesting Packet Capture***

Notes below from https://www.netresec.com/?page=PcapFiles

Publicly available PCAP files
This is a list of public packet capture repositories, which are freely available on the Internet.
Most of the sites listed below share Full Packet Capture (FPC) files, but some do unfortunately only have truncated frames.

Cyber Defence Exercises (CDX)
This category includes network traffic from exercises and competitions, such as Cyber Defense Exercises (CDX) and red-team/blue-team competitions.

MACCDC - Pcaps from National CyberWatch Mid-Atlantic Collegiate Cyber Defense Competition
https://www.netresec.com/?page=MACCDC

ISTS - Pcaps from the Information Security Talent Search
https://www.netresec.com/?page=ISTS

WRCCDC - Pcaps from the Western Regional Collegiate Cyber Defense Competition (over 1TB of PCAPs)
https://archive.wrccdc.org/pcaps/

Captures from the "2009 Inter-Service Academy Cyber Defense Competition" served by Information Technology Operations Center (ITOC), United States Military Academy
http://www.westpoint.edu/crc/SitePages/DataSets.aspx

Malware Traffic
Captured malware traffic from honeypots, sandboxes or real world intrusions.

Contagio Malware Dump: Collection of PCAP files categorized as APT, Crime or Metasplot
http://contagiodump.blogspot.com/2013/04/collection-of-pcap-files-from-malware.html
(the PCAP archive is hosted on DropBox and MediaFire)
WARNING: The password protected zip files contain real malware
Also see Contagio's PCAP files per case:

Trojan.Tbot http://contagiodump.blogspot.com/2012/12/dec-2012-skynet-tor-botnet-trojantbot.html
ZeroAccess Trojan http://contagiodump.blogspot.com/2012/10/blackhole-2-exploit-kit-files-partial.html
CVE-2012-4681 http://contagiodump.blogspot.com/2012/09/cve-2012-4681-samples-original-apt-and.html
Trojan Taidoor http://contagiodump.blogspot.com/2011/11/nov-3-cve-2011-0611-1104statmentpdf.html
Poison Ivy CnC http://contagiodump.blogspot.com/2011/07/message-targeting-experts-on-japan.html
Malware analysis blog that shares malware as well as PCAP files
http://malware-traffic-analysis.net/

GTISK PANDA Malrec - PCAP files from malware samples run in PANDA, created by @moyix and GTISK
http://panda.gtisc.gatech.edu/malrec/

Stratosphere IPS - PCAP and Argus datasets with malware traffic, created by Sebastian Garcia (@eldracote) at the ATG group of the Czech Technical University
https://www.stratosphereips.org/datasets-overview/

VM execution of info-stealer malware. Created by the Services, Cybersecurity and Safety research group at University of Twente.
http://scs.ewi.utwente.nl/downloads/

Regin malware PCAP files, created by @moyix (see his blog post)
http://laredo-13.mit.edu/~brendan/regin/pcap/

Ponmocup malware/trojan (a.k.a. Milicenso) PCAP by Tom Ueltschi a.k.a. @c_APT_ure
https://download.netresec.com/pcap/ponmocup/vm-2.pcap
Also see original source (password protected zip) and analysis writeup (text)

PCAP file with PowerShell Empire (TCP 8081) and SSL wrapped C2 (TCP 445) traffic from CERT.SE's technical writeup of the major fraud and hacking criminal case "B 8322-16".
https://drive.google.com/open?id=0B7pTM0QU5apSdnF0Znp1Tko0ams

Free malware analysis sandbox. Malware samples can be uploaded or searched, PCAP files from sandbox execution can be downloaded.
https://hybrid-analysis.com/

Online client honeypot for sharing, browsing and analyzing web-based malware. PCAP download available for analyzed sites.
http://threatglass.com/

Shadowbrokers PCAPs by Eric Conrad, including ETERNALBLUE and ETERNALROMANCE.
https://www.dropbox.com/sh/kk24ewnqi9qjdvt/AACj7AHJrDHQeyJTuo1oBqeQa

Network Forensics
Network forensics training, challenges and contests.

Hands-on Network Forensics - Training PCAP dataset from FIRST 2015
https://www.first.org/conference/2015/program#phands-on-network-forensics
Files mirrored by Netresec:

Slides/Cases (PDF)
SecurityOnion VM (5.8 GB)
VirtualBox VM with PCAP files. VM login credentials are: user/password
Capture files (4.4 GB)

ENISA's Network Forensics training
https://www.enisa.europa.eu/topics/trainings-for-cybersecurity-specialists/online-training-material/technical-operational#network_forensics

Document for teachers (PDF)
Document for students (PDF)
Virtual Machine (OVA)

Forensic Challenge 14 – “Weird Python“ (The Honeynet ProjectThe Honeynet Project)
http://honeynet.org/node/1220

Network Foreniscs Puzzle Contest (by Lake Missoula Group, LLC)
http://forensicscontest.com/puzzles

DFRWS 2008 Challenge
http://old.dfrws.org/2008/challenge/submission.shtml

DFRWS 2009 Challenge
http://old.dfrws.org/2009/challenge/submission.shtml

DFIR MONTEREY 2015 Network Forensics Challenge (by Phil Hagen of SANS)
http://digital-forensics.sans.org/blog/2014/12/05/dfir-monterey-2015-network-forensics-challenge-released

SCADA/ICS Network Captures
4SICS ICS Lab PCAP files - 360 MB of PCAP files from the ICS village at 4SICS
https://www.netresec.com/?page=PCAP4SICS

DigitalBond S4x15 ICS Village CTF PCAPs
https://www.netresec.com/?page=DigitalBond_S4

Compilation of ICS PCAP files indexed by protocol (by Jason Smith)
https://github.com/automayt/ICS-pcap

DEF CON 23 ICS Village
https://media.defcon.org/DEF CON 23/DEF CON 23 villages/DEF CON 23 ics village/DEF CON 23 ICS Village packet captures.rar (requires RAR v5)

TRITON execition of the TriStation protocol by Nozomi Networks
https://github.com/NozomiNetworks/tricotools/blob/master/malware_exec.pcap

TriStation traffic
https://packettotal.com/app/analysis?id=0e55d9467f138d148c9635617bc8fd83

Chinese ICS CTF with Modbus/TCP and Siemens S7comm traffic (CTF WP – 工控业务流量分析)
http://www.icsmaster.org/archives/ics/741

ICS Cybersecurity PCAP repository by Univ. of Coimbra CyberSec team
https://github.com/tjcruz-dei/ICS_PCAPS

Capture the Flag Competitions (CTF)
PCAP files from capture-the-flag (CTF) competitions and challenges.

Note: Sniffing CTF's is known as "capture-the-capture-the-flag" or CCTF.
DEFCON Capture the Flag Contest traces (from DEF CON 8, 10 and 11)
http://cctf.shmoo.com/

DEFCON 17 Capture the Flag Contest traces
http://ddtek.biz/dc17.html

DEFCON CTF PCAPs from DEF CON 17 to 24 (look for the big RAR files inside the ctf directories)
https://media.defcon.org/

DEFCON CTF 2018 PCAP files
https://www.oooverflow.io/dc-ctf-2018-finals/

CSAW CTF 2011 pcap files
http://captf.com/2011/csaw-quals/networking/
http://repo.shell-storm.org/CTF/CSAW-2011/Networking/

Pcap files from UCSB International Capture The Flag, also known as the iCTF (by Giovanni Vigna)
https://ictf.cs.ucsb.edu/pages/archive.html

HackEire CTF Challenge pcaps from IRISSCON (by HackEire)
https://github.com/markofu/hackeire/

No cON Name 2014 CTF Finals, "Vodka" challenge
https://github.com/MarioVilas/write-ups/raw/master/ncn-ctf-2014/Vodka/vodka (bzip2 compressed PCAP-NG file)

PhreakNIC CTF from 2016 (by _NSAKEY). Contains traffic to/from the target, the NetKoTH scoring server and the IRC server.
https://drive.google.com/drive/folders/0B9TXiR9NkjmpOHNMRTl6VVA2RnM

Packet Injection Attacks / Man-on-the-Side Attacks
PCAP files from research by Gabi Nakibly et al. in Website-Targeted False Content Injection by Network Operators
http://www.cs.technion.ac.il/~gnakibly/TCPInjections/samples.zip

Packet injection against id1.cn, released by Fox-IT at BroCon 2015
https://github.com/fox-it/quantuminsert/blob/master/presentations/brocon2015/pcaps/id1.cn-inject.pcap

Packet injection against www.02995.com, doing a redirect to www.hao123.com (read more)
https://www.netresec.com/files/hao123-com_packet-injection.pcap

Packet injection against id1.cn, doing a redirect to batit.aliyun.com (read more)
https://www.netresec.com/files/id1-cn_packet-injection.pcap

Pcap files for testing Honeybadger TCP injection attack detection
https://github.com/david415/honeybadger-pcap-files

Man-in-the-Middle (MitM) attacks (a.k.a. "in-path attacks") in Turkey and Egypt discovered by Bill Marczak (read more).
https://github.com/citizenlab/badtraffic/tree/master/pcaps

Uncategorized PCAP Repositories
Wireshark Sample Capures
http://wiki.wireshark.org/SampleCaptures
http://wiki.wireshark.org/Development/PcapNg#Example_pcapng_Capture_File

SharkFest'15 Packet Challenge
https://sharkfest.wireshark.org/assets/presentations15/packetchallenge.zip (via SharkFest)

Packet analysis challenge by Johannes Weber
https://blog.webernetz.net/2017/03/29/wireshark-layer-2-3-pcap-analysis-w-challenges-ccnp-switch/
Additional PCAP files from Johannes can be found here: https://blog.webernetz.net/tag/pcap/

TcpReplay Sample Captures
http://tcpreplay.appneta.com/wiki/captures.html

Applied Communication Sciences' MILCOM 2016 datasets
https://www.netresec.com/?page=ACS_MILCOM_2016

Australian Defence Force Academy (ADFA) UNSW-NB15 data set (100 GB)
https://cloudstor.aarnet.edu.au/plus/index.php/s/2DhnLGDdEECo4ys?path=%2FUNSW-NB15%20-%20pcap%20files

DARPA Intrusion Detection Data Sets from 1998 and 1999
http://www.ll.mit.edu/mission/communications/cyber/CSTcorpora/ideval/data/

OpenPacket.org Capture Repository (maintained by JJ Cummings created by Richard Bejtlich)
https://www.openpacket.org/capture/list

Tim's packet Zoo
http://uluru.ee.unsw.edu.au/~tim/zoo/

Over 4 GB of network forensic training data from DEEP (Digital Evaluation and Exploitation Department of Computer Science, Naval Postgraduate School). Case details can be found at Jesse Kornblum's blog.
http://digitalcorpora.org/corpora/network-packet-dumps (HTTP)
http://terasaur.org/item/downloads/computer-forensics-2009-m57-scenario/187 (Torrent)

PacketLife.net Packet Captures (Jeremy Stretch)
http://packetlife.net/captures/
http://packetlife.net/captures/leech/

MOME database
http://www.ist-mome.org/database/MeasurementData/?cmd=databrowse

EvilFingers PCAPs
https://www.evilfingers.com/repository/pcaps.php

Mixed PCAP file repo with a great deal of BACnet traffic (by Steve Karg)
http://kargs.net/captures/

Wireshark Network Analysis Study Guide (Laura Chappell)
http://wiresharkbook.com/studyguide.html (see "Book Supplements" or use this direct link to the 1.5 TB pcap file set)

Wireshark 101 Essential Skills for Network Analysis (Laura Chappell)
http://wiresharkbook.com/wireshark101.html (see "Book Supplements" or use this direct linkt to the 330 MB zip file)

Laura's Lab Kit v.9 ISO image (old)
http://cdn.novell.com/cached/video/bs_08/LLK9.iso

Freely available packet captures collected by Chris Sanders
http://chrissanders.org/packet-captures/

Sample capture files from: "Practical Packet Analysis - Using Wireshark to Solve Real-World Network Problems" by Chris Sanders
http://www.nostarch.com/download/ppa-capture-files.zip

Megalodon Challenge by Jasper Bongertz - "a real world network analysis problem, with all its confusion, drawbacks and uncertainties" (3.8 GB sanitized PCAP-NG files)
Blog post: https://blog.packet-foo.com/2015/07/the-megalodon-challenge/
Direct link: http://www.packet-foo.com/megalodon2015/MegalodonChallenge.7z

Pcaps and logs generated in @elcabezzonn's lab environment. Spans from malware, to normal traffic, to pentester tools
https://github.com/elcabezzonn/Pcaps

Anonymous FTP connections to public FTP servers at the Lawrence Berkeley National Laboratory
http://www-nrg.ee.lbl.gov/anonymized-traces.html

Pcapr (Mu Dynamics) - A capture repository with pcap files of various traffic types
http://www.pcapr.net/

Understand project Downloads - Lots of different capture file formats (pcap, pcapng/ntar, pcangpklg and more...)
http://code.google.com/p/understand/downloads/list

I Smell Packets (website)
https://docs.google.com/leaf?id=0Bw6BFSu9NExVMjBjZDRkMTgtMmMyZi00M2ZlLWI2NzgtODM5NTZkM2U4OWQ1

ISCX 2012 Dataset. Over 80 GB of pcap data available for researchers (created by Ali Shiravi, Hadi Shiravi, and Mahbod Tavallaee from University of New Brunswick)
http://www.unb.ca/research/iscx/dataset/index.html

Research PCAP datasets from FOI's Information Warfare Lab (FOI is The Swedish Defence Research Agency)
ftp://download.iwlab.foi.se/dataset/smia2011/Network_traffic/ (SMIA 2011, FTP server)
https://download.netresec.com/pcap/smia-2011/ (SMIA 2011, web mirror)
ftp://download.iwlab.foi.se/dataset/smia2012/network_traffic/pcap/ (SMIA 2012, FTP server)
https://download.netresec.com/pcap/smia-2012/ (SMIA 2012, web mirror)

Packet collections in PCAP-NG format by Teguh P. Alko
http://stuff.rop.io/packets/

Internet Traffic Archive (Berkeley Lab) - mostly tcpdump ASCII output
http://ita.ee.lbl.gov/html/traces.html

WITS: Waikato Internet Traffic Storage (traces in ERF format with headers plus 4 bytes of application data)
http://wand.net.nz/wits/
The FTP site uses rate limiting for IPv4 connections, but no ratelimit for IPv6 connections.

Bro IDS trace files (no application layer data)
ftp://ftp.bro-ids.org/enterprise-traces/hdr-traces05/

SimpleWeb captures (mainly packet headers)
http://www.simpleweb.org/wiki/Traces

Wireless LAN Traces from ACM SIGCOMM'01 (no application layer data)
http://sysnet.ucsd.edu/pawn/sigcomm-trace/

Wireshark Fuzzed Protocol Capures (only fuzzed packets)
ftp://wireshark.org/automated/captures/

Single PCAP files
Honeynet.org's Scan of the Month PCAPs
http://www.honeynet.org/scans/scan21/
http://www.honeynet.org/scans/scan22/
http://www.honeynet.org/scans/scan23/
http://www.honeynet.org/scans/scan27/
http://www.honeynet.org/scans/scan28/

CrypMic ransomware infection (read the blog post)
http://www.broadanalysis.com/wp-content/uploads/2016/08/2016-08-16-Neutrino-EK-pcap.zip

MDSec, Packets from a GSM 2.5G environment showing uplink/downlink, two MS devices, SIM APDU information.
https://github.com/HackerFantastic/Public/blob/master/misc/44CON-gsm-uplink-downlink-sim-example.pcap?raw=true

SDN OpenFlow pcap-ng file by SDN/IPv6 expert Jeff Carrell.
https://download.netresec.com/pcap/JeffCarrell/SDN-OpenFlow-Analysis-2.pcapng

Demo of JexBoss (Jboss EXploitation Tool) "JBoss exploits - View from a Victim" by Andre M. DiMino
http://www.deependresearch.org/2016/04/jboss-exploits-view-from-victim.html

Raul Siles, “Pcap files containing a roaming VoIP session”
http://www.raulsiles.com/downloads/VoIP_roaming_session.zip

Russ McRee, W32/Sdbot infected machine
http://holisticinfosec.org/toolsmith/files/nov2k6/toolsmith.pcap

Joke Snelders, WiFi traffic encrypted with WPA pre-shared key (passphrase "subnet16121930", SSID "dd-wrt2").
http://www.lovemytool.com/files/test.pcap
Read Joke's "Wireshark and TShark: Decrypt Sample Capture File" blog post for decryption instructions.

hack.lu 2009 Information Security Visualization Contest (honeypot traffic, mostly SSH and HTTP)
http://2009.hack.lu/index.php/InfoVisContest

Barracuda Labs on the PHP.net Compromise [blog post]
PCAP: http://barracudalabs.com/downloads/5f810408ddbbd6d349b4be4766f41a37.pcap

Barracuda Labs on the Cracked.com Malware [blog post]
PCAP: https://copy.com/UoJTysFFh6ef

Online PCAP Services
Convert PcapNG files to PCAP format
http://pcapng.com/

CloudShark - Wireshark-like analysis in your browser
http://www.cloudshark.org/

NetworkTotal - Runs uploaded PCAP through Suricata IDS
https://www.networktotal.com/

PacketTotal - Runs uploaded PCAP through Bro and Suricata IDS and makes it publicly avaliable
https://packettotal.com/

ANY.RUN - Interactive malware analysis service that generates PCAP files.
https://app.any.run/

```
