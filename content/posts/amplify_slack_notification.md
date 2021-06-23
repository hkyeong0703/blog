---
title: "AWS Amplify Console 슬랙으로 배포 알림 받기"
date: 2021-06-23T20:31:09+09:00
description: ""
tags: [
	"aws",
	"amplify",
	"slack",
	"notification",
	"alarm",
	"deploy",
]
categories: []
disableComments: false
---



AWS Amplify Console에서 배포시 슬랙으로 알림을 받아보는 방법을 작성해보려고한다.

현재  [AWS Amplify Console User Guide](https://docs.aws.amazon.com/amplify/latest/userguide/notifications.html) 을 보면 이메일 알림을 기본적으로 제공하고, 이를통해 만들어진 SNS 주제를 통해 Slack과 같은 다른 도구에 알림을 보내는 데 활용할 수 있다.


## email notification 생성

1. [Amplify 콘솔을](https://console.aws.amazon.com/amplify/) 연다.
2. 이메일 알림을 설정할 앱을 선택합니다.
3. 
