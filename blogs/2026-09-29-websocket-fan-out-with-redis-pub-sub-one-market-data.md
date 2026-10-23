---
title: "WebSocket fan-out with Redis pub/sub: one market data connection for every service"
url: "https://sifting.io/blog/websocket-fan-out-redis-pub-sub-one-market-data-connection"
date: "2026-09-29"
author: "SiftingIO Team"
feed_url: "https://sifting.io/blog/feed.xml"
---
Share one market data WebSocket across services: read max_conn and max_subs from the ack, republish ticks on Redis, snapshot for late joiners. Python code.
