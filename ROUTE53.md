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

## Route53

Route53 provides two main services:
- Domain registrar
- Host zones on managed nameservers

Route53 is a global service as a single database.

A `hosted zone` is hosted on four managed nameservers. Hosted zones can be public (available on the public internet) or private (associated with specific VPCs).

Hosted zones store `record sets` - nameserver records used to control DNS traffic.

### Hosted Zones

A `hosted zone` is a DNS database for a domain or subdomain. Hosted zones are globally resilient.

Hosted zones can be created explicitly in Route53. A hosted zone is automatically created for you after a Route53 domain registration.

By default, zones are hosted on four R53 nameservers.

`Public hosted zones` are accessible from the public internet and VPCs.

`Private hosted zones` are only accessible from its associated VPCs. Private hosted zones can be associated with VPCs in other accounts via the AWS CLI only.

The `split-view` strategy involves having a public and private hosted zone for the same domain name. This allows you to specify different DNS behavior for internal users via the private hosted zone than for general public via the public hosted zone.