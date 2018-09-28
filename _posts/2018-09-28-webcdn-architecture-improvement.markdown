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
코드로 간단하게 표현해보면 아래와 같습니다.

<code>
const sliceFile = function(offset) {
    let reader = new window.FileReader()
    reader.onload = (function() {
        return function(e) {
            console.log("Sending image blob : end image false", e.target.result.byteLength)
            sendDataChannelList[pId].send(e.target.result)

            if(image.size > offset + e.target.result.byteLength) {
                window.setTimeout(sliceFile, 0, offset + chunkSize)
            } else {
                // image blob을 쪼개서 다 보낸 후, 다 전송되었다고 알리는 flag
                if(JSON.parse(event.data).num === imageBlobList.length - 1) {
                    console.log("Sending flag : end all image true", e.target.result.byteLength)
                    sendDataChannelList[pId].send("allImageDownloadEnded")
                } else {
                    console.log("Sending flag : end image true", e.target.result.byteLength)
                    sendDataChannelList[pId].send("thisImageDownloadEnded")
                }
            }
        }
    })(image)

    let slice = image.slice(offset, offset + chunkSize)
    reader.readAsArrayBuffer(slice)
}
sliceFile(0)
</code>

## 새로운 webCDN 아키텍처
****


## ○ 참고문서
****
#### 기존의 webCDN 아키텍처
* [Using WebRTC data channels](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Using_data_channels#Understanding_message_size_limits)