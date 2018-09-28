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
#### 1. Pure peer-to-peer
![pure-p2p](/assets/img/20180929/pure-p2p.png)
중앙에서 피어를 관리하는 허브가 `전혀 없는` P2P 시스템입니다.

모든 피어들이 동일한 역할을 수행하고, 100% 분산되어 있는 형태로 이루어져 있기 때문에, 피어 몇 개가 이탈한다고 해도 전체 퍼포먼스에 거의 영향을 미치지 않습니다.
또한, 공격취약점이 없다는 것도 큰 장점입니다.

#### 2. Hybrid peer-to-peer
![hybrid-p2p](/assets/img/20180929/hybrid-p2p.png)
중앙에서 피어를 관리를 하는 허브가 `있긴 하지만`, 허브들을 최대한 분산시켜 놓은 P2P 시스템입니다.

허브를 아무리 분산시켜 놓았다고 해도, 해당 허브가 작동하지 않을 경우에는 그 하위에 연결되어 있는 피어들이 고립된다는 문제점이 있습니다.
또한, 허브가 공격취약점이 될 수가 있습니다.

하지만, 중앙 허브들끼리의 통신을 통해서 데이터를 빠르게 주고받을 수 있기 때문에, Pure P2P 보다 속도 측면에서는 장점이 있습니다.

## BitTorrent Architecture
****
BitTorrent 아키텍처는 크게 4 부분으로 구성되어 있습니다.
> - Torrent File : filename, size, hashing information, URL 등의 `메타데이터`를 포함합니다. 실제 BitTorrent Client로 열게 되는 파일입니다.
> - Tracker : 피어들의 다운로드 상태를 기록하고, 피어들을 찾아주는 `시그널링 서버`와 같은 역할을 합니다.
> - Seed : 파일의 전체를 모두 가지고 있는, 다운로드를 마친 피어를 의미합니다.
> - Leecher : 파일의 일부분을 가지고 있는, 다운로드 중인 피어를 의미합니다.

기본적으로 진행되는 다운로딩 프로세스는 아래와 같습니다.
> 1. 새로 접속한 유저가 Tracker에게 다운로드 받으려는 파일, 그리고 자신이 연결 가능한 포트 등의 정보를 전송합니다.
> 2. Tracker는 위 요청에 대한 응답으로 동일한 파일을 다운로드 중인 피어들의 목록과, 그들에게 연결할 수 있는 방법에 대한 정보를 전송합니다. 이렇게 해서 묶이게 되는 피어 집단을 Swarm이라고 부릅니다.
> 3. P2P로 전송될 파일은 `512KB 혹은 256KB 조각`으로 분할되며, SHA-1 암호화를 통해 해싱됩니다. 해싱은 다운로드 이후에 Torrent File의 hashing information과 대조하여 조각이 깨졌다던지, 악성 코드 삽입 유무 등을 판별하는데 사용됩니다.
> 4. 조각이 다운로드 이후 검증이 완료되면, 피어는 Swarm내의 다른 피어들에게 해당 조각 업로드가 가능함을 알립니다.

__실제로 용량이 큰 데이터는 P2P로 전송이 이루어지고 Tracker는 그 사이에서 시그널링 작업만을 행하기 때문에, Tracker는 많은 컴퓨팅 파워를 필요로 하지 않습니다.__

## Piece Selection Algorithm
****
Swarm 내에서 다운로드 불가능한 조각이 남아있으면 안되고, 다운로드 속도를 최대한으로 끌어올려야 하기 때문에 각 피어들이 어떤 조각을 지금 바로바로 다운받아야 하는지는 매우 중요한 문제입니다.
이를 달성하기 위하여 몇 가지 작은 알고리즘들이 적용됩니다.

#### 0. Sub-pieces
BitTorrent는 기존의 512KB 혹은 256KB로 나눠놓은 조각을 `16KB로 더 잘게 Sub-piece`로 조각냅니다.
이 Sub-piece에 대한 다운로드 요청은 기본적으로 5개까지 Pipelining 가능하며, Sub-piece는 서로 다른 피어들에게 다운받아서 마지막에 결합하는 것이 가능합니다.

#### 1. Policy #1 : Strict Policy

#### 2. Policy #2 : Rarest First

#### 3. Policy #3 : Random First Piece

#### 4. Policy #4 : Endgame Mode

## Resource Allocation Algorithm
****

## ○ 참고문서
****
#### Resource Allocation Algorithm
* [파레토 최적](https://ko.wikipedia.org/wiki/%ED%8C%8C%EB%A0%88%ED%86%A0_%EC%B5%9C%EC%A0%81)
* [Choking과 조각, 피어 선택 알고리즘](http://thesoul214.tistory.com/158)
* [비트토렌트](https://ko.wikipedia.org/wiki/%EB%B9%84%ED%8A%B8%ED%86%A0%EB%A0%8C%ED%8A%B8)

#### 새로운 webCDN 아키텍처
* [BitTorrent](https://en.m.wikipedia.org/wiki/BitTorrent)
* [The World of P2P: BitTorrent Protocols and Software](https://en.wikibooks.org/wiki/The_World_of_Peer-to-Peer_(P2P)/Networks_and_Protocols/BitTorrent)
* [Peer-to-peer networking with BitTorrent](http://web.cs.ucla.edu/classes/cs217/05BitTorrent.pdf)