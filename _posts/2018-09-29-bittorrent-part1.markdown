---
layout: post
title: "Peer-to-peer networking with BitTorrent Part1"
subtitle: "Peer-to-peer networking with BitTorrent Part1"
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
Sub-piece를 다운로드 하기 시작하면, 해당 Sub-piece가 속해있는 조각을 가장 우선적으로 다운로드 합니다.

이는 기본적으로 조각을 기준으로 피어간의 전송 요청이 오가며 Sub-piece는 단지 전송 효율을 위하여 잘게 쪼갠 것이기 때문에, 최대한 빠르게 조각 하나를 완성하기 위함입니다.

#### 2. Policy #2 : Rarest First
피어가 다음 조각을 다운로드할 때, Swarm 내에서 가장 희귀한 조각을 다운로드 합니다.
이런 알고리즘이 필요한 이유는 아래와 같습니다.
> - Spreading the Seed
> - Increased Download Speed : 가장 희귀한 조각이라면 가장 다운로드 요청도 많을 것입니다. 이 많은 요청을 감당하기 위해서는 많은 피어가 해당 조각을 가지고 있어야 합니다.
> - Enabling Uploading
> - Most Common Last
> - Prevent Rarest Piece Missing : 해당 조각이 가장 희귀하다면 아마도 Seed만이 그것을 가지고 있을 가능성이 높습니다. seed는 또한 다운로드를 완료한 피어이기 때문에 Swarm을 떠날 가능성이 가장 높습니다. 따라서, Seed가 떠났을 때에도 해당 조각이 다운로드 가능하려면 최대한 많은 피어들에게 다운로드가 되어 있어야 합니다.

#### 3. Policy #3 : Random First Piece
새로운 피어는 가지고 있는 조각이 없기 때문에 단지 다운로드 뿐이 가능합니다.

이 새로운 피어를 업로드 작업에 최대한 빨리 참여시키기 위해서는 일단 랜덤하게 아무 조각이나 정해서 다운로드 하는 방법이 가장 효율적입니다.

__Rarest First__ 알고리즘의 경우에는 희귀한 조각을 찾기 위한 오버헤드가 발생하기 때문에 이 경우에서는 좋은 방법이 아닙니다.

#### 4. Policy #4 : Endgame Mode
마지막 조각을 업로드 속도가 느린 피어에게 다운로드 받게 되면 다운로드 완료시점이 더욱 늦춰지고, 이는 Leecher에서 Seed전환이 지연된다고 생각할 수 있습니다.

이를 해결하기 위하여 마지막 조각을 다운로드 할 때에는 모든 피어들에게 다운로드 요청을 보냅니다.

## Resource Allocation Algorithm
****
BitTorrent에서의 자원 배분은 기본적으로 게임이론의 `Tit-for-Tat` 전략을 따릅니다.
> 1. 우선은 협력적인 행위를 합니다.
> 2. 다음 행위에서는 상대방의 이전 행위에 부합하는 행위를 합니다.
> 3. 최종적으로 협력적인 네트워크를 구축해야 하기에, 한 번의 보복 행위 이후에 용서를 구할 준비가 되어 있어야 합니다.

너무 표현이 일반적인가요?
아래에 구체적인 예시들을 확인하면 위의 `Tit-for-Tat`이 무엇인지 이해하실 수 있을겁니다.

#### The Choking Algorithm
Choking이란 다른 피어에게 일시적으로 업로드는 중지하고, 다운로드만 계속하는 상태를 말합니다.
이런 Choking 행위를 하는 피어는 조별과제에서 무임승차하는 조원이랑 다를 게 없습니다.

그렇다면 BitTorrent는 이런 무임승차자를 걸러낼 방법이 있어야겠죠.
그 방법이 바로 Choking Algorithm 입니다.

기본적으로 하나의 피어는 다른 4명의 피어에게 Unchoking하게 되어 있습니다.

이 4명의 Unchoking 피어를 결정할 때, 20초 간의 다운로드 결과를 바탕으로 나에게 가장 많이 업로드해준 4개의 피어를 선정해서 Unchoking하게 됩니다.
또한 이 피어 선정과정은 10초에 한 번 꼴로 반복해서 실행됩니다.

이 알고리즘을 통해서 다운로드만 하고 업로드는 진행하지 않던 무임승차자는 오랜 시간이 지나지 않아서 배제됩니다.

#### Optimistic Unchoking
Bandwidth가 남는 피어에 한해서 더욱 많은 Unchoking을 행하는 것을 말합니다.

이는 30초마다 한 번씩 새로 결정됩니다.

#### Anti-Snubbing
피어가 순간적으로 현재 나에게 업로드를 해주고 있던 피어들에게 Choking을 당하는 경우가 발생할 수 있습니다.

여기에서 서로 다운로드, 업로드 상호작용을 하고 있는 피어들을 피어1, 피어2라고 하겠습니다.

피어2에게서 다운로드를 거부당하는 상황이 동일한 피어1에게서 계속 발생한다면, 더 이상 상대방(피어2)과 거래를 할 이유가 없습니다.

BitTorrent는 60초간 피어2에게서 아무런 데이터도 받지 못하면, `Tit-for-Tat`에 의하여 피어1이 피어2에게 업로드를 거부하도록 설계되었습니다.
그리고, 이 남는 대역폭을 Optimistic Unchoking으로 전환하여 최대한 빨리 새로운 피어를 찾기 위하여 전환하게 됩니다.

#### Upload Only
다운로드를 마친 Seed는 이제 언제 Swarm을 떠나더라도 이상하지 않습니다.

따라서 해당 Seed가 Swarm을 떠나도 조각들이 최대한 빠르게 전송, 복제될 수 있도록 Seed는 떠나기 직전까지 업로드 속도가 가장 빠른 피어를 찾아 조각들을 전송해주게 됩니다.

긴 글 읽어주셔서 감사합니다.

Part2 에서는 중앙에서 피어를 관리해주는 Tracker없이도 BitTorrent가 작동할 수 있도록 만들어주는 `Decentralized Tracker`에 관한 내용을 정리해서 포스팅 할 예정입니다.

오탈자라던지, 표현이 이상하다, 제 글에 틀린 부분이 있다 하는 부분들이 있다면 지적해주시면 감사히 수정하도록 하겠습니다!

## ○ 참고문서
****
#### Resource Allocation Algorithm
* [파레토 최적](https://ko.wikipedia.org/wiki/%ED%8C%8C%EB%A0%88%ED%86%A0_%EC%B5%9C%EC%A0%81)
* [Choking과 조각, 피어 선택 알고리즘](http://thesoul214.tistory.com/158)
* [비트토렌트](https://ko.wikipedia.org/wiki/%EB%B9%84%ED%8A%B8%ED%86%A0%EB%A0%8C%ED%8A%B8)