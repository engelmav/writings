Title: Blind Analysis
Date: 2013-04-01 20:08
Tags: networking, wireshark, encryption, firewall, postgresql, Diffie-Hellman
Category: Solutions 

This was brutal - beware. But damn was it fun.

In my previous role, our firm was moving datacenters. Among the myriad problems this poses, there was another one added:
we would porting our APIs and websites to a new, privately-owned block of IP addresses (with some new hostnames). The challenge was given to me to determine who uses our services (websites and APIs) and inform them that the service endpoint addresses would be changing.

The problem was that, from the perspective of our applications, we could see *the fact that* people were connecting to our various services, but we could not tell over what circuits and to which external firewall they had connected to.

Let me try to explain.

From the outside in, it looks like this. For each circuit (including the Internet), we had a firewall. Behind each firewall stood a VIP (Virtual IP address appliance) for each service (including websites).

I would diagram this but fear revealing too much.

From our applications, we can see the universe of users connected. But we can't see the *medium over which they connected*. When I looked at the API server logs, the applications logged the source address as the internal VIP address, rather than the original source address. So I looked in the firewall logs. This presented an entirely different problem. From the firewall I could see the original source addresses of the incoming connections, but I could not tell who the clients were. I mean, I could run a whois on the IP addresses, but there were a couple of problems with relying on only whois data:

* many of the users connecting to use are large organizations - it is a needle-in-haystack scenario to call a multinational firm and say "someone is connecting to this IP address. Here is their source address. Can you find them and get us in contact with them? Not happening.
* many of the users connecting did not own their IP addresses, so the whois data only shows the ISP information.
* there are many, many spurious connections on these firewalls - bots, hackers, zombie API clients, etc.

There may be hope, I thought. These API connections contain certain messages on the application-layer that themselves contain usernames. I knew that if I could get the usernames, I could easily run them through our client database.

But there was a catch - the majority of these connections on the firewall were made over SSL encryption (and rightly so). It was time to do some packet sniffing.

Our Networks team had recently set up a Network Analysis Module (NAM) for a separate project. They had it watching the traffic on the external router. This thing was a beast. If I recall correctly, one second of capture was several hundred gigabytes, depending on the time of day. So I had one potential solution which offered me, up front, two new problems:

1. How do I sift through so much data to find the messages I need?
2. Can I decrypt these messages?

To address the first problem, I realized I would have to either continuously comb through the data, or accumulate some data over the course of some useful amount of time, and sift through it later. To sift through the packets, I would need to decrypt them.

WireShark, of course, would come to the rescue. In the past I had casually noted that you could decrypt packets "on the wire" using WireShark. All you really need is the
private key. You can then configure the WireShark command line or GUI to use that key. There is one important caveat: you need to have captured the SSL handshake in order to decrypt the consequent stream. So it was evident that if I were able to decrypt some API client streams, I would not be
reaching all of them, as some users would have logged in (read: performed an SSL handshake) after the time range of my packet capture and decryption.

I set out to get the private key from our Networks team. When I finally obtained it, I wrestled for hours and hours trying to set up WireShark to decrypt the packets. I was able to successfully decrypt packets over on a FIX connection (a few of our APIs were FIX) and literally *query for the COMPID* within the decrypted message. How cool is that?

Production was, for a reason I had yet to determine, a different story. No matter how much I tweaked the settings, I could not get the packets decrypted. After many conversations and google searches, I realized I was trying to decrypt a Diffie-Hellman encrypted stream. DF encryption
uses the session key as part of the encryption, so looking in packet captures after a session had already closed was useless. I could not do a live capture either, since the NAM would have needed to be recompiled with additional flags in order to "load in" the server key.

Alas, I was back at square one. The only view I have is of hundreds of firewall logs and hundreds of thousands of connections per day. In a way, this started feeling more fun. I had to take a programmatic approach off the bat.

The company had someone in their Networks team on a graduate rotation. I knew this guy liked Ruby, so I asked the Networks manager if he would mind parting with his grad resource for a little while. I had been toying a bit with Ruby and really enjoyed the syntax (still do). Could a better opportunity be found elsewhere? I thought not.

He and I set out planning and writing a program that would extract source and destination addresses. Source addresses were obviously useful for finding out who the company was (of course, still a bad quality data point, given the aforementioned challenges), and the destination addresses were important to determine which service a given connection was targeting. We studied the [ARIN](https://www.arin.net/resources/whoisrws/whois_api.html), RIPE, and APNIC APIs and made webcalls from the Ruby script out to these services.

We set up a Postgres database to store this information (it was a good opportunity to use [normalized form](http://www.phlonx.com/resources/nf3/). After this first pass, the grad got plucked and I was on my own again. Initially, we set this up such that hashes were flattening multiple IP addresses into one (in order to
remove the repeated log entries, for example in the case that one IP had established a connection serveral hundred times in a day). After the unique addresses were gathered from the logs, the HTTP calls  were made to the various ARIN-type  APIs.

The ARIN APIs were, in and of themselves, challenging. It was not the calls themselves, but the complete irregularity of the responses! I didn't want to receive blobs of text - I wanted a chunk of XML that I could actually represent in a Ruby object graph (think of consuming JSON in Javascript - instant, albeit untyped, composed objects). The challenge was that we needed to dig into these XML structures and find signs of "who owns this IP?" or "What company is this?" in addition to "Who are the points of contact?" 

This data is under particular headers, but the headers fall into different parts of the structure, depending on how much data was available for an entity.

Ironically, while writing this article, I noticed something on the ARIN documentation that might have helped:

>> 4.5.2. Showing Only POCs

>> Some organizations have many, many associated resources. Having all those resources returned when gathering information about an organization maybe very undesirable. To allow getting all the POCs of an organization without taking the penalty of retrieving all the resources of the organization, the showPocs=true parameter may be used. This parameter is only recognized when an organization's information is referenced by handleinformation exists in an ARIN entity. I settled on a few different common locations for this information and was able to get a few thousand company names with their corresponding IP addresses in my database (as well as their points of contact).

I stood back and flipped through the database  the program had generated several times. I still had a lot of unknowns (basically, some ISPs and some failed ARIN lookups).

I realized there was definitely value in how many times an IP address showed up in logs. If you are a high volume client, you will hit our APIs at least once per week. So I started summing the addresses with a hash in Ruby, instead of just flattening them. I came into the scenario of single IP addresses hitting several services. But since the hash "overwrote" a previous instances of a given IP address, I was also losing counts of clients hitting those other services.

This seemed overly complicated.

Then it dawned on me: why not just dump all the data in the database, and let SQL do the work? This was the answer. Never take for granted the years of blood sweat and tears implemented in highly optimized analytic functions (as well as your typical aggregates like sum() and avg()). So I ditched the hash completely. I wrote in all the IPs and all their duplicate connections into the database. I was then able to run queries like this:

```sql
SEELECT COUNT(cn.ip),cp.companyname
FROM connection cn
LEFT JOIN company cp
ON cn.ip = cp.ip
LEFT JOIN service s
ON cn.destination = s.destination
WHERE s.servicename = 'API 2'
GROUP BY cn.ip
```

Now I could do some actual analysis. How many times did each IP address connect? Per day? Per week?

(I should offer a plug for Postgres, as if it needed more - you can insert a date string without even specifying the string format, and Postgres will parse it into a datetime or timestamp. Of course you'd want to be more sure of this if the stakes are higher, but this was a sweet speed
boost).

It turned out the vast majority (literally around 99%) of the connections that were lower than one connection count per week were the failed lookups or ISPs. They were also the majority of the connections. What this told us was that our services really were being hit by a lot uninvited guests.
Good thing we were changing addreses.

So we had a somewhat better view of how many clients were impacted. The tens of thousands had been whittled down to a little over a thousand. This was still too much. A couple of days later, while troubleshooting a production issue, I noticed a detail in an email. One of the guys in the Networks
team had sent a log snippet of a client's session closing on the firewall. Therein lied the detail toward the end of one of the lines.

>> ``[...] Teardown TCP Connection {connection_id} for outside: {IP/Port} to inside: {IP/Port} duration 2:12:11 bytes 653244 [...]``

You have to be kidding me. That little piece - duration 2:12:11. My best friend.

How long would a *legitimate connection* be up for? Certainly longer than a second. I canvassed development and management. The general idea was that even clients that connect momentarily to download data would take somewhere over a few seconds at a time.

I re-wrote part of the program to capture this piece of data and enter it into the database (this is where Postgres's duration string parsing came in handy - a little less work for me).

I tweaked the query in a few ways to get a taste of the "legitimately connected" population. I tried ``having avg(cn.duration) > 1`` or ``where cn.duration > 1``. To my great surprise, this lowered the population to about 300.


Using the same database, I was able to split out (with ``group by`` clauses) the populations of each API (or website) that was targeting these old IP addresses. After showing these reports to management, I was told that some of the old IP addresses were actually what the DNS addresses of these services resolved to. In other words, if a client needed to connect to us, they were instructed to use someservice.somecompany.net, and this very DNS address resolved to one of the old addresses. Management and I decided I was able to write-off the people pointing to these DNS addresses. This was actually two websites, and we cared less about the website access vs our actual APIs used for business. Eliminating these website services with a simple ``NOT IN ('site1','site2')`` reduced our problem user population to around 50 unique IP addresses, most of which we had contact information for.

So we had our risk assessment.
