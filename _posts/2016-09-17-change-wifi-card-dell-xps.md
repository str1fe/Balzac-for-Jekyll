---
layout: post-light-feature
title: Change wifi card on dell xps 15(9530)
description: "Running linux on the xps 15 has been problematic due to bad driver support for the standard broadcom wifi card. This is how you replace it."
category: articles
tags: [hardware, diy, xps]
comments: true
image:
  feature: "posts/2016-09-17-wifi/XPS.jpg"
---

As i never quite got the wifi card to function one hundred percent, i decided it would be easier to just switch it.
Fortunately this is a very simple task, and it takes about ten minutes. I ordered a **Intel Dual Band Wireless-AC 8260** card to replace
the broadcom card. Intel has great driver support for linux.

What you need:
![What you need]({{ site.url }}/images/posts/2016-09-17-wifi/tools.jpg)

- Small phillips screwdriver
- Small(T5) torx screwdriver
- New wifi card

Start by unscrewing the back cover. There should be 10 small torx screws:
![Unscrew the small screws]({{ site.url }}/images/posts/2016-09-17-wifi/unscrew.jpg)

Unscrew the two phillips screws under the small lid:
![Unscrew the under small lid]({{ site.url }}/images/posts/2016-09-17-wifi/unscrew-lid.jpg)

Locate the wifi card(top left):
![Locate wifi card]({{ site.url }}/images/posts/2016-09-17-wifi/inside.jpg)

Compare it with your new one to make sure you have bought one that will fit. I installed the intel card
some days ago, that is why the broadcom card is not installed in this picture.
![Compare cards]({{ site.url }}/images/posts/2016-09-17-wifi/side-by-side.jpg)

Remove the cover for the antenna connectors:
![Remove antenna cover]({{ site.url }}/images/posts/2016-09-17-wifi/unscrew-wifi.jpg)

Unclip the antenna connectors. I used a small nipper, but you can probably use a flat screwdriver as well.
Just be careful not to ruin the connectors when you do this. Be gentle:
![Unclip antenna]({{ site.url }}/images/posts/2016-09-17-wifi/antenna.jpg)

When both the antenna connectors are off, just pull out the old card and insert the new one
and reverse the process. The intel card had illustrations showing which connector goes where
black right, white left – just like the broadcom card.
