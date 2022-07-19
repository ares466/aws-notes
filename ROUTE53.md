# Route53

## DNS

DNS is a discovery service that translates machine addresses (i.e., IP addresses) into human-readable addresses (e.g., www.google.com) and vice versa.

Since there are so many addresses, DNS must be distributed.

DNS records are distributed into `zones` represented by `zone files`. In each zone file, there are DNS records associated with specific domains and subdomains. Zone files are hosted by special DNS servers called `Nameservers`.

`DNS resolvers` are responsible for translating a human-readable address to an IP address, and vice versa. It does this by querying the zone file associated with that domain.

A `DNS client` is a piece of software running on an OS in some device (e.g., laptop, phone, tablet, PC) that queries the DNS resolvers.

DNS is a distributed database. At the top of the DNS tree are the `root nameservers`. The root nameserver is represented as a `.` at the end of the domain name.

`www.amazon.com.`

There are 13 root nameservers that are the first hop in any DNS query. The DNS resolver in your OS contains as `root hints` file that points to the DNS root servers.

> In reality, each "root nameserver" is actually a cluster of servers for high throughput.
>
> You can view the root name servers on [iana.org](https://www.iana.org/domains/root/servers).

The `authoritative servers` are the name servers that are delegated to respond to DNS queries for a specific zone (e.g., the Amazon nameservers are the authoritative servers for amazon.com).

`gTLDs` are global top-level domain (TLD) names (e.g., .com, .org, .edu). `ccTLDs` are country code TLDs (e.g., .co.uk, .de).