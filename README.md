# myip-pdns
PowerDNS pipe backend adapter powering myip.exposed heavily influenced by http://xip.io

I've run my own HTTP endpoint that returns my IP address for a while now and wanted
to make the process of getting my own IP even faster. Why bother with a HTTP
request with curl if I can simply do a DNS TXT record lookup and get my IP address.

I literally only need to do the DNS Lookup and can avoid that huge overhead
of a TCP connection followed by all that HTTP talk.

Okay, okay... While that may seem a legitimate reason to do this the _real_ reason
I decided to do this is to:

1. Get a _very_ simple intro to PowerDNS as a dynamic, authoritative name server
I'm familiar with, and run a few NSD authoritative name servers but I've never really
played with PowerDNS.

2. Detect when an ISP or suspicious person is transparently routing all my interesting
DNS requests to their own DNS servers as I travel the world.

Basically, I can run ./bin/amijackt and I'll have a good estimation of whether I'm
talking to my DNS servers directly (the request came from me via my default gateway)
or my request is coming via a transparent third party DNS server which could
serve me alternate IP addresses or redirect me to phishing pages.
