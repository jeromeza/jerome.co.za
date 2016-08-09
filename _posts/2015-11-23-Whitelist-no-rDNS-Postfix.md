---
layout: post
title: Whitelist mailservers with no rDNS - Postfix
---

For more info you can see the previous post, but the gist of it I was SUPPOSED to be receiving an email, which I wasn't.

After taking a look at my mailserver I found that it was rejecting the messages as the sender didn't have rDNS setup. As I automatically block all servers with no rDNS this was a problem. I didn't want to open up my server to spam, so I wasn't going the rule for everyone. Thus my only option was to whitelist the problem server.

Luckily with Postfix this is pretty easy.

The initial error:
<div class="highlight">
<pre>
Nov 20 19:59:35 meyling postfix/smtpd[10689]: NOQUEUE: reject: RCPT from unknown[209.203.37.214]: 450 4.7.1 : Helo command rejected: Host not found; from= to= proto=ESMTP helo=
</pre>
</div>

My reject rules - in this case reject\_unknown\_helo\_hostname is the culprit:
<div class="highlight">
<pre>
root@meyling:/# cat /etc/postfix/main.cf | grep reject\_unknown
   reject\_unknown\_sender\_domain
   reject\_unknown\_helo\_hostname,
   reject\_unknown\_reverse\_client\_hostname,
</pre>
</div>

The fix - create a whitelist file with the bad mailserver included AND set the Postfix rule BEFORE the reject so that it kicks in first (Postfix applies rules sequentially, from top to bottom). This is set in /etc/postfix/main.cf under smpd_recipient_restrictions.
<div class="highlight">
<pre>
smtpd\_recipient\_restrictions = permit\_sasl\_authenticated,
   permit\_mynetworks,
   check\_client\_access hash:/etc/postfix/whitelist\_helo
   reject\_non\_fqdn\_helo\_hostname,
   reject\_unknown\_helo\_hostname,
   reject\_unknown\_reverse\_client\_hostname,
   reject\_unauth\_destination
</pre>
</div>

We now need to create the whitelist file and include the bad servers IP which we wish to ignore.
<div class="highlight">
<pre>
root@meyling:/etc/postfix# cat /etc/postfix/whitelist\_helo
209.203.37.214 OK
</pre>
</div>

Next we run postmap to update our changes.
<div class="highlight">
<pre>
postmap /etc/postfix/whitelist\_helo
</pre>
</div>

Now all that's left is for us to reload Postfix with our new config.
<div class="highlight">
<pre>
service postfix reload
</pre>
</div>

We should now be receiving email from the badly configured mailserver! Now all that's left is to educate the admin on the other end and get them to fix, which is often easier said than done ;)
