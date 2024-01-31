<h1><center> setNeedsLayout vs layoutIfNeeded </center></h1>

###### tags: `💻 TIL`
###### date: `2024-01-30T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [Corn 블로그 - setNeedsLayout vs layoutIfNeeded](https://baked-corn.tistory.com/105)
> [Demystifying iOS Layout](https://tech.gc.com/demystifying-ios-layout/)

> [layoutSubviews() - Apple Documentation](https://developer.apple.com/documentation/uikit/uiview/1622482-layoutsubviews)
> [setNeedsLayout() - Apple Documentation](https://developer.apple.com/documentation/uikit/uiview/1622601-setneedslayout)
> [layoutIfNeeded() - Apple Documentation](https://developer.apple.com/documentation/uikit/uiview/1622507-layoutifneeded)

# 개요 

> 최근에 한 프로젝트에서 애니메이션을 줄 때 제대로 알지 못하고 setNeedsLayout을 사용해서 뷰를 업데이트 해본 적이 있다. 그 때 제대로 알지 못했기 때문에 지금 학습하고자 한다. 

일단 두 개의 메서드를 비교하기에 앞서 `main run loop` 개념에 대해 알아야 한다. [Run Loop](https://hackmd.io/tev-lJIOTN6IMcRqYT4qyw)는 여기서 확인하자. 

## Main Run Loop 

Main Thread에서는 앱 시작 과정(UIApplication) 중에 자동으로 Run Loop가 실행된다. Main Run Loop는 터치 이벤트, 위치 변화, 디바이스 회전, 타이머 등의 이벤트들을 처리하게 된다. 이런 처리 과정은 각 이벤트에 적절한 핸들러를 찾아 처리 권한을 위임해서 진행하게 된다.

Main Run Loop에서 이벤트가 처리되는 과정에서 버튼을 누르거나 크기, 위치가 이동하는 애니메이션 실행으로 position의 값을 바꾸는 핸들러가 실행될 때도 있다. 

**하지만** 이런 변화는 즉각적으로 반영되는 것이 아니다. 

System은 layout or position이 바뀌는 뷰를 체크한다. 뷰가 갱신되는 시점은 모든 핸들러가 종료되고 Main Run Loop로 권한이 되돌아 오는 시점인 Update Cycle에서 된다.(Position, layout의 변화를 적용) 

**NOTE**
> Update Cycle: 발생한 이벤트들을 모두 처리하고 권한이 Main Run Loop로 돌아오게 되는 시점

![스크린샷 2024-01-30 21.34.31](https://hackmd.io/_uploads/S1M1avUqp.png)

다시 말해서, 뷰의 position or layoutd의 변화를 갱신하는 것은 시간차가 존재한다는 것이다.

우리가 개발할 때는 이런 시간차를 항상 고려해야 한다.

## layoutSubviews()

> 이 메서드가 실행되는 시점에서 하위 뷰를 배치하게 된다. 호출하면 해당 뷰의 모든 Subview들이 layoutSubviews를 연달아 호출한다. 시스템에 의해서 뷰의 값이 재계산되는 시점에 자동으로 호출되기 때문에 개발자가 따로 호출할 경우 비용이 많이 드는 메서드이다. 

개발자가 설정한 Contraints를 사용하여 하위 뷰의 size, position을 결정한다. 

이 메서드는 필요에 따라 하위 뷰의 레이아웃을 더 정확하게 조절하기 위해 필요한 경우 서브클래스에서 override할 수 있다. 

이 메서드를 override해야하는 이유는 하위 뷰의 autoresizing 및 constraint-based 동작을 제공하지 않을 때이다. 그리고 우리는 하위 뷰의 Frame 사각형을 직접 설정할 수도 있다.

**하지만**
이 메서드를 직접 호출해서는 안된다. 레이아웃을 강제적으로 업데이트하려면 drawing이 업데이트 되기 전에 setNeedsLayout()를 호출해야 한다. 

또는, 뷰의 레이아웃을 즉시 업데이트하려면 layoutIfNeeded()를 호출하라.

**NOTE**

> UIViewController 내의 뷰가 재계산되어 다시 그려지는 행위가 발생하는 과정은 layoutSubviews를 호출하고 하위 뷰가 배치되면 그 다음으로 viewDidLayoutSubviews가 호출된다. 그렇기 때문에 갱신된 View 값에 의존하는 행위들은 이 메서드에 명시해야 한다. <br>
EX) Layer 값은 자동으로 업데이트되지 않는다. 그래서 속한 뷰의 Frame이 변경되면, 이에 대응하여 `viewDidLayoutSubviews` 안에서 레이어의 frame을 수동으로 조정해야 한다. 

- View의 크기를 조절할 때
- Subview를 추가할 때
- 사용자가 UIScrollView를 스크롤할 때
- 디바이스를 회전시켰을 때 (Orientation)
- View의 AutoLayout 제약 조건을 변경시켰을 때

위에 언급된 시점들은 layoutSubviews를 자동으로 호출하기 위한 update cycle에서의 특정 지점들을 나타낸다. 

아래는 자동 예약이 아닌 수동으로 업데이트 주기를 예약하는 메서드들이다.

## setNeedsLayout()

> Receiver의 뷰를 무효화하고 다음 update cycle동안 레이아웃을 업데이트 하기 위해 트리거 하는 것이다. 

하위 뷰의 레이아웃을 조절하려면 이 메서드를 Main Thread에서 호출해야 한다. 이 메서드는 요청한 것을 기록하고 즉시 반환한다. 특징으로는 즉시 뷰를 갱신하지 않고 다음 Update cycle을 기다리기 때문에 업데이트되기 전에 setNeedsLayout을 사용하여 여러 뷰의 레이아웃의 무효화할 수 있다. 또한, 이 동작을 사용하면 모든 레이아웃의 업데이트를 하나의 update cycle로 통합할 수 있으므로 일반적으로 성능에 더 좋다. 

## layoutIfNeeded()

> 레이아웃의 업데이트가 보류 중인 경우, 하위 뷰를 즉시 갱신한다. 

이 메서드를 사용해서 뷰의 레이아웃을 즉시 업데이트할 수 있다. Auto layout을 사용할 때, 레이아웃 엔진은 제약 조건의 변경 사항을 충족하기 위해 필요에 따라 뷰의 위치를 업데이트 한다. 메세지를 받은 뷰를 루트 뷰로 사용하여 루트에서 시작하는 뷰 하위 트리를 배치한다. 보류 중인 레이아웃 업데이트가 없으면, 이 메서드는 레이아웃을 수정하거나 레이아웃 관련 콜백을 호출하지 않고 종료된다.

## 두 메서드를 호출한 경우

Main Run Loop에서 한 View가 `setNeedsLayout`을 호출하고 바로 다음에 `layoutIfNeeded`를 호출한다면, `layoutIfNeeded`는 즉시 View의 값을 재계산하고 뷰에 반영하게 된다. 이로 인해 `setNeedsLayout`이 예약한 `layoutSubviews` 메서드는 해당 update cycle에서 변경된 값이 없어 호출되지 않게 된다. 

이런 작동 방식으로 `layoutIfNeeded`는 즉시 값이 변경되어야 하는 애니메이션에서 널리 사용된다. 반면에 `setNeedsLayout`을 사용하면 애니메이션 블록에서 값이 즉시 변경되지 않고 나중의 업데이트 주기에서 값이 반영되므로 변경은 이루어지지만 애니메이션 효과가 눈에 띄지 않습니다. 