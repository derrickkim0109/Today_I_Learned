<h1><center> Modernizing Grand Central Dispatch Usage(9) </center></h1>

###### tags: `💻 WWDC 스터디`, `💻 TIL`, `GCD`
###### date: `2024-01-11T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Modernizing Grand Central Dispatch Usage - wwdc17](https://developer.apple.com/videos/play/wwdc2017/706/)

> WWDC 2017 Session 중 하나인 `Modernizing Grand Central Dispatch Usage`에 대해 알아보자

# 개요 

> 43:02부터 다시 시작이다. (8)에서는 No Mutation Past Activation에 대해 학습했다.

## Protecting the Target Queue Hierarchy

![스크린샷 2024-01-11 22.06.27](https://hackmd.io/_uploads/B1p0DwTda.png)

- Build your queue hierarchy bottom to top

- Opt into "static queue hierarchy"

![스크린샷 2024-01-11 22.07.24](https://hackmd.io/_uploads/BJLGdv6OT.png)
  
## Finding proble spots

![스크린샷 2024-01-11 22.11.13](https://hackmd.io/_uploads/B1igtD6_a.png)

re-target after activation event 

활성화 후 재타겟팅된 이벤트를 클릭했을 때

![스크린샷 2024-01-11 22.12.15](https://hackmd.io/_uploads/Hk9VKDaOp.png)

메서드를 두번 클릭하면 Instruments에서 문제가 발생한 코드를 직접 표시한다. 

![스크린샷 2024-01-11 22.12.54](https://hackmd.io/_uploads/Bklwtvp_p.png)

Xcode로 직접 이동할 수 있다.

## Summary

- Not going off-core is every more important
- Size your work appropriately
- Choose good granularity of concurrency 
- Modernize your GCD usage
- Use tools to find problem spots
