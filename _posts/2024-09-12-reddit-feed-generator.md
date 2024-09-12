---
layout: post
title: Custom Reddit feed using RSS
date: 2024-09-12 20:12:00 -0300
comments: true
---

I have been using RSS for some years but in the last few (since the X saga
began) I started to dive deeper into the possibilities with this protocol. It
is one of those awesome things most people don't have the opportunity to play
with. I'm not sure why it doesn't receive much attention, but [it is not dead
yet](https://isrssdead.com/).

For those who don't know, RSS allows you to follow other content sources by
subscribing to a feed. It was mainly used for blogs, but as of today it can
be used with other kinds of feeds like podcasts, security vulnerability
feeds, Mastodon accounts and many more.

The idea is receiving all the content in one single application, being able
to consume text offline, just from the sources you want, in chronological
order, with few distractions without other intermediaries between you and the
feed you follow.

Reddit provides an RSS feed for every subreddit, which is really cool.
However, following subreddits as RSS feeds sucks. There is always too much
noise, duplicated posts, things that you would not even notice accessing the
website depending on the filters you apply. If you follow many subreddit it
becomes unmanageable.

Last Saturday, I decided to fix this because I am on a quest to centralize as
much as I can in my RSS reader. Fortunately, Reddit still provides an API
that can allow us to fetch the content of a subreddit with filters applied.
So I just needed to transform the response from the API into an RSS feed.

A quick search revealed some GitHub projects that could be used for that, but
they were too much for what I needed. I am always skeptical of using tools
that are too generalist and have lots of dependencies, but I also didn't want
this to be another [unfinished
project](https://www.bytedrum.com/posts/art-of-finishing/), so I had to draw
a line. Thinking about the issue for a while, I decided it needed to:

- Be really simple and have few dependencies
- Consume Reddit API and spit out the RSS
  ([Atom](https://datatracker.ietf.org/doc/html/rfc4287)) feed to be subscribed to
- Filter posts by a given number of upvotes for each subreddit
- Provide links to the link shared, if any, and to the post/comments page
- Show the amount of comments
- Show images

An here is the result: [reddit-feed-generator](https://github.com/patrickrbc/reddit-feed-generator)

After the setup I quickly noticed I needed a couple of things more to be
happy:

- Run the server on my Raspberry Pi (alongside with other utilities). So I
  would not have the process running on my machine all the time and it would
  look like I'm fetching from the web.
- Provide an OPML file that can be imported by RSS reader so I don't have to
  import every feed manually. Ok, this was not necessary but whatever.

This project excites me because it empowers individuals to reduce reliance on
large websites. Computers and the Internet are incredible tools, and the
ability to tailor my experience to get more of what I want and less of what I
don't is liberating.

The same method could be applied to other websites that you want to consume. The
core idea is to ingest a source (via APIs, web scraping, or existing RSS feeds),
filter and modify the content, and produce a customized RSS feed. I don't expect
people to use this code, but I hope it inspires someone to get your hands dirty.
Build your own algorithm!

### Other cool stuff I've found along the way

- [hnrss](https://github.com/hnrss/hnrss) provides custom, realtime RSS feeds for Hacker News
- [AboutRSS](https://github.com/AboutRSS/ALL-about-RSS) is a collection of links
related to RSS
