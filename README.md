# myip-pdns
PowerDNS pipe backend adapter powering myip.exposed heavily influenced by http://xip.io

I've run my own HTTP endpoint that returns my IP address for a while now and wanted
to make the process of getting my own IP even faster. Why bother with a HTTP
request with curl if I can simply do a DNS TXT record lookup and get my IP address.
