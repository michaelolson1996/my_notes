# DNS

## High Level

- the concept of name servers were created in the mid 1970's

## DNS Hierarchy

- DNS uses a hierarchical naming structure. The order is root (typically a silent .), top-level domains (TLDs), second-level domains (SLDs), and any number of lower level domains.
- Top level domains are split into 2 types: Generic (gTLDs) and Country Code (ccTLDs) ex. .com, .edu, .net (gTLD) and .us, .io, .uk (ccTLD).
- google.com contains a gTLD (.com) and the SLD is google
- The authority for the root domain (.) lies with the Internet Corporation for Assigned Numbers and Names (ICANN)
- A zone refers to the unit of delegation when reading a name left to right, ex. www.google.com, google is the authority over www, com is the authority over google, etc.
- The left-most part of a domain name is referred to as the hostname (ie. www)

- Example hierarchy of DNS Delegation

-            ROOT (.)
-             /  \
-            /    \
-           /      \
-    gTLD (.com)    cTLD (.us)
-      /    \         /    \
-     .a    .b      .ny    .md

- There are currently 13 root servers world-wide
- These 13 root servers are known to every device world-wide due to a special zone file included in all DNS software
- The TLD nameservers (gTLD and ccTLD) are operated by various organizations under ICANN agreements
- Root servers are a. - m. and FQDNs are like so: a.root-servers.net. 
- Registry Operators contract with ICANN to operate the gTLD DNS servers, ex. Verisign operates the .com gTLD with multiple name servers
- Registrars handle the distribution of domain name allocation to the public and are accredited by ICANN, Verisign is also a registrar
- A DNS system consists of 3 components: the data that describes the domain(s) (resource records within zone files), one or more name servers, and a resolver.
- A resolver (typically stub resolvers are used on PCs) operates mainly in UDP and is responsible for translation of names to numbers. A browser would utilize a resolver library.
- A zone file translates a domain name into operational entities such as mail servers, hosts, services, etc. Zone files contain resource recors (RRs).
- In a master/slave relationship, the master holds the master copy of the zone file.

## Zone Files

- Forward mapping maps hosts to IP addresses in the zone, whereas Reverse-mapping is the mappting of IP addresses to hosts
- There are 3 types of entries in a zone file: comments (comments begin with a semicolon ;), directives (used to control processing of zone files, begin with dollar sign $), and resource records (define entities in a domain, are single line unless parenthesis are used in which case can span multiple lines).
- A zone file generally contains the following resource records:
  - $TTL - the time a RR may be cached or saved by another DNS server. (MANDATORY)
  - $ORIGIN - domain name for the file being defined. (OPTIONAL)
  - SOA - Start of Authority, must be first RR, and only SOA. Describes global characteristics of zone. (MANDATORY)
  - NS - Name Server, defines name servers authoritative to the domain, can point to servers in this domain or external. There needs to be a minimum of 2 NS RR's. (MANDATORY)
  - MX - Mail Exchange, defines mail servers of the zone. Can have zero. (OPTIONAL)
  - A - IPv4 addresses of all the hosts in the domain and are required to be publicly visible. (OPTIONAL)
  - AAAA - IPv6 addresses of all the hosts in the domain and are required to be publicly visible. (OPTIONAL)
  - CNAME - An alias name record to a host. (OPTIONAL)
- TTL directive in DNS refers to how long a record may be cached by DNS servers.
- The balance with TTLs is in access load (reducing the amount of queries and operational load on DNS servers) and change propogation (max time it will take for changes to propogate to all users).
- TTL can be defined for each individual RR
- ORIGIN is appended to any host records that do not have a FQDN (FQDNs are determined to end with a .) so a record of www would be www.example.com once ORIGIN is appended. www. would stay www.
- ORIGIN can be defined multiple times, and records are read in order so the most recent ORIGIN definition will apply to the following records.
- SOA RR syntax is as follows: name|ttl|class|rr|name-server|e-mail|sn|refresh|retry|expiry|min

@    IN    SOA    ns1.example.com. hostmaster.example.com. (
                  2003080800 ; sn = serial number
                  3h         ; refresh time
                  15m        ; retry = update retry
                  3w         ; expiry
                  3h         ; min = minimum
                  )
                  
- The @ symbol for name is equivalent to origin, or "example.com" in this example
- No ttl is set, so the default $TTL directive is used
- IN defines the class to be internet. Other values exist but are rarely used
- SOA is the resource record type
- name-server (MNAME) (ns1.example.com.) is for specifying the primary name server, has a special meaning when utilizing DDNS. The host still requires a NS RR to be defined
- e-mail (RNAME) (hostmaster.example.com.) defines the administrative email address for the zone (hostmaster@example.com)
- sn (serial number) needs to be updated whenever a zone file is updated, and the number must increase. In the above example we can see the zone file is from August 8, 2003. If I updated it today (April 21, 2026) it may look like 2026042100
- refresh is for master/slave relationships and determines when the slave will attempt to read the master SOA RR to determine if the sn has changed. If the sn has increased, a zone transfer will be initiated. Typically 3 - 24 hr
- retry is for master/slave relationships and determines time to wait between retries if the master is unreachable during a refresh cycle. Typically 10 - 60 m
- expiry is for master/slave relationships, in the event that refresh and retry do not work, the master will no longer be considered authoritative. If connection is made prior, the refresh and retry values are reset. Typically 1 - 3 w
- min when a dns name does not resolve, the DNS server will say no domain (NXDOMAIN) exists for the time specified as min. Typically 0 - 3 h

### Record Resources

- external NS must contain a zone file and must be either master or slave to main NS. Otherwise it is reffered to as lame delegation, no authority over the zone
- NS RR syntax is as follows: name|ttl|class|rr|name

       IN    NS    ns1.example.com.
       IN    NS    ns2.example.net.

- MX RR is the same as NS with the addition of the preference field, MX servers take priority with the lower number being the preffered server, usually 10 is most prefferred, in the case another server is added with a lower priority the field does not need to change
- MX ttl in the example is 3 weeks, we do not expect this value to change
- MX RR syntax is as follows: name|ttl|class|rr|preference|name

   3w  IN  MX  10  mail.example.com.
       IN  MX  20  mail.example.net.

- A/AAAA RR is where hostname is tied to an IP address.
- below are examples of valid implementations, where we can see multiple names are set to the same address, as well as one name being applied to multiple addresses. The DNS server will round robin or randomly send resolutions based on underlying configs. This may be used to provide load balancing to the services
- A/AAAA syntax is as follows: name|ttl|class|rr|ipv4/6

ns1    IN  A   10.10.10.10
mail   IN  A   10.10.10.11
www    IN  A   10.10.10.12
ww     IN  A   10.10.10.12
wwww   IN  A   10.10.10.13

web    IN  A   10.10.10.12
       IN  A   10.10.10.13

- CNAME RR is a canonical name resource, and maps a name to another name
- CNAME RRs should not be used with NS or MX records
- CNAME RRs produce more workload on the DNS server since both CNAME and CNAME'd RR must be looked up and returned, and if many CNAMES exist they could potentially exceed the UDP packet size, reducing performance
- the 2 cases where CNAME is mandatory is for referencing a host that is external to this domain, or when adding the www subdomain to the website
- CNAME syntax is as follows: name|ttl|class|rr|canonical-name

ftp    IN  CNAME ftp.example.net.

sftp   IN  CNAME www
web    IN  CNAME www
www    IN  A     10.10.10.12

- PTR RR pointer records contain the reverse lookup records, where an IP address is mapped to a hostname

- TXT RR historically used to define generic text, also useful to provide proof of domain ownership to external services

- NSEC, RRSIG, DS, DNSKEY, and KEY RRs used for DNSSEC

- SRV RRs are relatively new and used to map services to hosts

### DNS Protocol

- DNS protocol by default uses port 53/UDP with a block size limit of 512 bytes. If a response exceeds this size, TCP is negotiated and used. Zone maintenance operations use port 53/TCP for reliability.
- DNS systems support 3 types of queries:
  - recursive queries - DNS server takes responsibility for determining the answer, and will communicate with other name servers in order to get the answer. Name servers are not required to support recursive queries.
  - iterative queries - If the name server has the answer, it will return it. If it does not, it will send any info it deems to be useful for finding the answer, nothing more. All name servers must support these queries.
  - inverse queries - deemed obsolete, were used to find domain names via resource records.

#### Recursive Queries

- will either be fully answered or get a returned error
- name servers are not required to support recursive queries
- recursive queries are negotiated between the resolver and the name server via bits in the query headers
- 3 possible responses are:
  - The answer to the query along with CNAMEs (aliases), as well as info is authoritative or not (not meaning cached)
  - Error stating the domain name does not exist (NXDOMAIN) as well as any CNAMEs (aliases) to that domain name
  - A temporary error - meaning unable to reach other name servers due to network issues, etc.





## References

https://www.iana.org/domains/root/db Root Zone Database
https://www.rfcreader.com/#rfc799 Internet Name Domains
https://www.rfcreader.com/#rfc1034 DOMAIN NAMES - CONCEPTS AND FACILITIES
https://www.rfcreader.com/#rfc1035 DOMAIN NAMES - IMPLEMENTATION AND SPECIFICATION
https://www.rfcreader.com/#rfc1591 Domain Name System Structure and Delegation
https://www.icann.org/en/governance/documents/cooperative-research-and-development-agreement-between-icann-and-us-department-of-commerce-15-05-1999-en CRADA - Agreement between ICANN and USDoC
