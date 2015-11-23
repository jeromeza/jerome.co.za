---
layout: post
title: Why rDNS matters for mail!
---

My old ISP had gone to hell. They had started shaping and throttling traffic aggressively and I simply wasn't getting what I paid for...

The result being that I moved to a new ISP (Vox Telecom). I signed up and eagerly awaited my details via email, yet they never arrived. 

As I manage my own email server I decided to log in, and see if I could spot any issues, which I soon did.

Vox's mail servers aren't configured correctly (go figure). They a) haven't setup rDNS and b) resolve to a local DNS entry - so my mailserver had blocked them.

Checking mail logs I found:
<div class="highlight">
<pre>
Nov 20 18:59:22 meyling postfix/smtpd[9332]: NOQUEUE: reject: RCPT from unknown[209.203.37.214]: 450 4.7.1 <titania.localdomain>: Helo command rejected: Host not found; from=<help@voxtelecom.co.za> to=<me@mydomain.co.za> proto=ESMTP helo=<titania.localdomain>
</pre>
</div>

I reject mail if I can't verify the helo hostname - as it not being verifiable generally means you're a SPAMMER.
<div class="highlight">
<pre>
root@meyling:/# cat /etc/postfix/main.cf | grep reject\_unknown
   reject\_unknown\_sender\_domain
   reject\_unknown\_helo\_hostname,
   reject\_unknown\_reverse\_client\_hostname,
</pre>
</div>

A quick dig on the IP confirms what I suspected. They've got no rDNS record for the IP set and thus any decent mailserver will refuse their mail.
<div class="highlight">
<pre>
root@meyling:# dig -x 209.203.37.214

; <<>> DiG 9.9.5-9+deb8u3-Debian <<>> -x 209.203.37.214
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 54847
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;214.37.203.209.in-addr.arpa.	IN	PTR

;; AUTHORITY SECTION:
37.203.209.in-addr.arpa. 189	IN	SOA	ns1.datapro.co.za. hostmaster.datapro.co.za. 2013013004 1800 900 86400 3600

;; Query time: 0 msec
;; SERVER: 62.210.16.6#53(62.210.16.6)
;; WHEN: Fri Nov 20 19:31:13 SAST 2015
;; MSG SIZE  rcvd: 120
</pre>
</div>

So they need to setup rDNS and configure the server to use titania.localdomain as the server name if applicable.

Additionally they're not conforming to RFC 2821 (https://www.ietf.org/rfc/rfc2821.txt) for mail and for a huge ISP that offers connectivity / hosting / mail - I'd expect more:

**"The domain name given in the EHLO command MUST BE either a primary host name (a domain name that resolves to an A RR) or, if the host has no name, an address literal as described in section 4.1.1.1"**

I've let their support know, but I doubt they'll bother to fix. For now I'll whitelist them in my Postfix config. If you're interested in how, then check the next post.
