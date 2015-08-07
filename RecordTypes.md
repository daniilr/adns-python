# Introduction #

adns supports several DNS record types, in both raw and "cooked" formats. These all come
from adns.rr (resource records). Generally the
"cooked" formats may require several internal queries and return additional information.
All queries return data in a tuple as (_status_, _CNAME_, _expires_, _answer_), where:
  * _status_: status code from adns.status
  * _CNAME_: CNAME of the answer; if the target was not a CNAME, this will be None
  * _expires_: When the answer expires, in UNIX ticks
  * _answer_: The answer to the query

The format of _answer_ varies depending on the type of query/record. Since DNS queries
should be assumed to return multiple answers, _answer_ is always a sequence (tuple) of
resource records.

# Record Types and Answers #

In all examples below:

```
>>> import adns
>>> c=adns.init()
```

## A ##

Returns the IP address of the target.

```
>>> c.synchronous("www.google.com", adns.rr.A)
(0, 'www.l.google.com', 1167604334, ('216.239.37.99', '216.239.37.104'))
```

## ADDR ##

Like A, but returns 2-tuples of the address family and IP address:

```
>>> c.synchronous("www.google.com", adns.rr.ADDR)
(0, 'www.l.google.com', 1167604334, ((2, '216.239.37.104'), (2, '216.239.37.99')))
```

Note that 2 is the address family for IPv6, though these are IPv4 addresses.
(Address families are defined in RFC 1700.) So why is it 2 and not 1?...

http://www.iana.org/assignments/address-family-numbers

## CNAME ##

Returns the CNAME (if any) of the target:

```
>>> c.synchronous("www.google.com", adns.rr.CNAME)
(0, None, 1167864839, ('www.l.google.com',))
```

Note the _CNAME_ field of the result tuple will probably always be None, unless
the CNAME points to another CNAME, which is officially discouraged by the RFCs.

## PTR and PTRraw ##

Returns the hostname of the given IP address in in-addr.arpa notation:

```
>>> c.synchronous("104.37.239.216.in-addr.arpa", adns.rr.PTR)
(0, None, 1167676329, ('va-in-f104.google.com',))
```

**XXX** What's the difference?

## TXT ##

Returns a tuple of strings:

```
>>> c.synchronous("google.com", adns.rr.TXT)
(0, None, 1167605128, (('v=spf1 ptr ?all',),))
```

## MXraw ##

Returns MX records as (_priority_, _hostname_):

```
>>> c.synchronous("gmail.com", adns.rr.MXraw)
(0, None, 1167608363, (
  (5, 'gmail-smtp-in.l.google.com'),
  (10, 'alt1.gmail-smtp-in.l.google.com'),
  (10, 'alt2.gmail-smtp-in.l.google.com'),
  (50, 'gsmtp163.google.com'),
  (50, 'gsmtp183.google.com')))
```

## MX ##

Like MXraw, but resolves all the hostnames. Each hostname is returned as
(_hostname_, _status_, _ADDR_) where _ADDR_ is a sequence of ADDR return values:

```
>>> c.synchronous("gmail.com", adns.rr.MX)
(0, None, 1167605059, (
  (5, ('gmail-smtp-in.l.google.com', 0, ((2, '209.85.133.27'), (2, '209.85.133.114')))), 
  (10, ('alt1.gmail-smtp-in.l.google.com', 0, ((2, '64.233.167.114'), (2, '64.233.167.27')))),
  (10, ('alt2.gmail-smtp-in.l.google.com', 0, ((2, '64.233.183.27'), (2, '64.233.183.114')))),
  (50, ('gsmtp183.google.com', 0, ((2, '64.233.183.27'),))),
  (50, ('gsmtp163.google.com', 0, ((2, '64.233.163.27'),)))))
```

## NSraw ##

Returns NS records as hostnames:

```
>>> c.synchronous("gmail.com", adns.rr.NSraw)
(0, None, 1167676668, ('ns3.google.com', 'ns4.google.com', 'ns1.google.com', 'ns2.google.com'))
```

## NS ##

Like NS,but resolves all the hostnames. Each hostname is returned as
(_hostname_, _status_, _ADDR_) where _ADDR_ is a sequence of ADDR return values:

```
>>> c.synchronous("gmail.com", adns.rr.NS)
(0, None, 1167676668, (
  ('ns1.google.com', 0, ((2, '216.239.32.10'),)),
  ('ns2.google.com', 0, ((2, '216.239.34.10'),)),
  ('ns3.google.com', 0, ((2, '216.239.36.10'),)),
  ('ns4.google.com', 0, ((2, '216.239.38.10'),))))
```

## SOA and SOAraw ##

Returns the SOA record.

```
>>> c.synchronous("gmail.com", adns.rr.SOA)
(0, None, 1167691858, (
  ('ns1.google.com', 'dns-admin@google.com', 2006120100, 21600, 3600, 1209600, 300),))
>>> c.synchronous("gmail.com", adns.rr.SOAraw)
(0, None, 1167691862, (
  ('ns1.google.com', 'dns-admin.google.com', 2006120100, 21600, 3600, 1209600, 300),))
```

The only difference is that SOAraw returns the unmolested contact field and SOA
changes the first period in the contact field to an @ sign so that it is
immediately useful as an e-mail address.

The values are these tuples are described in detail in RFC 1035. Also see
http://www.zytrax.com/books/dns/ch8/soa.html

## SRV and SRVraw ##

Returns SRV records as (_priority_, _weight_, _port_, _hostname_):

```
>>> c.synchronous("_kerberos._tcp.example.com", adns.rr.SRVraw)
(0, None, 1167606966, ((1, 0, 88, 'krb1.example.com'),))
```

SRV, like MX, resolves all hostnames, whereas SRVraw does not.

SRV reccords are described in RFC 2052.

## HINFO ##

These are "host info" records, which are rarely seen on the internet,
due to being a security exposure.

## RP and RPraw ##

These are "responsible person" records. Like HINFO, they are rarely
seen on the internet.