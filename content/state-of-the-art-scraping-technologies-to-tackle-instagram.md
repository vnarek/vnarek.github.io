---
title: State of the art scraping technologies to tackle Instagram
tags: article
---

#### State of the art scraping technologies to scout the Instagram programmatically

Even thought it's not technically [illegal](https://en.wikipedia.org/wiki/Web_scraping#Legal_issues) to scrape, Instagram still tries to prevent this behavior, by rate-limiting and blocking IP addresses. There are platform as a service solutions, that can tackle this ([1](https://apify.com),[2](https://www.scrapingbee.com)). They run multiple instances of headless browsers to be able to render modern [SPA](None) applications and use multiple proxies to prevent blocking and detection of actual scrapers.

The downside is that they are pretty pricey even for the development purposes. For that reason I will try simulating their product using `selenium` and `http-request-randomizer` python libraries or their equivalents in a different language. It possible that the Instagram is going to reject requests from publicly known IP address lists. Buying access to a private one should be cheaper than `PaaS` mentioned above.

By using proxies we could host the code on cloud platform developed by the company I work in called Zerops for free.