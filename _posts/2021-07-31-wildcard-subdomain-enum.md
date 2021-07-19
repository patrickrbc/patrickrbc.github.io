---
layout: post
title: 'Subdomain enumeration with wildcard records '
date: 2021-07-31 13:49:00 -0300
comments: true
categories: footprinting, subdomain, dns
---

**TL;DR**

Enumerating subdomains with wildcard records is tricky but not impossible, here
are some tips. Also, don't trust wildcards as a security mechanism for hiding
sensitive apps.

# The problem

If you did some subdomain brute-force enumeration in the wild you already
bumped into a record that resolves for any type of prefix. This is called a
wildcard record and it can be configured by inserting a record entry with a
label "**\***". This record will also resolve for other sublevels unless it is
inhibited by another record entry.

Many companies use wildcard records as part of their architecture. A well-known
example is Slack which uses it for their workspaces. For example, today I asked
my favorite DNS server to resolve the following records and got the same IP
address:

```
shopify.enterprise.slack.com 18.231.0.250
enterprise.slack.com 18.231.0.250
big-name-non-existent.slack.com 18.231.0.250
```

In this case you might conclude that there is a wildcard record
**\*.slack.com** and maybe we should ignore this domain in your subdomain
enumeration. However, you could end up missing something like
**status.slack.com** which does not resolve to this address. Instead it has a
CNAME pointing to another infrastructure that could be interesting to you.

It is curious how often subdomain enumeration tools mess up or do not handle
this kind of behaviour. Many times the wildcard records are just dropped
without any further check. The problem is that you might lose some interesting
apps by discarding them .

With that in mind, adding a wildcard record can be a tempting strategy to hide
your own services like a needle in the haystack. I can't blame anyone for doing
that, but just keep in mind that this is not going to save you for long.

# Finding interesting stuff

Thinking about how to make a better reconnaissance one could try to overcome
this problem by treating enumeration in wildcard records differently. The
response returned by the wildcard could be stored (sorted if it is multiple
entries) and every subsequent DNS response would be compared with this one.
Everytime we find a new response it would be saved in a map structure.

This would make sure we have at least one subdomain that points to that new
location that we found. However, *the world ain't all sunshine and rainbows*
and we could obviously have a different application sitting on a machine that
will only show up when we set a specific Host header in the HTTP request.

Therefore, this is just something you could use to have more places to look for
security vulnerabilities. There are many other more edgy cases (for example
when including CNAME) that can happen when trying to find assets using DNS. I
hope I can dig into that more in future posts.

Do you have any tips for finding apps on records with wildcard?
