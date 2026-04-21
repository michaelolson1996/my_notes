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

             ROOT (.)
              /  \
             /    \
            /      \
     gTLD (.com)    cTLD (.us)
       /    \         /    \
      .a    .b      .ny    .md

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
- here is how a recursive query would work from a user's perspective:
  1. user searches for https://www.example.com in their browser
  2. the browser sends that hostname to the built-in stub resolver
  3. the stub resolver queries the locally configured name server
  4. the local server checks the local tables (its cache) but it is not there
  5. the local server sends a query to one of the 13 root servers for the hostname
  6. the root server does not do recursive queries and sends a refferral, meaning a list of servers that are authoritative over that TLD (gTLD or ccTLD)A .com
  7. the local name server chooses one of the authoritative servers and queries again for the hostname
  8. the gTLD authoritative server does not do recursive queries and sends a referral, meaning a list of servers that are authoritative over that SLD. example.com
  9. the local name server chooses one of the authoritative servers from the referral and queries for the A record from that server
  10. the domain name resolves to a CNAME in the zone file, so the authoritative server sends back the CNAME and A record (example.com)
  11. the local name server then sends that response, A record with IP and CNAME aliased to the A record to the original client stub resolver
  12. the stub resolver sends the A record and IP address to the browser
  13. The browser sends the request to the target server via IP address (xxx.xxx.xxx.xxx)

- the name server used when receiving the referral largely depends on the way the local name server is configured, but there are algorithms in place to determine which server is the best to use.

#### Iterative Queries

- Iterative queries, also referred to as non-recursive queries, are required to be supported on all name servers. Iterative queries may return the answer, any info it has to help the requestor get the answer, or an error.
- the 4 possible responses to an iterative query are:
  - the answer accompanied by any CNAMES (aliases), as well as whether the data is authoritative or not (cached)
  - an error indicating the domain does not exist (NXDOMAIN), may also contain CNAMEs (aliases) to the host
  - a temporary error indication in the case that DNS servers are not reachable due to network, etc.
  - a referral, list of 2 or more name servers (and their IPs) that are closer to the domain the client is requesting
- here is how a iterative query would work from a user's perspective:
  1. user searches for https://www.example.com in their browser
  2. the browser sends that hostname to the built-in resolver
  3. the resolver queries the locally configured name server
  4. the local server checks the local tables (its cache) but it is not there. the local server returns a referral, or list of the root servers
  5. the resolver sends a query to the root server for info about www.example.com
  6. the root server responds with a list of gTLD authoritative servers, aka a referral
  7. the resolver chooses one of the hosts from the list and sends a request to that chosen server
  8. the gTLD server responds back with a list of SLD authoritative servers, or a referral
  9. the resolver chooses a host from the list of SLD servers, and sends its request to that chosen server
  10. the zone file has www.example.com CNAME record, which points to the example.com A record. Both records are returned to the resolver
  11. the resolver sends the A record and IP address to the browser
  12. The browser sends the request to the target server via IP address (xxx.xxx.xxx.xxx)

- stub resolvers cannot follow referrals. Locally configured name servers must support recursive querying because of this fact.

#### DNS Reverse Mapping

- A special domain name is used for reverse mapping to allow for the above queries to work with reverse mapping, and that domain name is called IN-ADDR.ARPA
- given an ip address, 192.168.122.140, the subnet defines the network, while the last octet provides the host on the network. This is the opposite of DNS hierarchy
- so the solution is to cut off that last octet, reverse the remaining address, and append the reverse TLD, like so: 122.168.192.IN-ADDR.ARPA, as you can see the hierarchy is mantained.
- this is a genius solution, when you consider how names are resolved. We can now have this special TLD ARPA, and querying looks like this:

          root (.)
           /    \
          /      \
   gTLD (.com)   ARPA
                   \
                    \
                  IN-ADDR
                   /   \
                  /     \
                .192    .10

- seperate zone files are created for these special PTR records, here is an example of how $ORIGIN and a pointer record could look in a reverse zone file:

$ORIGIN 122.168.192.IN-ADDR.ARPA

140    IN    PTR    www.example.com.

- only one number can map to one domain name with PTR records, unlike A and CNAME resorce records.

- previously described queries (iterative and recursive) work for reverse DNS names, with the ARPA. domain residing under the root like other TLDs.
- Unlike forward domains, IPv4 addresses are allocated to Regional Internet Registries (RIRs). RIRs are listed:
  - APNIC - Asia Pacific, www.apnic.net
  - ARIN - North America & Southern Africa & parts of Carribean, www.arin.net
  - LACNIC - South America & parts of the Carribean, www.lacnic.net
  - RIPE - Europe & Middle East & Northern Africa & parts of Asia, www.ripe.net
  - AFRINIC - Will take responsibility of Africa from ARIN and RIPE, www.afrinic.net

### Zone Maintenance

- AXFR, or full zone transfers, are the old methods of updating slave servers. Changes made will be read by the slave once the SOA RR serial number has been increased, and a full transfer takes place, this happens as frequently as the refresh value is set. AXFR takes place at port 53/TCP
- IXFR is incremental zone transfer. Slaves and masters attempt IXFR by default, assuming their configs explicitly say not to. They will fallback to AXFR if necessary. As the name suggests, only changed data is transferred. IXFR takes place on port 53/TCP
- NOTIFY can significantly decrease time to propogate. Master server sends a NOTIFY probe to slaves to say hey, I may have gotten a change. Slave server checks the SOA sn to determine, then performs an AXFR or IFXR if that sn number has increased. 
- Dynamic Updating becomes a necessity for ISPs and organizations with frequent changes taking place. Classic method is to restart the DNS service in order to read the changes on restart
- the 2 architectural approaches are to either 1, allowing runtime updating from external programs, or 2, dynamically feed the zone RRs from a DB that can be updated. All records can be added or deleted excluding the SOA, or zone.
- you can have multiple master servers, but with Dynamic DNS you need a primary master server. The only different characteristic is that this is the server defined in the SOA




## References

 - https://www.iana.org/domains/root/db Root Zone Database
 - https://www.rfcreader.com/#rfc799 Internet Name Domains
 - https://www.rfcreader.com/#rfc1034 DOMAIN NAMES - CONCEPTS AND FACILITIES
 - https://www.rfcreader.com/#rfc1035 DOMAIN NAMES - IMPLEMENTATION AND SPECIFICATION
 - https://www.rfcreader.com/#rfc1591 Domain Name System Structure and Delegation
 - https://www.icann.org/en/governance/documents/cooperative-research-and-development-agreement-between-icann-and-us-department-of-commerce-15-05-1999-en CRADA - Agreement between ICANN and USDoC
 - https://www.rfcreader.com/#rfc1995 Incremental Zone Transfer in DNS
 - https://www.rfcreader.com/#rfc2136 Dynamic Updates in the Domain Name System (DNS UPDATE)
