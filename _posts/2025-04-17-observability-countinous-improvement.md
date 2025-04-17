---
layout: post
title: Continuous improvement through observability
date: 2025-04-17 14:10:00 -0300
comments: true
---

Our platform didn’t start out with observability. In fact, it started with the
bare minimum needed to prove value to our first customers. That fast-paced
beginning ended up bypassing some best practices.

When we started integrating observability tools, the chaos felt impossible to
untangle.

- The number of errors was more than we could handle. Some of them were already
known, but we just kind of "let them slide."

- The number of painfully slow queries was more than we could optimize.

Over time (and with a lot of work), we started to overcome these challenges that
once felt way too big. Here are a few tips:

- Don’t be too strict with limits in the beginning. Setting a threshold that’s
  considered high (bad) by industry standards might actually be better than
  using what “everyone else” uses. That way, you can keep improving steadily,
  tackling the most critical issues first. Over time, you can lower the
  threshold and push quality even higher.

- Go way beyond the basics. In addition to classic metrics like machine
  resources and response times, define business-oriented metrics. That ensures
  your system is getting better for your users, not just on paper.

- Keep a close eye on signs of regression. Document and set alerts for the goals
  your team defines, so you can catch regressions as soon as they pop up and
  avoid letting things pile up.

Bringing this over to security, I see a lot of companies that genuinely don’t
know where to start because there’s just so much to fix. If you’ve got an open
wound, it’s easy to know what to patch first. But a lot of times, prioritization
gets fuzzy because it feels like there’s no way to solve everything in time.

Set a threshold (not too easy, not impossible), and keep fixing things bit by
bit—eventually, the tide will turn!

By the way, what do you all use to make sure your products are continuously
improving?
