---
layout: post
title: "webCDN에 대한 간략한 소개"
subtitle: "webCDN에 대한 간략한 소개"
categories: essay
tags: webcdn
comments: true
---

[**webcdn**](https://twice154.github.io/tag/essay-webcdn/) 태그에는 제가 현재 개발하고 있는 webCDN의 개발이야기에 대하여 써보려고 합니다.

블로그를 만들기 약 3주 전부터 개발을 진행하고 있던 터라 맨 처음 시작부터 하나하나 써나갈 수는 없지만, 지금부터 제가 개발하며 겪었던 이슈들에 대하여 옴니버스 형식으로, 독립적으로 이해할 수 있게 써나가려고 합니다.

저의 목표는 webCDN이 무엇인지 알지 못해도 그냥 제가 겪었던 이슈 하나하나에 대해서 독자 분들이 이해할 수 있도록 쓰는 것이지만, 그래도 태그명이 webcdn이니까 이게 뭘 하려고 만드는 것인지에 대하여 간단하게 첫 포스팅에 남기려고 합니다.

## CDN 이란?
****
CDN(Content Delivery Network)는 컨텐츠(이미지, 비디오 등) 전송에 최적화 되어있는 전세계적으로 분산된 서버로 이루어진 네트워크를 말합니다.

그냥 인터넷 사업자의 서버에서 컨텐츠를 전송해 주면 될 것 같은데, 왜 굳이 이런 시스템이 필요할까요?

![CDN](/assets/img/20180919/cdn.png)

위의 그림 좌측의 아키텍처가 CDN을 사용하지 않은, 단일 서버에서 모든 유저의 컨텐츠 요청에 응답하는 경우입니다.
이는 수많은 컨텐츠 요청을 받고, 컨텐츠를 전송하는 서버에 막대한 트래픽 부하를 주게 됩니다.

이런 막대한 트래픽이 계속 발생하거나, 서버가 감당할 수 있는 한도를 넘어선 트래픽이 발생한다면, `결국 추후에 서버에 장애, 병목이 생길 확률이 증가`하게 됩니다.
또한, 요청을 보내는 유저의 물리적 위치가 컨텐츠를 전송해주는 서버와 매우 멀리 떨어져 있다면 컨텐츠 전송에 매우 긴 시간이 소요될 것이라고 예측할 수 있고, 이는 UX측면에서 큰 악영향을 끼치게 됩니다. 
많은 트래픽이 발생하는 서비스를 유지하기에는 `부적절한 아키텍처`라고 할 수 있습니다.

위의 그림 우측에 그려져 있는 CDN을 이용한다면, 이런 문제들을 해결할 수 있습니다.
CDN 네트워크는 `전세계 곳곳에 분산`되어 있어서, 기본적으로 유저와 가장 가까운 지역에 있는 CDN 서버가 유저의 컨텐츠 요청에 응답을 하게 됩니다.

CDN을 사용하면 위에 단일 서버에서 발생하는 문제들을 해결할 수 있습니다.
유저들의 컨텐츠 요청 및 전송을 분산해서 처리하기 때문에 각 서버의 트래픽 부하가 현저히 줄어들게 되어 `컨텐츠 전송 병목, 서버 장애 등의 문제가 발생할 확률이 훨씬 감소`합니다.
그리고 전세계에 분산되어 있는 CDN 서버 중에 유저와 가장 가까운 지역에 있는 서버가 유저의 요청에 응답하게 되어, `매우 빠른 컨텐츠 전송 속도`를 보장할 수 있습니다.

CDN에 대한 간략한 설명은 여기서 마치도록 하고, 더 많은 것들이 궁금하시다면 아래 `참고문서` 항목에 있는 링크들도 어렵지 않으니 읽어보시는 것도 좋을 것 같습니다.

## webCDN 이란?
****
자 이제 webCDN이 무엇인지에 대해서 설명드릴 차례가 된 것 같습니다.
webCDN은 이름 그대로 `web위에서 CDN`을 구현하고자 하는 것입니다.

제가 어떻게 수많은 인프라 비용이 들어가는 CDN을, 어떻게 그것도 웹브라우저에서 구현할 수 있을까요?
여기에서 CDN처럼 전세계에 분산된 네트워크를 갖추기 위한 하나의 트릭이 들어갑니다.

![torrent](/assets/img/20180919/torrent.png)
> 토렌트.. 다들 아시죠?

바로 `P2P(Peer to Peer)`를 사용하여 일반 유저들을 마치 CDN 서버 하나처럼 이용하는 것입니다.

즉, 어떤 웹페이지에 먼저 접속해서 해당 웹페이지의 컨텐츠(이미지, 비디오 등)를 이미 전송받은 유저들이 이후에 접속하는 유저들에게 컨텐츠를 전송해주는 방식입니다.

* webCDN을 사용했을 때의 __장점__ 이라면..
> - 데이터 트래픽의 대부분을 차지하는 비디오, 이미지의 상당부분을 P2P로 유저간에 전송이 이루어지기 때문에 인터넷 사업자 입장에서 트래픽 비용을 절감할 수 있다.
> - 이러한 인터넷 사업자의 비용 절감은 서비스 이용자들에게 더욱 좋은 서비스, 혜택으로 돌아갈 수 있다.

* webCDN을 사용했을 때의 __단점__ 이라면..
> - CDN을 이용할 떄 보다 전송속도 측면에서는 성능이 떨어진다.
> - 전송속도 저하를 최소화하는 구현이 이루어지지 않는다면 UX에 치명적인 영향을 미칠 수 있다.

현재 우선은 image만을 webCDN에서 처리할 수 있도록 구현중에 있습니다.

앞으로 간간히 제가 개발하며 막혔던 부분들, 고생했던 부분들을 한 편씩 칼럼으로 쓸 예정이니 재미있게 읽어주시면 감사하겠습니다!

## webCDN을 만들기 위하여 사용하고 있는 것들?
****
* Javascript
* Browser Javascript
* Node.js
* __WebRTC__

## ○ 참고문서
****
#### CDN 이란?
* [CDN이란 무엇입니까?](https://www.akamai.com/kr/ko/cdn/what-is-a-cdn.jsp)
* [CDN이란 무엇인가요?](https://cdn.hosting.kr/cdn%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94/)