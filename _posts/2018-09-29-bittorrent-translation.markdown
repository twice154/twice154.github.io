---
layout: post
title: "Peer-to-peer networking with BitTorrent"
subtitle: "Peer-to-peer networking with BitTorrent"
categories: essay
tags: webcdn
comments: true
---

안녕하세요, 저번 포스팅인 [지금까지 개발해온 webCDN에서의 아키텍처 개선](https://twice154.github.io/essay/2018/09/28/webcdn-architecture-improvement/) 에서 BitTorrent를 어느정도 참고하여 새로운 아키텍처를 설계하겠다고 마지막에 언급했었죠.

이번 포스팅에서는 BitTorrent에 관하여 공부할 때 제가 읽었던 가장 좋은 논문인 [Peer-to-peer networking with BitTorrent](http://web.cs.ucla.edu/classes/cs217/05BitTorrent.pdf) 를 제 나름대로 요약, 해석하고, 추가적인 정보들을 붙여보려고 합니다.

BitTorrent 창시자인 Bram Cohen이 발표한 논문 [Incentives Build Robustness in BitTorrent](bittorrent.org/bittorrentcon.pdf) 도 같이 보시면 많은 도움이 되니 읽어보시면 좋을 것 같습니다.

## P2P Network Topologies
****
1. Pure peer-to-peer
![pure-p2p](/assets/img/20180929/pure-p2p.png)
중앙에서 관리해주는 체계가 없는 P2P 시스템입니다.

2. Hybrid peer-to-peer
![hybrid-p2p](/assets/img/20180929/hybrid-p2p.png)
중앙에서 관리를 하는 존재들이 있긴 하지만, 그 관리자들을 최대한 분산시켜 놓은 시스템입니다.