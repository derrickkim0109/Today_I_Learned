<h1><center> viewIsAppearing </center></h1>

###### tags: `💻 TIL`,`Swift`
###### date: `2023-12-26T17:21:33.284Z`

> [color=#724cd1][name=데릭]
> [viewIsAppearing Method](https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing)
> [WWDC23 What's new in UIKit](https://developer.apple.com/videos/play/wwdc2023/10055/)

# Overview

이전의 view Lifecycle에서는 없던 viewIsAppearing에 대해 학습해보자. WWDC23 What's new in UIKit에서 나오게 되었다.

> 시스템이 뷰 컨트롤러의 뷰를 계층구조에 추가하고 있음을 뷰컨트롤러에 알린다고 한다. 

그전에 아래의 그림을 보면 viewWillAppear(_:)이후에 호출되는 메서드인 것을 확인할 수 있다. 

## 이전 버전의 View Lifecyle

![스크린샷 2023-12-26 오후 1.03.38](https://hackmd.io/_uploads/SyVolCDPa.png)

> viewIsAppearing 메서드가 없던 이전 View Lifecycle
> 
**viewWillAppear**

- View가 계층구조에 추가되기 전에 호출된다.
    - Screen에 보이기 전에 호출
- View의 bounds가 정의되지만 위치는 정의되지 않은 상태이다.

**viewWillLayoutSubviews**

- bounds가 최종적으로 결정된다. 
- ViewController의 subview가 로드될 때마다 호출된다. 

**layoutSubviews**

- sub views를 배치한다.
- 직접 호출되지 않는다.

**viewDidLayoutSubviews**

- ViewController에게 subview들이 설정되었음을 알린다.
- ViewController의 subview가 로드될 때마다 호출된다. 

**viewDidAppear**

- View 계층에 추가되고, Screen에 보이고 난 후 호출된다. 


## viewIsAppearing이 추가된 View Lifecyle

시스템은 viewWillAppear 메서드를 호출 후 뷰 컨트롤러의 뷰가 나타날때 마다 이 메서드를 호출한다. viewWillAppear와 달리 시스템은 뷰 컨트롤러의 뷰를 뷰 계층 구조에 추가한 후 이 메서드를 호출하고 super view는 뷰 컨트롤러의 뷰를 배치한다. 
viewIsAppearing 메서드를 호출할 때까지, 뷰 컨트롤러와 해당 뷰는 업데이트된 trait 컬렉션을 받고 뷰는 정확한 위치 및 크기를 갖고 있다.

이 메서드를 재정의하여 뷰 표시와 관련된 custom 작업을 수행할 수 있다. 예를 들어, 이 메서드를 사용하여 뷰나 뷰 컨트롤러의 trait 컬렉션을 기반으로 뷰를 구성하거나 업데이트 할 수 있다. 또는, 스크롤 위치를 계산하는 데 뷰의 크기와 geometry가 필요한 경우, 뷰가 나타날 떄 선택된 셀이 보이도록 컬렉션 또는 테이블 뷰를 프로그래밍적으로 스크롤을 조정할 수 있다. 

**만약 재정의를 하려면 메서드 내부에 super를 호출해야한다.**

## Choosing the appropriate callback

viewWillAppear 메서드 후에 이 viewIsAppearing 메서드를 호출하지만 두 콜백은 모두 CATransaction 내에서 동일하게 호출된다. 즉, 두 메서드 중 하나에서 변경한 내용이 사용자에게 표시된다.

![스크린샷 2023-12-26 오전 11.40.33](https://hackmd.io/_uploads/B1iXa3PvT.png)

> viewIsAppearing이 추가된 View Lifecycle

Trait과 geometry는 viewWillAppear를 호출할 때는 최신 상태가 아니지만 viewIsAppearing을 호출할 때는 최신 상태이므로 이 메서드를 사용하여 뷰를 업데이트하라.

**viewWillAppear를 사용할 때**

- 애니메이션과 함께 추가하기 위해 transitionCoordinator에 접근할 때는 뷰의 전환이 시작되기 전에 콜백이 필요하다. 뷰 컨트롤러 전환 애니메이션과 동시에 수행되도록 프레임워크에 지시하는 애니메이션도 있습니다.

- 균형 잡힌 콜백이 필요한 이유는 뷰 컨트롤러나 뷰 특성, 계층 또는 geometry에 의존하지 않는 작업을 수행하기 위함이다. 이런 경우, viewWillAppear메서드에서 데이터베이스 Notification을 등록하고 viewDidDisappear메서드에서 등록을 해제할 수 있다. 

그 외의 다른 경우에는 viewIsAppearing 메서드를 사용하여 뷰를 업데이트 하라. 

![스크린샷 2023-12-26 오후 4.20.13](https://hackmd.io/_uploads/B1vnRxdPp.png)

viewWillLayoutSubviews, viewDidLayoutSubviews 같은 layout 메서드들은 layoutSubviews를 실행할 떄마다 여러 번 호출될 수 있다. 
하지만, viewIsAppearing 메서드는 뷰의 외형이 만들어지는 동안 한번만 호출하며, 뷰가 나타날 때 레이아웃이 필요하지 않아도 호출된다. 