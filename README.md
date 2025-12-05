Project 6: Recursive Domain Name Server

Overview

This is the code of my recursive DNS server on Project 6. The program, 4700dns, is both an authoritative nameserver of a particular zone and a recursive resolver of the client. It takes in DNS queries on UDP and searches its local cache/zone, and in case it is needed recursively searches the root servers and follows the delegation chain to locate the answer.

I coded it in Python 3, with the dnslib library to perform the low-level packet packing and unpacking, which left me to concentrate on the actual code of recursion and caching.

High-Level Approach

I did this by dividing the server logic into two major modes:

Authoritative Mode: When the query lies within the range covered by my zone file (e.g. example.com), I look up my local data, and supply the answer immediately with the AA (Authoritative Answer) flag on.

Recursive Mode: In case the query is on a foreign domain, then the recursive look up occurs.

The threading library was utilized in order to deal with concurrency and to make sure that the server did not block as it awaited responses upstream. Each request is spawned to a new thread (worker) enabling the server to serve many requests at the same time.

Key Features & Design

Recursion Loop: I followed an iterative loop (resolve function) which begins at the Root server. It takes a look at the response to determine whether it is an Answer, a Referral (NS records) or an Error. In case it is a referral, it seizes glue records (IPs) of the Additional section or resolves the NS name in case there is no glue.

Caching: I developed a RecordCache which is a dictionary storing the records with a key being (name, type, data). It honours the TTL (Time To Live). I look at the cache before making a query out. When a record expires it is deleted.

Bailiwick Checking: A issub helper function was introduced in order to avoid cache poisoning. I ensure that whatever I add to the cache or use is actually part of the zone which the server is authoritative.

CNAME Handling: The server does CNAME chains. In the case when a user requests an A record but receives a CNAME, my resolver recursively solves the CNAME target until it is an IP or it reaches the end of a road.

Retries (Reliability): Although I am in the CS4700 course, I applied the retry logic (maximum of 5 attempts to make a query) to make the server more resilient to packet losses.

Challenges

The most difficult aspect of this project was certainly solving the recursion logic in case of the lack of the glue records. To ensure that I could be sure that, in the event that I received an NS record without an IP address in the Additional section, my code would stop the current lookup operation and then recursively look up the IP of that Name Server before resuming the initial lookup.

The other issue that was tricky involved dealing with CNAMEs and caching. I needed to ensure that in the event of a CNAME being pulled out of the cache, I always verified whether or not I wanted to resolve the target and not simply issue the CNAME and continue on with my day.

Testing

I also tested the program with the run script given and the configuration files within the directory of configs.

I made sure that the server has all the necessary tests:

Local Tests: 1-local-a.conf, 2-local-vv-local.conf, 3-local-local0.conf, and 4-local-nxdomain.conf were all passed.

Recursive Tests: 10-root-ns.conf to 16-sub-sub.conf test passed.

Passed 17-bailiwick.conf and 18-cache.conf.

Usage

To run the server manually:

./4700dns [port] <rootip> <zone_file>

To run the test suite:

./run configs/1-local-a.conf

Dependencies

Python 3

dnslib (Installed through pip3 install dnslib)