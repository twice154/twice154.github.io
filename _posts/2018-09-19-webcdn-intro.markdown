---
layout: post
title: "webCDN에 대한 간략한 소개"
subtitle: "webCDN에 대한 간략한 소개"
categories: essay
tags: webcdn
comments: true
---

[**webcdn**](https://twice154.github.io/tag/essay-webcdn/)태그에는 제가 현재 개발하고 있는 webCDN의 개발이야기에 대하여 써보려고 합니다.
블로그를 만들기 약 3주 전부터 개발을 진행하고 있던 터라 맨 처음 시작부터 하나하나 써나갈 수는 없지만,
지금부터 제가 개발하며 겪었던 이슈들에 대하여 옴니버스 형식으로, 독립적으로 이해할 수 있게 써나가려고 합니다.

저의 목표는 webCDN이 무엇인지 알지 못해도 그냥 제가 겪었던 이슈 하나하나에 대해서 독자 분들이 이해할 수 있도록 쓰는 것이지만,
그래도 태그명이 webcdn이니까 이게 뭘 하려고 만드는 것인지에 대하여 간단하게 첫 포스팅에 남기려고 합니다.

## CDN 이란?
****
CDN(Content Delivery Network)는 컨텐츠(이미지, 비디오 등)를 전송하도록 최적화 되어있는 전세계적으로 분산된 서버로 이루어진 네트워크를 말합니다.
그냥 인터넷 사업자의 서버 컴퓨터에서 컨텐츠를 전송해 주면 될 것 같은데, 왜 굳이 이런 시스템이 필요할까요?

[![CDN 개요](/assets/img/20180919/cdn.png)](#)

위의 그림

이 CDN 네트워크는 전세계 이곳저곳에 분산되어 있어서, 기본적으로 유저와 가장 가까운 지역에 있는 CDN 서버가 유저의 컨텐츠 요청에 응답을 하게 됩니다.

## webCDN 이란?
****
우리는 웹상에서 무수히 많은 이미지, 비디오를 접하게 됩니다.

## webCDN을 만들기 위하여 사용하고 있는 것들?
****
> Javascript
> Browser Javascript
> Node.js
> __WebRTC__

## ○ 참고문서
****
* [CDN이란 무엇입니까?](https://www.akamai.com/kr/ko/cdn/what-is-a-cdn.jsp)