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
2.  Email 알림을 설정할 앱을 선택합니다.
3. App settings > Notifications > Manage notifications 을 클릭한다.
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-1.png)
4.  알림을 받을 Email을 입력하고, 배포 알림을 받을 대상 브랜치를 선택 후 Save한다.
   ![notification email](https://hkyeong0703.github.io/posts/images/2021-06-23-2.png)
5. Notifications 페이지로 돌아오면, **"We Sent you an email to confirm subscriptions, please click on the link in the email to start receiving notifications."** 라는 문구가 뜰 것이다. 4번 과정에서 입력한 Email에 메일 1통이 전송되었을 것이다.
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-3.png)
6.  Confirm subscription을 클릭한다. Email 인증 과정이라고 생각하면 된다.
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-4.png)
7. 그럼 아래와 같은 팝업창이 뜬다.
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-5.png) 
8. 다시 Amplify Console Notifications 창으로 돌아오면 Status가 Confirm으로 바뀐 것을 확인 할 수 있을 것이다. 
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-7.png)
9.  이제 만들어진 SNS를 통해 slack으로 알림을 보내는 작업을 진행해보자. 
   [Amazon SNS](https://console.aws.amazon.com/sns/v3/home) > Topics 에 접속하여, Amplify App ID로 검색한다. (App ID는 Amplify Console 앱에 들어갔을 때 url에서 알 수 있다. 또는 App ARN에서 알 수 있다.)
   ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-8.png)
10. 9번 과정에서 검색된 Topic을 선택 후,  클릭한다.
    ![notification](https://hkyeong0703.github.io/posts/images/2021-06-23-9.png)
11.  
12.   
13. 

