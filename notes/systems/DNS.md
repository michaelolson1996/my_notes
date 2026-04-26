# DNS

### Name Space

- the 3 main functions of DNS are: Name space, Name Registration and Name resolution
- Domain names are interpreted right to left
- Domain name format, right to left, is root, TLD, Second-Level domain (TLD + SLD = zone apex), Third-Level domain
- Peices of a domain separated by the dot (.) delimiters are called labels
- the FQDN is the absolute reference to the domain
- each label must be less than or equal to 63 characters and letters, numbers, and hyphens are allowed in labels
- A FQDN can be less than or equal to 255 characters
- the root's label is called label of null
- DNS is a collection of databases forming a tree like structure that is indexed by domain names
- There is a site root-servers.org you can go and get detailed info regarding servers running as DNS root servers, very cool!
- the authoritative name server is the one holding the DNS name db specific to the requested domain, and the one that completes the process of converting a name into an address

### Name Resolution

#### Local Name Resolution

- occurs locally on end users machine without external contact
- operates with a host table, a simple text file which maps hosts to addresses
- prior to DNS, all machines downloaded a host table
- wildcard entries are not supported in local host tables
- can be used for small local networks
- improve performance
- fallback last resort method when DNS fails

#### DNS Resolvers

- The DNS resolver sits between the end user and the DNS tree
- The DNS when receiving a request will first see if it can resolve the query
- If the DNS resolver cannot, it will act on behalf of the client to perform the steps to getting the resolution
- the DNS resolver contains a copy of the root hints file with the 13 root servers

#### Iterative Name Resolution

- returns a referral (list of servers to contact for answer, up to client to choose server from list)
- it is the clients responsibility to keep querying servers until the required info is obtained

#### Recursive Name Resolution

- the server when not having an answer assumes the responsibility of the client and handles the requests
- it is the server's responsibility to obtain the DNS info requested by the client

#### Caching

- when a name is resolved, the DNS server will cache the data so as not to perform the queries for every request
- clients cache data as well
- negative caching is the act of caching failed requests
- cache-only servers are commonly used to reduce load, they do not act authoritative over any domains
- TTL determines the time to live of a resource record, and controls how long a resource record may be cached
- reverse queries are not cached and negative answers are cached with a different TTL

#### DNS Name Resolution Workflow

- end user types a name into the browser
- browser checks its cache
- browser asks the client OS
- OS checks its cache and host table
- OS sends a recursive query to the DNS resolver
- Resolver checks its cache
- Resolver sends request to root server based on the root hints file
- root server sends back a referral
- Resolver sends request to a TLD server from the referral list
- TLD server responds with a referral to the authoritative name server
- Resolver sends request to the authoritative name server
- Authoritative name server responds back with an answer containing the address
- Resolver then caches the information and forwards the request to the client
- OS stores the information in its cache and passes the data to the browser
- browser will typically then cache the information as well

#### Reverse Name Resolution

- Special TLD known as in-addr.arpa that treats IPv4 octets as separate domains
- the octets are flipped because the most important info is on the right initially, so DNS wants the data reversed
- reverse names are not required and may not exist for certain domains

### Name Registration

#### Domain Name Registration Hierarchy

- At the top is ICANN stands for Internet Corporation for Assigned Names and Numbers
- Main role of ICANN is to oversee the huge and complex interconnected network of unique identifiers that allows computers on the internet to find one another
- ICANN is responsible for managing generic TLDs and country code TLDs
- ICANN manages how root servers function
- ICANN coordinates allocation of IP addresses to avoid repitition and clashes
- ICANN maintains a central repository of IP addresses
- Beneath ICANN are the 5 regional internet registries
- These 5 RIRs are responsible for obtaining IP addresses from ICANN and allocating them to ISPs across a specific geographic region
- Subordinate to the registries are the registrars, ICANN accredited orgs responsible for processing the registration of domain names (GoDaddy, NameCheap, Bluehost, etc.)
- Beneath registrars are resellers, third party companies that offer domain name registration services through registrars, i.e. Route 53
- At the bottom of the hierarchy are the registrants, people who register domains

#### Domain Name Registration Process

- registrant chooses a domain name and submits a request to a reseller or a registrar
- Registrar registers the name and then creates a whois record
- whois record contains info on registrant, registration date and expiry date
- registrar will then send the domain request to the appropriate registry, the registry will then file the info
- The registry will add the domain zone file to the master servers which will tell other servers where your website is located

### DNS Storage

- DNS data is stored in a database that is known as a zone
- there are 2 types of zones, forward lookup zones and reverse lookup zones
- zones are a collection of resource records
- Common RR format contains: name, type, class, ttl, rdlength (resouse data length), rdata (actual data)
- SOA record is start of authority, should be first RR and indicates beginning of a zone
- One SOA record per zone
- SOA - <domain name> <TTL> <class> SOA <m-name (primary authoritative name server)> <r-name (email of admin responsible for the zone)> (<serial number> <refresh interval> <retry interval> <expire interval> <minimum>)
- every zone must have a minimum of 2 NS resource records, NS records ensure the availability of a domain
- name servers typically reside in topologically separate networks for resiliency

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

## DNS Protocol

- DNS protocol by default uses port 53/UDP with a block size limit of 512 bytes. If a response exceeds this size, TCP is negotiated and used. Zone maintenance operations use port 53/TCP for reliability.
- DNS systems support 3 types of queries:
  - recursive queries - DNS server takes responsibility for determining the answer, and will communicate with other name servers in order to get the answer. Name servers are not required to support recursive queries.
  - iterative queries - If the name server has the answer, it will return it. If it does not, it will send any info it deems to be useful for finding the answer, nothing more. All name servers must support these queries.
  - inverse queries - deemed obsolete, were used to find domain names via resource records.

### Recursive Queries

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

### Iterative Queries

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

### DNS Reverse Mapping

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

## DNS Types

### Master Name Server

- the term master relates to the zone file location rather than any operational characteristics, a master may be expected to make zone transfers to other servers
- a master server obtains zone data from a local zone, whereas a slave server obtains zone data via zone transfers
- zone file changes need to be synchronized between zone masters
- if caching behavior is permitted and recursive queries are enabled, a master server will perform all actions to resolve a domain for a request
- if caching behavior is permitted and recursive queries are disabled, a master server will return: a answer from cache, a referral, or an error
- if caching behavior is not permitted and recursize queries are disabled, a master server will return either a referral or an error

### Slave Name Server

- master servers require reloading for zone changes, slave servers are updated via zone transfers. So while a master is inaccessable a slave will act as the authoritative server to a zone. This authoritative capability time period is defined in the zone SOA rr, expiry value
- a slave server attempts to update its zone records when the refresh value from the SOA rr has been reached, and if the slave server has not reached the master server by the time the expiry time from the SOA rr has been reached, it will stop serving requests for that zone or responding to queries
- when multiple  slave servers are in play, the master server can be hidden from public view. That way, if a zone file is corrupted in a slave server remediation becomes simpler, whereas if the master server gets corrupted that could mean requiring backup files
- the allow-notify param in slave servers specify which servers may send a NOTIFY, or master servers. Slave servers only accept NOTIFYs from master servers

### Caching Name Servers

- a server can cache records that have been requested, and adheres to the record TTLs for how long to cache
- a caching server only sets the response to authoritative when the records are directly from the master or slave, not from cache
- caching increases DNS performance and reduces network loads
- if a caching server is reloaded or restarted, the cache is wiped clean and the process starts over
- setting the recursion option to yes (default BIND behavior) will enable caching for that server
- if a server provides caching services, it must support recursive queries
- most common configs are: a general purpose (master or slave) server that also caches, and a caching-only server typically to support stub-resolvers
- cache poisoning and other malicious attacks are opened when caching is enabled

### Forwarding (Proxy) Name Servers

- forwards all queries to another name server and caches the results
- since the server is local, this can increase resolution speeds and decrease external network traffic
- can be used to ease maintenance, external changes can be resolved with a simple config change, no responsibility of zone management

### Stealth (DMZ or Split) Name Servers

- a stealth server does not appear in any publicly visible NS resource records for the domain
- a stealth zone will provide DNS services to its internal network, and is configured to be used by internal services, then usually a separate DNS server is used externally
- the purpose of a stealth DNS server is to conceal the internal network structure and identities

### Authoritative-only Name Servers

- authoritative-only name servers do not cache, and provides authoritative-only responses to queries (master or slave)
- typically used in 2 scenarios: split name servers to provide perimeter security in the DMZ, or high-performance (ie root servers, TLD servers, high volume sites)

## Common DNS Tasks

### Working with Subdomains

- domain name owners are able to pass domain responsibilites to a subdomain owner, subdomain owner takes responsibility of all zone files
- virtual subdomains use a sigle zone file for domain and subdomain to provide subdomain addressing
- subdomains are invisible to TLD name servers
- when working with virtual subdomains, they can be included directly in the same zone file as the actual domain, typically adding a $ORIGIN initialization
- glue records are A records (or AAAA if IPv6) which are required for every name server within a domain
- glue records are required in order to properly direct requests to the subdomain name servers, if they are not included the hosts would never know how to reach the subdomain name servers
- virtual domains eliminate the need of additional nameservers while still taking advantage of the naming scheme
- the downside of virtual domains is that any reload to either zone requires reload of an otherwise unchanged zone
- the $INCLUDE directive may be included to add files to the zone file

### Mail-Server Failover

- multiple mail servers may be added with a preference number, the lower the number the higher the preference. If the main MX server fails, those servers with higher priority numbers should be configured to hold onto the mail until the main server is back up, then forward those messages

### Reverse Subnet Maps

- when dealing with reverse subnet mappings, as opposed to the easy 122.168.192.IN-ADDR.ARPA C class naming convention, lets say there are 4 domains with a /26
- as opposed to naming a host 1.122.168.192.IN-ADDR.ARPA, it would now include the ip address, broadcast address, and subnet mask within the name. A assignee dns name server may contain records like so (assume the assignee name server domain is example.net):

$ORIGIN 122.168.192.IN-ADDR.ARPA

64/26   IN   NS   ns1.example.com
64/26   IN   NS   ns2.example.com

65      IN   CNAME   65.64/26
66      IN   CNAME   66.64/26
;...
126     IN   CNAME   126.64/26

- this method forces the CNAME lookup to use the name servers defined for the subnet, ns1 and ns2.
- The assigner zone file may look like the following:

$ORIGIN 64/26.122.168.122.IN-ADDR.ARPA

        IN   NS    ns1.example.com
        IN   NS    ns2.example.com

65      IN   PTR   ftp
66      IN   PTR   www
;...
126     IN   PTR   fred

### Load Balancing

- for load balancing mail, there are 2 possible strategies. Consistent across both strategies is the same preference numbers for MX servers. For demonstration purposes there are three MX servers: mail1, mail2, mail3. These servers ip's in the same order are: 10.10.10.1, 10.10.10.2 and 10.10.10.3
- The first method is different resource records for each server:

       IN    MX  10   mail1
       IN    MX  10   mail2
       IN    MX  10   mail3

mail1  IN    A      10.10.10.1
mail2  IN    A      10.10.10.2
mail3  IN    A      10.10.10.3

- note the fact that the preference number is consistent across all three servers, typically uses a round robin fashion of load balancing. The second example uses one MX rr and achieves the same objective:

       IN    MX  10   mail

mail   IN    A      10.10.10.1
       IN    A      10.10.10.2
       IN    A      10.10.10.3
       
- the first approach puts the load on the client sending the request in order to determine which MX server it will use, the second method puts the load on the name server.
- other services can be load balanced in a similar fashion (i.e. a website via multiple A records)
- the config option bind uses is called rrset-order, this allows you to manipulate how load is balanced
- SRV records serve an importance in load balancing and partly came about due to LDAP, they contain a weight and a priority for fine grained handling
- The rrset-order directive allows users to set the LB order, see example:


options {
  rrset-order { order fixed; }  ; order services were defined
  rrset-order { order cyclic; } ; round robin
  rrset-order { order random; } ; randomly chosen
}

- caching can significantly impact the effectiveness of DNS load balancing, so at times TTLs can be set to zero, or under 60 seconds
- DNS load balancing happens at the network level, it is incapable of understanding services and resource utilization, you would need a layer 7 load balancer for that type of setup

### SPF Record

- A Sender Policy Framework (SPF) is for a domain and its mail servers. Its purpose is to legitimize sources of email from that domain. The intent is to allow a receiving Message Transfer Agent to verify the source IP is authorized to send mail from the domain it claims to be part of
- SPF records were deprecated in 2014, the current standard is to define the SPF within a TXT RR
- the SPF fields are as follows:
  - v=spf1 field, mandatory, specifies the version (currently only supported version is 1)
  - pre field, defaults to + (pass), can also be - (fail), ~ (softfail) and ? (neutral)
  - type field, specifies the mechanism type to use for verifying the sender
  - mod field, two options available:
    - redirect redirects verification to more processing in the result of a failure or to provide a single domain-wide spf policy
    - exp should come last, defines the name of a site to redirect users to notify that they are not authorized to send mail through the domain
- the SPF type param defines either the mechanism to be used to verify the sender, or to modify the verification sequence
  - basic mechanisms, do not define a verification mechanism but affect the verification sequence:
    - include:domain recurse testing using supplied domain, common when clients send mail through ISP servers
    - all terminates a test sequence if no positive results have been found previously
  - sender mechanisms define a verification mechanism
    - ip4 type uses ip or range of ips for verification, ie ip4:192.168.122.0/24
    - ip6 type uses same concept as ip4 type, but it is ip6
    - type a format verifies based on A RRs within the domain. Benefits, if IP changes this need not be updated, downsides is it does add 1 additional check for each verification
    - type mx format verifies based on MX and A RRs. This is the MX RR for the domain, may not be the same as the SMTP for the domain








## DNS Security

### DNS Security Threats


| Number | Area             | Threat                                                     | Classification       | Solutions                                                |
|:------:|------------------|------------------------------------------------------------|----------------------|----------------------------------------------------------|
|   1    | Zone files       | File corruption (malicious or accidental)                  | Local                | System Administration                                    |
|   2    | Zone transfers   | IP address spoofing                                        | Server-to-Server     | Network Architecture, Transaction signatures, or disable |
|   3    | Dynamic updates  | Unauthorized updates, IP address spoofing                  | Server-to-Server     | Network Architecture, Transaction signatures, or disable |
|   4    | Remote queries   | Cache poisoning, data interception, subverted master/slave | Server-Client        | DNSSEC                                                   |
|   5    | Resolver Queries | MitM, Spoofing, cache poisoning, subverted master/slave    | Remote Client-Client | DNSSEC                                                   |

### DNS Security Classifications

- local threats (1) are typically the easiest to prevent, this is sys admin tasks such as file permissions, strong security at the OS level, bind chroot jail
- server-server (2) zone transfers can be protected via tools such as TLS, TSIG and TKEY, and BIND configuration options
- server-server (3) dynamic DNS can be protected via network architecture, TLS, TSIG and SIG(0), and implementing stealth name servers
- server-client (4) the relationship between orgs and end users is very important, DNSSEC can increase trust and ensure security is protecting against things such as cache poisoning
- client-client (5) this involves the concept of a security-aware resolver

### Administrative Security

- maintaining up to date software is crucial to ensure proper security. Keeping up to date on newly released exploits as well as maintaining a living upgrade checklist will dramatically increase speed and reduce risk. Block the BIND version from being publicly available
- limiting functionality will reduce risks, for example instead of running a master-slave DNS strategy, you may want to consider having multiple masters and no zone transfers in the strategy. This can reduce an attacking surface
- use defensive configuration as opposed to relying on the default behavior of the currently installed software. If an option is disabled by default, you can explicitly set that in the config to explicitly state that the feature is disabled. This can help notify admins of the config as well as prevent changes in the case future software updates alter their default behavior
- global denial of actions, then specifying allow settings within individual zones implements the deny-all default mode and is a good security measure to take
- limiting permissions takes two forms, limiting outside users from being able to edit or view files used by the DNS software, and containment which means limiting DNS capabilities in the case that a name server were to be compromised
- DNS software (BIND) can be run in three different ways: run as root (not recommended), run as a unique user (standard), or run in a sandbox or chroot jail
- the concept of chroot limits the software to a single directory, all required files then need to exist within that directory
- software diversity should be implemented, for example running different OSes per DNS server seeing as an OS vulnerability is probably not available across all OSes
- Intrusion Detection Software (IDS) can be deployed to monitor for any undesirable actions being performed

### Cryptography

- cryptographic algorithms can not be provably secure, rather algorithms are tested against and then deemed either secure or insecure. all techniques are based on a concept of computationally infeasable
- cryptography is used for three purposes: confidentiality, authentication and data integrity
- for DNS, only authenticity and data integrity are considered. Where confidentiality is required would be over TLS, BIND does not support this
- there are two classes of key-based cryptographic methods: symmetric and asymmetric
- symmetric key algorithms, commonly called single-key or shared-key, use a single shared key to encrypt and decrypt data. Examples of symmetric key algorithms are DES, AES, IDEA and RC4, with typical key sizes being 64, 128, or 192 bits
- shared secrets are utilized in DNS TSIG operations, and are defined at the global level as opposed to per-zone
- these keys are stored in different files and granted very restricted access, DNS does not care how the keys are shared between DNS, they can be distibuted via a piece of paper
- asymmetric encryption, also called public-key encryption, works by having a public key (viewable to anyone) and a private key (not shared)
- the most widely used asymmetric encryption algorithms are RSA and elliptic curves, with typical key sizes being 512, 1024, or more bits
- encryption systems require a lot of computation, and since DNS does not require confidentiality message digests, or hashes can be used
- a message digest is a one-way hash. The sender performs a hash function on the plain text data, and sends both to the receiver. The receiver then performs the same hashing function on the plain text and compares it to the result provided by the sender. This can verify the integrity of the message
- there are two methods for ensuring the authenticity of the sender. For symmetric encryption Message authentication Codes (MACs) are used, for asymmetric encryption Digital Signatures are used
- MAC is the result of combining the plain text with the privately shared key in the hashing algorithm, which can then be verified on the other end
- common forms of MAC are the HMAC-SHA256 and HMAC-SHA512. MACs are used for TSIG
- digital signatures perform the hashing function on the plain text, then encrypt the result via the private key. The receiver can the decrypt the message via the public key, and perform the hash function on the included plain text to verify the result matches the provided hashed text
- digital signatures are used for SIG(0) and DNSSEC
- a replay attack is when a transaction is captured and replayed at a later time, to circumvent this attack all systems must be time synchronized

### Securing Zone Transfers

- There are three methods of securing zone transfers cryptographically:
  - TSIG - MAC, shared key, symmetric encryption, no changes to zone files or rr's, change is applied in config file
  - SIG(0) - Digital Signature, assymetric encryption, supported for DDNS but not for full zone transfers
  - TKEY - must be authenticated with TSIG or SIG(0), not widely implemented

- Transaction Signatures (TSIG) use a Message Authentication Code (MAC) and a shared key to authenticate and ensure integrity of data
- TSIG can be used for securing zone transfers (AXFR) and Dynamic DNS (DDNS) for authorization and integrity checks
- DNSSEC is performed on RR sets rather than individual RRs



## References

 - https://www.iana.org/domains/root/db Root Zone Database
 - https://www.rfcreader.com/#rfc799 Internet Name Domains
 - https://www.rfcreader.com/#rfc1034 DOMAIN NAMES - CONCEPTS AND FACILITIES
 - https://www.rfcreader.com/#rfc1035 DOMAIN NAMES - IMPLEMENTATION AND SPECIFICATION
 - https://www.rfcreader.com/#rfc1591 Domain Name System Structure and Delegation
 - https://www.icann.org/en/governance/documents/cooperative-research-and-development-agreement-between-icann-and-us-department-of-commerce-15-05-1999-en CRADA - Agreement between ICANN and USDoC
 - https://www.rfcreader.com/#rfc1995 Incremental Zone Transfer in DNS
 - https://www.rfcreader.com/#rfc2136 Dynamic Updates in the Domain Name System (DNS UPDATE)
