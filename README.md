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

## Get my IP with DNS
```
$ dig myip.exposed @myip.exposed TXT

; <<>> DiG 9.8.3-P1 <<>> myip.exposed @myip.exposed TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4641
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;myip.exposed.			IN	TXT

;; ANSWER SECTION:
myip.exposed.		300	IN	TXT	"202.7.172.86"

;; Query time: 482 msec
;; SERVER: 178.62.119.204#53(178.62.119.204)
;; WHEN: Mon Mar 13 19:48:54 2017
;; MSG SIZE  rcvd: 55
```
Or for *just* your IP: 
```
$ dig myip.exposed @myip.exposed TXT +short
"202.7.172.86"
```

## Get my IP with HTTP(S)
```
$ curl -s https://myip.exposed
14.202.188.197
```

## Are my DNS requests being hijacked?

Well, when we queried the DNS server the IP the request came from was apparently `202.7.172.86` and
when we queried the HTTPS endpoint the _real_ IP the request came from was apparently `14.202.188.197`.

Something weird is going on:
* Our DNS traffic is possibly being transparently passed to a third party DNS server
* Our HTTP / HTTPS traffic is being proxied
* We're on a split tunneling or incorrectly configured VPN

### How can we check if our DNS is being hijacked? 
Simple, query a different DNS server:
```
$ dig myip.exposed @8.8.8.8 TXT

; <<>> DiG 9.8.3-P1 <<>> myip.exposed @8.8.8.8 TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28484
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;myip.exposed.			IN	TXT

;; ANSWER SECTION:
myip.exposed.		300	IN	TXT	"202.7.165.68"

;; Query time: 461 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Mar 13 19:58:45 2017
;; MSG SIZE  rcvd: 55
```
If all our DNS traffic is being fed to an ISP / third party DNS server the values vill be the same. These are infact the IP addresses of the recursive DNS servers our requests are being passed on to.

## Quickly see if ./amijackt
To make it easier for me to work out whether or not my DNS was being screwed with I wrote a check script:
```
./bin/amijackt
CRITICAL - DNS traffic is being hijacked. Use your VPN.
```
If everything looks good:
```
./bin/amijackt
OK - DNS and HTTP look OK. Better use your VPN just to be sure.
```

## That's all!
I really wish that ISP's, hotels and governments would stop screwing with our traffic.
DNS especially, is critical for application performance. If you screw with it you're
potentially making an already crappy experience even worse.
