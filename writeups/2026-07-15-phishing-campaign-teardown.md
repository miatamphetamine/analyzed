The short version

I started getting the same phishing scam in my inbox multiple times a day, every day, for three months. Instead of just deleting them, I decided to actually track the pattern — where they were coming from, what infrastructure they were using, 
and whether there was a way to get it shut down instead of just filtered out.

By the end, I had logged 65+ individual phishing emails, identified a shared piece of infrastructure tying most of them to a single server, and filed a formal abuse report to that server's hosting provider with the evidence to back it up.

What the scam actually was

Fake "your cloud storage subscription failed to renew" emails, designed to look like they came from a legitimate service, trying to get the recipient to click through to a fake payment page and hand over card details. 
Classic credential/payment harvesting phishing — but run at real scale, with the operator actively working to dodge spam filters.

What I actually did

1. Started reading the raw email headers, not just the message.
Every email has hidden technical metadata attached — where it actually originated, what server sent it, whether it passed or failed authentication checks (SPF/DKIM, which are basically ways email providers verify "does this sender actually own the domain it claims to be from"). 
I learned to read that metadata instead of just looking at the visible sender name, which is trivial to fake.

2. Noticed the emails were using a consistent evasion trick.
The body of each email was encoded in Base64 — a way of scrambling text so it doesn't visually contain the words or links a spam filter would flag. 
Once decoded, every email led to a link on storage.googleapis.com — Google's cloud storage service, being abused to host fake payment pages because a Google-owned domain looks trustworthy and slips past a lot of filters that would flag a sketchy .ru or .biz domain instantly.

3. Built a tracking tool instead of doing it by hand.
Claude & I built a web-based incident log where I could paste in a raw email and it would automatically pull out the useful fields — sending domain, originating IP address, and (later) auto-decode the Base64 body to grab the bucket name. 
This turned each new email from a five-minute manual dig into a paste-and-go entry.

4. Found the pattern that mattered.
Individually, the emails looked like they came from dozens of different random domains — that's on purpose, it's called a domain generation algorithm (DGA), 
used specifically to make each email look unrelated to the last one so spam filters can't just blocklist one domain and catch everything. 
But when I aggregated all 65 incidents, one originating IP address showed up behind 53 of them — meaning nearly the entire campaign, despite looking scattered on the surface, was routing through a single server.

5. Verified before acting on it.
Not every unusual detail in the data was actually a lead. One domain that showed up looked like it belonged to an unrelated legitimate business — I recognized that as a compromised or abused third party rather than part of the operation, and didn't treat it as a target. 
Knowing what not to chase is part of the job.

6. Wrote and filed a real abuse report.
Once I had the pattern, I wrote a formal complaint to the hosting provider (HostingDunyam, Turkey) that owns the IP address in question — laying out the specific IP, the volume of abuse (53 documented incidents), the technical fingerprint tying them together, 
and a clear, specific ask: suspend the instance. Also flagged the Google Cloud Storage buckets being abused directly to Google.

Skills this actually demonstrates


Email header / metadata analysis — reading SPF, DKIM, Return-Path, and routing information to separate real sender info from spoofed display info
Pattern recognition across a dataset — spotting that a DGA was in use, and that a shared IP address was the real common thread underneath dozens of seemingly unrelated domains
Basic encoding/evasion awareness — understanding why Base64 was being used and decoding it to get at the real payload
Tool-building to scale a manual process — recognized a repetitive task and built something to automate the extraction, then iterated on it
Writing a technical incident report for a real audience — translating raw findings into a report a hosting provider's abuse desk could act on, not just a list of complaints
Judgment under ambiguity — correctly identifying which data points were real leads and which were noise (the compromised third-party domain)


Status

Abuse report filed with HostingDunyam; awaiting response. Google Cloud Storage abuse reports filed on the buckets identified. Investigation ongoing — new incidents still being logged as they arrive.

Why this matters (to me, and maybe to a hiring manager)

Nobody assigned me this. I didn't do it because a class or a job required it — I did it because I got tired of being an easy target and decided to actually understand what was happening to my inbox instead of just tolerating it. 
That's most of what the job is: noticing something's wrong, being annoyed enough to dig in, and having the patience to follow it all the way through to a real report instead of stopping at "huh, weird."
