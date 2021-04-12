---
title: "Rest_api"
date: 2020-09-07T00:12:11+09:00
draft: true
---


참고 동영상: [https://youtu.be/RP_f5dMoHFc](https://youtu.be/RP_f5dMoHFc)

### REST를 구성하는 스타일
- client-server
- stateless
- cache
- uniform interface
    - 제약 조건
        - identification of resource → resource가 uri로 표현되면 된다.
        - mainpulation of resource through representations → resource를 변경할 때 http message에 표현을 담아서 처리하면 된다.
        - **self-descriptive messages**
            - 확장 가능한 커뮤니케이션이 가능하도록 함
        - **hypermedia as the engine of application state** (HATEOAS: application 상태는 Hyperlink를 이용해 전이되어야한다.)
            - application 상태 전이의 late binding, 어디서 어디로 전이가 가능한지 미리 결정되지 않는다. 어떤 상태로 전이가 완료되고 나서야 그 다음 전이될 수 있는 상태가 결정된다.
    - 왜 필요한가?
        - 서버와 클라이언트가 각각 독립적인 진화를 하기 위해
            - 서버의 기능이 변경되어도 클라이언트를 업데이트할 필요가 없다.
            - REST를 만들게 된 계기 "How do I improve HTTP without breaking the Web."
- layered system
- code-on-demand (optional)
