---
layout: post
title: "Javascript Callback, Promise에 관하여"
subtitle: "Javascript Callback, Promise에 관하여"
categories: dev
tags: javascript
comments: true
---

안녕하세요, [Javascript](https://twice154.github.io/tag/dev-javascript/) 태그의 첫번째 포스팅이 될 이번 포스팅에는 Javascript의 Callback과 Promise에 대한 글을 써보려 합니다.

요새 OnOwn이라는 프로젝트를 시작하며 정말 오랜만에 Node.js 서버 작성을 시작하였는데요, 이게 또 오랜만에 보다보니 `Callback이랑 Promise랑` 뭐가뭔지 엄청 헷갈리는 것 같습니다.
Javascript 처음 배울때도 헷갈렸는데, 아직까지도 그러길래 한 번 간단하게 정리하고 넘어가도록 할게요.

## Callback
****
Callback은 Javascript에서 함수의 파라미터로 함수를 전달하는 것을 의미합니다.
간단하게 이런 식이죠.

```js
function running(callback) {
    callback();
}
running(()=> {console.log("HI")}) // HI가 출력됩니다.
```

이는 C나 JAVA같은 언어에서는 불가능한 일인데요, 이는 Javascript에서 함수가 `일급 객체`로 취급되기 때문에 가능한 일입니다.
일급 객체는 함수형 프로그래밍에서 나온 단어인데요, 관심 있으신 분들은 부록에 제가 함수형 프로그래밍에 관한 링크를 올려놓을테니 가서 읽어보시면 좋을 것 같습니다.

`일급 객체`가 되려면 아래의 조건들을 만족해야 합니다.
> - 변수나 데이터 구조 안에 담을 수 있어야 한다.
> - 함수의 파라미터로 전달할 수 있어야 한다.
> - 리턴값으로 사용할 수 있다.
> - 런타임 환경에서 생성될 수 있다.

위의 조건 중 두 번째 조건을 Callback이라고 부르는 것입니다.

Node.js가 워낙 유명한 Javascript Runtime이고, 이를 통해서 서버를 작성하는 경우가 비일비재하기 때문에 맨날 서버만 보고있다보면 Callback이 다른 서버에 요청을 보내고, 그에 따른 응답을 받아서 실행하는 함수라고 착각을 하게 될 수도 있습니다.

하지만 명심하세요. Callback은 단지 함수의 파라미터로 함수를 전달하는 것 뿐입니다!

## Promise
****
Callback을 다른 서버에 요청을 보내고, 그에 따른 응답을 받아서 실행하는 용도로 사용하게 될 경우 다음과 같은 코드를 작성하게 됩니다.

```js
const request = require('request') // request라는 외부에 요청을 쉽게 보낼 수 있도록 해주는 모듈

request({
    url: 'https://api.forecast.io/forecast/4a04d1c42fd9d32c97a2c291a32d5e2d/39.9396284,-75.18663959999999',
    json: true
}, (error, response, body) => {
    if (error) {
        console.log('Unable to connect to Forecast.io server.')
    } else if (response.statusCode === 400) {
        console.log('Unable to fetch weather.')
    } else if (response.statusCode === 200) {
        console.log(body.currently.temperature)
    }
})
```

url로 요청을 보내고, 응답이 오던 에러가 나던 그 이후에 파라미터로 전달한 Callback이 실행되는 구조 입니다.
그런데 Callback에서 발생할 수 있는 모든 경우를 처리해줘야 하기 때문에 괜히 복잡해지게 됩니다.

그래서 Async한 프로그래밍을 더 쉽게 할 수 있도록 Promise가 ES6부터 도입되었는데요, 사용 방법은 매우 간단합니다.

```js
var somePromise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('Hey. It worked!')
        // reject('Unable to fulfill promise')
    }, 2500)
})

somePromise.then((message) => {
    console.log('Success: ', message) // Success: Hey. It worked! 가 출력됩니다.
}.catch((errorMessage) => {
    console.log('Error: ', errorMessage)
})
```

> - new Promise() 라는 코드를 통해 Promise를 생성합니다.
> - Promise안에는 resolve, reject가 포함된 callback을 정의합니다. resolve는 promise가 성공했을 때, reject는 promise가 실패했을 때 반환을 진행합니다.
> - 실제 실행부에서는 then함수를 통해서 resolve 값을 받고, catch를 통해서 reject 값을 받습니다.
> - resolve, reject는 단 하나의 값만 전달할 수 있습니다.

아래는 실제로 request를 이용해서 서버에 요청을 보내는 예시입니다.

```js
const request = require('request');

var geocodeAddress = (address) => {
    return new Promise((resolve, reject) => {
        request({
            url: `https://maps.googleapis.com/maps/api/geocode/`,
            json: true
        }, (error, response, body) => {
            if (error) {
                reject('Unable to connect to Google servers.')
            } else if (body.status === 'ZERO_RESULTS') {
                reject('Unable to find that address.')
            } else if (body.status === 'OK') {
                resolve({
                    address: body.results[0].formatted_address,
                    latitude: body.results[0].geometry.location.lat,
                    longitude: body.results[0].geometry.location.lng
                })
            }
        })
  })
}

geocodeAddress('00000').then((location) => {
    console.log(JSON.stringify(location, undefined, 2))
}.catch((errorMessage) => {
    console.log(errorMessage)
})
```

geocodeAddress라는 함수는 Promise를 리턴하고, 해당 Promise에서 request 모듈을 이용하여 어떤 서버에게 요청을 보냅니다.
그리고 각 조건에 따라 resolve, reject를 실행합니다.

마지막으로 Promise의 중첩입니다.

```js
var asyncAdd = (a, b) => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (typeof a === 'number' && typeof b === 'number') {
                resolve(a + b)
            } else {
                reject('Arguments must be numbers')
            }
        }, 1500)
    })
}

asyncAdd(5, '7').then((res) => {
    console.log('Result: ', res)
    return asyncAdd(res, 33)
}).then((res) => {
    console.log('Should be 45', res)
}).catch((errorMessage) => {
    console.log(errorMessage)
})
```

실제로 쓸모있는 코드는 아니고, 억지로 Promise를 사용하기 위하여 setTimeout을 통한 async-add 입니다.
실제 Promise 실행부분을 보시면, 첫 번째 resolve가 된 블럭에서 두 번째 Promise를 호출하는 것을 보실 수 있습니다.
그리고 그 뒤에 바로 두 번째 Promise에 대한 resolve 블럭이 추가되게 되고요.

여기서 특이한 점은 catch 블럭 하나에서 중첩된 Promise에서 발생하는 모든 에러를 처리한다는 점입니다. 명심하세요!

이것으로 Javascript Callback, Promise에 관하여 포스팅을 마치도록 하겠습니다.

읽어주셔서 감사합니다!

## ○ 참고문서
****
#### Callback
* [함수형 프로그래머가 되고 싶다고? 파트6](http://blog.jeonghwan.net/etc/2017/01/16/so-you-want-to-be-a-functional-programmer-part-6.html)