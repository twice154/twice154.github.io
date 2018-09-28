---
layout: post
title: "지금까지 개발해온 webCDN에서의 아키텍처 개선"
subtitle: "지금까지 개발해온 webCDN에서의 아키텍처 개선"
categories: essay
tags: webcdn
comments: true
---

간단하게 intro형식으로 [**webcdn**](https://twice154.github.io/tag/essay-webcdn/) 태그에 글을 하나 쓰기는 했지만, 개발진행하며 있었던 첫 번째 글이 아키텍처 개선에 대한 글이 되어버렸네요.

사실 제가 아키텍처라고 쓰고 있는 이게 아키텍처라고 부를 수 있는 건지도 잘은 모르겠습니다.
하지만 뭐, 제 나름대로는 아키텍처라고 생각을 하니까 이번 포스팅에서는 `아키텍처` 라고 쓰도록 하겠습니다 ㅎㅎ.

지금까지 했던 작업들을 다시 거의 무(無)의 상태로 돌아가서 아키텍처 부분부터 다시 생각하고 만들어나가야 한다니 정말 막막하기도 하지만, 블로그에 webCDN 개발기에 대해서 거의 처음부터 다시 쓸 수 있을 것 같아서 한 편으로는 그렇게 나쁘지 않은 것 같기도 합니다.

![webcdn-google-search](/assets/img/20180928/webcdn-google-search.png)
그나저나 블로그 검색엔진 최적화 같은 작업도 진행하지 않았는데, Google 검색에 걸리긴 하네요. webcdn이라는 검색어에 대한 글이 제 블로그 밖에 없나봅니다.

아무튼간에, `webCDN 아키텍처`를 왜 바꾸는지, 어떤 형태로 바꿀 생각인지에 대해서 이제 써나갑니다!

webCDN에 대해서 이게 뭔지 모르시는 분들은 [webCDN에 대한 간략한 소개](https://twice154.github.io/essay/2018/09/19/webcdn-intro/) 를 읽으시면 이게 뭔지 알 수 있으실 거에요.

## webCDN 아키텍처가 달성하고자 하는 목표
****
지금까지의 webCDN 아키텍처는 제가 나름대로 고민을 하면서, 아래 두 가지 문제점을 최대한 보완할 수 있는 형태를 갖추려고 노력했습니다.

> - CDN을 이용할 떄 보다 전송속도 측면에서는 성능이 떨어진다.
> - 전송속도 저하를 최소화하는 구현이 이루어지지 않는다면 UX에 치명적인 영향을 미칠 수 있다.

즉, 한마디로 요약해서 저의 webCDN 개발은 `작은 용량의 다량의 파일들을 최대한 빠른 속도로 P2P 전송 가능한 아키텍처`를 만들기 위한 여정이라고 할 수 있습니다.

이를 구현하기 위하여 제가 할 수 있는 일은 
> - 전송 작업이 일어나기 이전의 `오버헤드`를 최대한 줄인다.
> - 전송 속도를 컨트롤 할 수 없기 때문에, 고정되어있는 전송속도에서 최대한 `효율적`으로 전송 작업을 처리한다. 여기서 `효율적`이란 것은 UX에 영향을 최소화 하는 것을 말한다.
위와 같습니다.

## 기존의 webCDN 아키텍처
****
기존의 아키텍처에서 `전송 작업이 일어나기 이전의 오버헤드를 최대한 줄인다.` 라는 목표를 달성하기 위해서 저는 이런 방식의 접근을 취하였습니다.

1. Connection 연결의 오버헤드를 줄이기 위하여 최소 연결 피어 수를 정한다.

WebRTC에서 권장하는 1회 최대 전송량인 16KB를 준수하여 모든 Image파일을 16KB 크기의 Blob 분할합니다.
코드로 간단하게 표현해보면 아래와 같이 Recursive하게 호출해서 Blob을 만들게 되죠.

```js
const sliceFile = function(offset) {
    let reader = new window.FileReader()
    reader.onload = (function() {
        return function(e) {
            if(image.size > offset + e.target.result.byteLength) {
                // chunkSize가 16KB입니다.
                window.setTimeout(sliceFile, 0, offset + chunkSize)
            }
        }
    })(image)
    let slice = image.slice(offset, offset + chunkSize)
    reader.readAsArrayBuffer(slice)
}
sliceFile(0)
```

그 후에 요즘 우리나라 이통사들의 인터넷 요금제를 찾아봤습니다.

10Mb/s, 50Mb/s, 100Mb/s 세 개 정도가 대부분이더군요.

저 수치는 다운로드 속도 기준이니깐, 업로드 속도는 보통 다운로드 속도의 30% 정도로 망사업자들이 제약을 건다고 하네요.
실제로 네트워크 속도 측정을 해봐도 이와 비슷한 결과가 나옵니다.

따라서 최소 업로드 속도를 3Mb/s => 375KB/s 에 무선 라우터가 추가되었을 때 업로드 속도가 대략 50% 감소한다고 가정하여 약 `190KB/s` 로 잡았습니다.
아까 16KB 단위로 분할한 이미지 Blob을 초당 10개씩 전송할 수 있겠네요.

그렇다면 이제 피어 하나가 저정도 업로드 속도를 가지고 있을때, 기존의 CDN에서 이미지를 받아오는 속도랑 동일한 퍼포먼스를 내려면 `몇 명의 피어가` 동시에 전송을 해줘야 할지 알아봐야죠.

![naver-network](/assets/img/20180928/naver-network.png)
> [네이버 메인페이지 입니다.](www.naver.com)

![hsinven-network](/assets/img/20180928/hsinven-network.png)
> [하스스톤 인벤 이라는 커뮤니티 사이트의 게시글 입니다.](http://www.inven.co.kr/board/webzine/2097/1070483?iskin=hs)

네이버 메인페이지의 경우 계산을 해보면 약 `870KB/s` 의 속도로 다운로드를 진행했음을 알 수 있고, 하스스톤 인벤 게시글의 경우 약 `5900KB/s` 의 속도로 다운로드를 진행했음을 알 수 있습니다.
네이버의 경우보다 작은 커뮤니티 사이트가 다운로드 속도로 훨씬 큰 것은 아마도 네이버 메인페이지의 용량이 작기 때문에, 2.76s 미만으로 줄이는 것이 큰 의미가 없어 더 이상 Bandwidth를 할당하지 않았기 때문이 아닐까 싶습니다.

위의 결과를 통해 추측해 보았을 때, 용량이 작은 페이지의 경우에는 `약 5명의 동시 전송 피어`로 충분하고, 용량이 큰 페이지의 경우에는 `약 30명의 동시 전송 피어`가 필요하다는 것을 알 수 있습니다.

여기서 저의 아키텍처 설계의 실수가 있었습니다.

```js
{
    url1 : {
        fghuirwhg343g324g34 : {
            socketId : fghuirwhg343g324g34,
            downloaded : true,
            numOfCurrentUploadPeers : 1
        },
        f489hf3247g2hg8gw34f : {
            socketId : f489hf3247g2hg8gw34f,
            downloaded : false,
            numOfCurrentUploadPeers : 2
        },
        ...
    },
    url2 : {
    },
    ...
}
```

저는 피어를 매칭해주는 시그널링 서버에서 위와 같은 형태로 피어들을 저장해서 다운로드가 완료된 피어와 완료되지 않은 피어를 구분했습니다.
`다운로드가 완료된 피어(업로더)`는 다른 피어들에게 전송을 해줄 수 있고, `다운로드중인 피어(다운로더)`는 그냥 계속 다운로드만 합니다.

제가 이런 다운로더와 업로더를 명확하게 구분해 놓은 이유는 다음과 같습니다.
> - UX에 영향을 미치지 않기 위해서는 무조건 Top-down 형식으로 다운로드를 진행해야 한다. (화면 상단에 위치한 녀석일수록 먼저 다운로드 해야한다.)
> - 그런 경우에는 다운로더가 동시에 업로더의 역할까지 수행할 경우, 많은 중복이 발생할 수 있다. 예를 들자면, 나에게 전송을 해주는 다수의 업로더들의 다운로딩 상태가 나와 동일할 수 있다. 그럴 경우에 이 업로더들은 없는 업로더와 동일하게 취급되고, 다운로더의 다운로딩 속도가 저하될 수 밖에 없다.

## 새로운 webCDN 아키텍처
****


## ○ 참고문서
****
#### 기존의 webCDN 아키텍처
* [Using WebRTC data channels](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Using_data_channels#Understanding_message_size_limits)
## 새로운 webCDN 아키텍처
* [BitTorrent](https://en.m.wikipedia.org/wiki/BitTorrent)
* [The World of P2P: BitTorrent Protocols and Software](https://en.wikibooks.org/wiki/The_World_of_Peer-to-Peer_(P2P)/Networks_and_Protocols/BitTorrent)
* [Peer-to-peer networking with BitTorrent](http://web.cs.ucla.edu/classes/cs217/05BitTorrent.pdf)