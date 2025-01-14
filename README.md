![banner](share/ettercap.png)


<div align="center">

# A suite for man in the middle attacks
**Copyright 2001-Current The Ettercap Dev Team**

</div>

## INTRODUCTION

Ettercap is a comprehensive suite for man in the middle attacks.
It features sniffing of live connections, content filtering on the fly and many other interesting tricks.
It supports active and passive dissection of many protocols and includes many features for network and host analysis.

> [!NOTE]
> Aoccdrnig to rscheearch at an Elingsh uinervtisy, it deosn't mttaer in waht
oredr the ltteers in a wrod are, the olny iprmoetnt tihng is taht the frist
and lsat ltteer are in the rghit pclae. The rset can be a toatl mses  and
you can sitll raed it wouthit a porbelm. Tihs is bcuseae we do not raed
ervey lteter by it slef but the wrod as a wlohe and the biran fguiers it
out aynawy.
>
> **... so please excuse us for every typo in the documentation, man pages or 
code, btw fixes and patches are welcome.**

## REQUIRED PROGRAMS

- C compiler
- flex (or other lex-compatible parser generator) for *.l files
- bison (or other yacc-compatible parser generator) for *.y files
- cmake (build tool)

## REQUIRED LIBRARIES

### MANDATORY:
- libpcap >= 0.8.1
- libnet  >= 1.1.2.1 (>= 1.1.5 for IPv6 support)
- openssl >= 0.9.7
- libpthread
- zlib
- libmaxminddb (successor of libgeoip)
- CMake 2.8
- Curl >= 7.26.0 to build SSLStrip plugin

> [!TIP]
> If you don't want to enable SSLStrip plugin you have to disable it.
    (more information about disabling a plugin in the [README.GIT](README.GIT) file)

### OPTIONAL:
- To avoid use of our internal strlcat and strlcpy implementation: `libbsd`
- To enable PDF documentation generation (enable via ENABLE_PDF_DOCS=On): `groff`
- To enable plugins: `libltdl` (part of libtool)
- To have perl regexp in the filters: `libpcre`
- For the cursed GUI: `ncurses` >= 5.3
- For the GTK+ GUI:
  - `Glib`      >= 2.2.2
  - `Gtk+3`    >= 3.12.0 (recommended >= 3.22.0)
  - `Atk`       >= 1.2.4
  - `Pango`     >= 1.2.3

> [!TIP]
> If you are running on debian, or any debian based distro you can install
the required dependencies by running:
>
> ```
> apt-get install build-essential debhelper bison check cmake flex groff libbsd-dev \
>      libcurl4-openssl-dev libmaxminddb-dev libgtk-3-dev libltdl-dev libluajit-5.1-dev \
>      libncurses5-dev libnet1-dev libpcap-dev libpcre2-dev libssl-dev
> ```

## LICENSE

see [LICENSE](LICENSE) file for details...

## AUTHORS
- Alberto Ornaghi (ALoR) <alor@users.sourceforge.net>
- Marco Valleri (NaGA) <naga@antifork.org>
- Emilio Escobar (exfil) <eescobar@gmail.com>
- Eric Milam (J0hnnyBrav0) <brax.hax@gmail.com>
- Gianfranco Costamagna (LocutusOfBorg) <costamagnagianfranco@yahoo.it>
- Alexander Koeppe (koeppea) <format_c AT online DOT de>

## INSTALLATION

The easiest way to compile ettercap is in the form:

```
mkdir build && cd build
```

```
cmake ..
```

> [!TIP]
> Use `ccmake .` to change options such as disabling IPv6 support, add plugins support, etc).

```
sudo make install
```

> [!NOTE]
> Read [INSTALL](INSTALL) for further details... and [README.PLATFORMS](README.PLATFORMS) for any issue regarding your operating system.

## HOW TO USE IT

 You can choose between 3 User Interfaces: `Text mode`, `Curses`, `GTK`.

 Please read the man pages `ettercap(8)` and `ettercap_curses(8)` to learn how 
 to use ettercap.

## TECHNICAL PAPER

### THE HOST LIST

 Sending one ARP REQUEST for each ip in the lan (looking at the current ip
 and netmask), it is possible to get the ARP REPLIES and then make the
 list of the hosts that are responding on the lan. With this method even
 windows hosts, reply to the call-for-reply (they don't reply on
 broadcast-ping).
 Be very careful if the netmask is a class B (255.255.0.0) because ettercap
 will send 255*255 = 65025 arp requests (the default delay between two 
 requests is 1 millisecond, can be configured in etter.conf)


### UNIFIED SNIFFING

 Ettercap NG uses the unified sniffing method which is the base for all the
 attacks. The kernel ip forwarding is always disabled and this task is
 accomplished by ettercap itself. Packet that needs to be forwarded are packets
 with destination mac address equal to the attacker's one, but with different ip
 address. Those packets are re-sent back to the wire to the real destination.
 This way, you can plug in various mitm attacks at a time. You can even use
 external attacker/poisoner, they only have to redirect packets to ettercap's
 host and the game is over ;)


### BRIDGED SNIFFING

 Uses two network interfaces and forwards the traffic between them while performing
 sniffing and content filtrating. This sniffing method is very stealthy as there
 is no way to to detect that someone is in the middle. You can look at this as a layer
 one attack. Don't use it on gateways or it will transform your gateway into a bridge.
 
 HINT: You can use the content filtering engine to drop packets that should not pass.
 This way ettercap will work as an inline IPS ;)


### ARP POISONING ATTACK

 When you select this method, ettercap will poison the arp cache of the
 two hosts, identifying itself as the other host respectively (see the
 next section for this).
 Once the arp caches are poisoned, the two hosts start the connection, but
 their packets will be sent to us, and we will record them and, next,
 forward them to the right side of the connection. So the connection is
 transparent to the victims, not arguing that they are sniffed. The only
 method to discover that there is a man-in-the-middle in your connection, is
 to watch at the arp cache and check if there are two hosts with the same
 mac address!
 That is how we discover if there are others poisoning the arp cache
 in our LAN, thus being warned, that our traffic is under control! =)

```
     HOST 1  - - - - - - - - - - - - - - - - - - - -> HOST 2
   (poisoned)                                      (poisoned)
       |                                               ^
       |                                               |
        ------------> ATTACKER HOST  ------------------
                      ( ettercap )

 Legenda:
             - - - ->   the logic connection
             ------->   the real one
```

 The arp protocol has an intrinsic insecurity. In order to reduce the
 traffic on the cable, it will insert an entry in the arp cache even if it
 wasn't requested. In other words, EVERY arp reply that goes on the wire
 will be inserted in the arp table.
 So, we take advantage of this "feature", sending fake arp replies to the two
 hosts we will sniff. In this reply we will tell that the mac address of the
 second host is the one hard-coded on OUR ethernet card. This host will now
 send packets that should go to the first host, to us, because he carries
 our mac address.
 The same process is done for the first host, in inverse manner, so we have
 a perfect man-in-the-middle connection between the two hosts, legally
 receiving their packets!!

   Example:

     HOST 1:  mac: 01:01:01:01:01:01         ATTACKER HOST:
               ip: 192.168.0.1                    mac: 03:03:03:03:03:03
                                                   ip: 192.168.0.3

     HOST 2:  mac: 02:02:02:02:02:02
               ip: 192.168.0.2


   we send arp replys to:

            HOST 1 telling that 192.168.0.2 is on 03:03:03:03:03:03
            HOST 2 telling that 192.168.0.1 is on 03:03:03:03:03:03

   now they are poisoned !! they will send their packets to us !
   then if receive packets from:

            HOST 1 we will forward to 02:02:02:02:02:02
            HOST 2 we will forward to 01:01:01:01:01:01

   simple, isn't it ?

 **LINUX KERNEL 2.4.x ISSUE**

 In the latest release of the linux kernel we can find in :
 `/usr/src/linux/net/ipv4/arp.c`

```
 /* Unsolicited ARP is not accepted by default.
    It is possible, that this option should be enabled for some
    devices (strip is candidate)
 */
```

 these kernels use a special neighbor system to prevent unsolicited arp
 replies (what ettercap sends to the victim).
 Good gracious, is ettercap unusable with that kernel ? the answer is NO !
 let's view why... in the same source code we find:

```
 /*
 *  Process entry.  The idea here is we want to send a reply if it is a
 *  request for us or if it is a request for someone else that we hold
 *  a proxy for.  We want to add an entry to our cache if it is a reply
 *  to us or if it is a request for our address.
 *  (The assumption for this last is that if someone is requesting our
 *  address, they are probably intending to talk to us, so it saves time
 *  if we cache their address.  Their address is also probably not in
 *  our cache, since ours is not in their cache.)
 *
 *  Putting this another way, we only care about replies if they are to
 *  us, in which case we add them to the cache.  For requests, we care
 *  about those for us and those for our proxies.  We reply to both,
 *  and in the case of requests for us we add the requester to the arp
 *  cache.
 */
```

 so, if the kernel receives a REQUEST it will cache the host...
 what does that mean ? if ettercap sends spoofed REQUESTS instead of
 REPLIES the kernel will cache them ? the answer is YES !!

 ettercap 0.6.0 and later has this new ARP REQUEST POISONING method.
 it will alternate request and replies on poisoning because other OS doesn't
 have this "feature"...


 **SOLARIS ISSUE**

 Solaris will not cache a reply if it isn't already in the cache.
 The trick is simple, before poisoning, ettercap sends a spoofed ICMP
 ECHO_REQUEST to the host, it has to reply on it and it will make an arp
 entry for the spoofed host. Then we can begin to poison as always, the
 entry is now in the cache...


### ICMP REDIRECTION

 This attack implements ICMP redirection. It sends a spoofed icmp redirect
 message to the hosts in the lan pretending to be a best route for internet. 
 All connections to internet will be redirected to the attacker which, in turn,
 will forward them to the real gateway. The resulting attack is an HALF-DUPLEX
 mitm. Only the client is redirected, since the gateway will not accept redirect
 messages for a directly connected network. 


### DHCP SPOOFING

 This attack implements DHCP spoofing. It pretends to be a DHCP server and try
 to win the race condition with the real one to force the client to accept
 replies from it. This way the attacker is able to manipulate the GW parameter and
 hijack all the outgoing traffic generated by the clients.
 The resulting attack is an HALF-DUPLEX mitm. 


### PORT STEALING

 This technique is useful to sniff in a switched environment when ARP poisoning
 is not effective (for example where static mapped ARPs are used).
 It floods the LAN with ARP packets. The destination MAC address of each 
 "stealing" packet is the same as the attacker's one (other NICs won't see these 
 packets), the source MAC address will be one of the MACs of the victims.
 This process "steals" the switch's port of each victim. 
 Using low delays, packets destined to "stolen" MAC addresses will be received 
 by the attacker, winning the race condition with the real port owner. 
 When the attacker receives packets for "stolen" hosts, it stops the flooding 
 process and performs an ARP request for the real destination of the packet. 
 When it receives the ARP reply it's sure that the victim has "taken back" his 
 port, so ettercap can re-send the packet to the destination as is.
 Now we can re-start the flooding process waiting for new packets.


### CHARACTERS INJECTION

 We have stated that the packets are for us...
 And the packets will not be received by destination until we forward them.
 But what happens if we change them?
 Yes, they reach destination with our modifications.
 We can modify, add, delete the content of these packets, by simply
 recalculating the checksum and substituting them on the traffic.
 But we can do also more: we can insert packets in the connection.
 We forge our packets with the right sequence and acknowledgement number and
 send them to the desired host. When the next packets will pass through us
 we simply subtract or add the sequence number with the amount of data we
 have injected till the connection is alive, preventing the connection to be
 rejected (this until we close ettercap, who maintains sequence numbers
 correct, after program exit, the connection must be RESET or all future
 traffic would be rejected, blocking the source workstation network).

> [!NOTE]
> Injector supports escape sequences. you can make multi-line injection
>  - eg: "this on line one \n this on line two \n and so on..."
>  - eg: "this in hex mode: \x65\x6c\x6c\x65"
>  - eg: "this in oct mode: \101\108\108\101"

> [!NOTE]
> remember to terminate your injection with \r\n if you want to inject command to the server.

### SSH1 MAN-IN-THE-MIDDLE

 When the connection starts (remember that we are the master-of-packets, all
 packets go through ettercap) we substitute the server public key with one
 generated on the fly and save it in a list so we can remember that this
 server has been poisoned before.
 Then the client send the packet containing the session key ciphered with
 our key, so we are able to decipher it and sniff the real 3DES session key.
 Now we encrypt the packet with the correct server public key and forward it
 to the SSH daemon.
 The connection is established normally, but we have the session key !!
 Now we can decrypt all the traffic and sit down watching the stream !
 The connection will remain active even if we exit from ettercap, because
 ettercap doesn't proxy it (like dsniff). After the exchange of the keys,
 ettercap is only a spectator... ;)


### PACKET FILTERING

 Like character injection, we can modify the packets payload and replace
 the right sequence and acknowledgement number if needed.
 With the integrated filtering engine you can program your own filters
 to make the best filter for your aims.
 A scripting languages is used to make filters source that must be compiled
 with etterfilter(8) in order to be used by ettercap.
 

### PASSIVE SCANNING OF THE LAN

 This feature is very useful if you want to know the topology of the lan but
 you don't want to send any packet on it. In this way the scan is done entirely
 by sniffing packets and extracting useful information from them.
 This scan will let you know the hosts in the lan (it watches ARP request), the
 Operating System of the hosts (it uses passive os fingerprint... see next
 section), the open ports of an host (looking the SYN+ACK packet), the gateway,
 the routers or hosts acting as a router (it watches ICMP messages).
 As a passive method it is useless on a switched lan (because it can make a
 topology only of the host that are connecting to you), but if you put it on a
 gateway and let it run for hours or days, it will make a complete report of
 the hosts in the lan.


### PASSIVE OS FINGERPRINT

 The main idea is to analyze the passive information coming from a host
 when it makes or receives connections with other hosts. This information
 is enough to detect the OS and the running services of the host.
 In this scenario, we look at SYN and SYN+ACK packet and collect some
 interesting info from them:
 Window Size: the TCP header field
 MSS: the TCP option Maximum Segment Size (can be present or not)
 TTL: the IP header field Time To Live (rounded to the next power of 2)
 Window Scale: the TCP option indicating the Scale
 SACK: the TCP option for the Selective ACK
 NOP: if the TCP options contain a NOP
 DF: the IP header field Don't Fragment
 TIMESTAMP: if the TCP timestamp option is enabled
 and obviously the type of the packet (SYN or SYN+ACK)

 The database contains different fingerprints for each type of packet
 because some OSes have different fingerprints from SYN to SYN+ACK.
 Obviously the SYN fingerprint is more reliable, because the SYN+ACK is
 influenced by the SYN (if a SYN doesn't contain a SACK the SYN+ACK will not
 have the SACK option even if the host support it). So while collecting
 information off the lan, if we receive a SYN+ACK we mark the OS of that
 host as temporary and when we receive a SYN we confirm that.
 Fingerprints ending with an ":A" are less reliable... this is 
 because some OS identification may change during the gathering process.

 The SYN+ACK packets are also used to discover the open ports of a host.
 (see next section)

 The interesting thing is that firewalls, gateways and NAT are transparent to
 passive OS detection. So collecting info for the LAN will let you know info
 even for remote hosts. Only proxies aren't transparent because they make a
 new connection to the target.

 Our fingerprint database has to be enlarged, so if you find a host with an
 unknown fingerprint and you know for sure the OS of that host, please mail
 us <alor@users.sourceforge.net> the fingerprint and the OS, we will insert
 in the database.


### OPEN PORTS

 Open ports are identified by looking for SYN+ACK packets.
 If a SYN+ACK comes from a port, it is for sure open, except for the
 channel command of FTP protocol, for that reason SYN+ACK going to port 20
 are not used to indicate a open port.
 For the udp ports the question is a little bit difficult because no SYN or
 ACK packet are present in the udp protocol, so ettercap assumes that a udp
 port < 1024 that sends packets is opened. We know that in this way we cannot
 discover open ports > 1024 but they can go undetected as open when a client
 sends packet to a server.


### GATEWAY AND ROUTERS

 The gateway is simply recognized looking at IP packet with a non local ip
 ( checking the netmask ). If a non local IP is found, ettercap look at the
 ethernet address (MAC) and store it as the gateway mac address, then it
 search for it in the list and mark the corresponding ip as the gateway.

 Looking in the ICMP messages we can rely that if a host sends a
 TTL-exceeded or a redirect messages it is a router or an host acting as it.


==============================================================================

vim:ts=3:expandtab

