<h1><center> Advances in diffable data sources </center></h1>

###### tags: `💻 WWDC 스터디`
> [color=#724cd1][name=데릭]
> [Advances in diffable data sources - wwdc](https://developer.apple.com/videos/play/wwdc2020/10045/)

> WWDC 2020 Session 중 하나인 `Advances in diffable data sources`에 대해 알아보자

# 개요

> Diffable Data Source에 대한 깊은 학습이 필요하다. 

 ![스크린샷 2023-12-20 21.03.05](https://hackmd.io/_uploads/B1HWdUlva.png)

-  첫 번째 섹션은 Horizontally scrolling grid이다. 

- 중간의 섹션은 iOS 14부터 새롭게 추가된 기능인데, 확장 및 축소가 가능한 outline-styled UI라고 한다. 

- 마지막 섹션은 UITableView처럼 보이는 UICollectionView이다. 

iOS 13에 도입된 Diffable Data Source는 새로운 snapshot 데이터 타입으로 UI 상태 관리를 단순하게 만들었다. 

### Snapshot

스냅샷은 고유한 섹션을 가지고 item의 identifier를 사용하여 UI State를 캡슐화한다.

따라서, UICollectionView를 업데이트할 때 먼저 새로운 스냅샷을 만들고 현재 보여지는 UI의 State를 채운 다음 Data Source에 적용한다. 

Diffable Data Source는 추가 작업 없이 자동으로 데이터의 차이를 계산하고 애니메이션을 적용한다. 

**WWDC 2019 Advances in UI Data Source**에서 이 API를 자세히 다루고 있다고 한다. 다음 주에 공부해보겠다. 

그래서 iOS 14부터 적용된게 무엇이냐면, 섹션 스냅샷과 first class Reordering(최고 수준의 재정렬..?) 지원이라는 두 가지 새로운 기능이 구축되었다고 한다. 

## Section Snapshot

위에서 iOS 14의 경우, Section Snapshot이라고 하는 기능이 추가되었다고 언급했는데..

Section Snapshot은 기존 스냅샷 바로 옆에 새로운 섹션 스냅샷을 추가하는 것이라고 한다. 

UICollectionView의 single snapshot에 대한 데이터를 캡슐화한다. (이름에서 알 수 있다는 표현이 있는데.. 나는 한국인이라 알기 힘들다.)

이렇게 향상 된 이유는 두 가지가 있다. 

1. Data Source를 section-sized chunk로 더 쉽게 구성할 수 있다고 한다.
2. iOS 14의 UI 렌더링을 지원하는 데 필요한 계층적 데이터 모델링을 허용하게 된다. 
    - 무슨말일까? 
    - 계층적 데이터 모델링은 주로 데이터를 계층적으로 구조화하는 방법이라고 한다. 

![스크린샷 2023-12-20 21.19.19](https://hackmd.io/_uploads/HkbRoIgD6.png)

첫 번째 Horizontally scrolling Section을 보면 single section snapshot을 사용하여 모델링을 한다. 

두 번째는 확장 됬다가 축소될 수 있는 기능이 있는 섹션인데 두 번째 섹션 스냅샷을 사용하여 이 계층적 데이터를 모델링 한다고 한다. 

그러니까 이 말은 extendable 케이스를 사용했다는 의미이지 않을까?

```swift 
enum Section {
    case horizontal
    case extendable
    case list
}
```

요약하자면 각각의 단일 섹션을 나타내는 총 세 개의 섹션 스냅샷으로 Diffable Data Source를 구성한다.

![스크린샷 2023-12-20 21.24.01](https://hackmd.io/_uploads/B1iyp8gP6.png)

> Section Snapshot API

섹션 스냅샷의 타입을 살펴보면 item 타입이 Generic인 것을 알 수 있다. Section의 Identifier 타입은 없다는 것을 알고 있자. Section Snapshot은 본질적으로 자신이 어떤 섹션을 나타내는지 알 수 없다고 한다. 

Section snapshot을 추가하기 위해서 `append`메서드를 사용하는데 매개변수 **parent**를 유심히 봐야한다.

parent를 통해 섹션 스냅샷에 필요한 데이터를 상위-하위 관계로 생성할 수 있다. 
만약 parent 매개변수를 사용하지 않으면 해당 Item을 Root로 적용한다는 의미이다. 


![스크린샷 2023-12-20 21.29.09](https://hackmd.io/_uploads/HkymCUew6.png)

또, 새로운 섹션 스냅샷을 위해 두 개의 새로운 API를 추가했는데 

apply 메서드는 섹션 스냅샷과 identifier를 가져오는 함수이다. 

snapshot 메서드는 특정 섹션의 콘텐츠를 나타내는 섹션 스냅샷을 검색할 수 있다고 한다.


![스크린샷 2023-12-20 21.31.56](https://hackmd.io/_uploads/H1DTCIeDp.png)

> 스냅샷 구현

![스크린샷 2023-12-20 21.33.30](https://hackmd.io/_uploads/HJL71Plv6.png)

코드에서 parent를 설정하여 상위-하위 관계를 구성한 것과 그렇지 않은 것을 확인할 수 있다. 


### Child snapshot

![스크린샷 2023-12-20 21.35.17](https://hackmd.io/_uploads/Skk9kvxwa.png)

이 코드는 Optional로 상위 Item을 가져오거나 상위 Item과 관련된 모든 하위 Item을 가져올 수 있다.

### Expanding and collapsing items

![스크린샷 2023-12-20 21.37.29](https://hackmd.io/_uploads/rJ7GgPeP6.png)

Expansion State는 섹션 스냅샷 State의 일부로 관리된다. 그리고 표시할 스냅샷을 생성할 때 해당 Item의 상위 Expansion State를 설정하여 하위 콘텐츠가 처음에 보여질지를 쉽게 결정할 수 있다.

또한, 스냅샷을 query하여 확장, 축소 여부를 확인할수도 있다. 

그리고 섹션 스냅샷의 Expansion State를 변경하는 경우, 실제 Diffable Data Source에 적용할 때까지 적용되지 않는다고 한다. 

만약 사용자가 outline disclosure 악세서리 같은 outline-styled UI와 상호 작용할 때 프레임워크는 자동으로 스냅샷을 새로운 Expansion State로 업데이트하고 해당 섹션 스냅샷을 Data Source에 적용한다. 

이런 확장 상태 변경에 대한 알림을 받는 것이 유용(?)한 경우가 많다고 한다. 

예를 들어, 특정 상위 Item이 절대 축소되지 않도록 요구하는 디자인이 있을 수 있다. 이런 경우를 지원하기 위해 또 새로운 API를 통해 code 방식으로 제어할 수 있다고 한다. 


![스크린샷 2023-12-20 21.45.10](https://hackmd.io/_uploads/BJf1MvevT.png)

옴마.. 이게 그 API이다.

위에서 말한 예제 상황을 처리하기 위해서 `shouldCollapseItem`을 사용하여 특정 상위 Item에 대해 false를 반환하면 된다. 

그리고 Lazy Loading도 지원하는데 snapshotForExpandingParent를 사용하면 된다. 

## Reordering Support

고유한 Item identifier를 사용하면 프레임워크가 사용자 상호 작용을 기반으로 재정렬할 변경 사항들을 자동으로 커밋할 수 있다.

근데 또 더 나아간 기술..?

### Optional closures

![스크린샷 2023-12-20 21.50.04](https://hackmd.io/_uploads/r1Ib7wgwT.png)

`canReorderItem` 클로저를 사용해서 Reordering을 활성화 시킬 수 있다.

클로저 내부에서 true가 반환되면 Reordering 상호작용이 시작될 수 있다. 

그리고 사용자가 Reordering 상호작용을 마치면 didReorder 클로저가 호출되어 새로운 정렬 State를 앱의 source of truth에 커밋할 수 있다고 한다. 

위에 언급한 canReorderItem와 didReorder 클로저를 모두 제공해야 Reordering 기능을 활성화 시킬 수 있다고 한다.(미리 말해주지..)


Transaction은 Diffable Data Source를 업데이트하는데 필요한 모든 정보를 제공한다.

![스크린샷 2023-12-20 21.55.06](https://hackmd.io/_uploads/ryIE4Dgvp.png)

ㅎㅎ..

**initailSnapshot**은 업데이트가 적용되기 전의 Diffable DataSource의 State라고 한다. 

그리고 **finalSnapshot**은 업데이트가 적용된 후의 State이다. 

스냅샷의 identifier로 App의 source of truth에 커밋해야하는 새로운 ordering을 결정할 수 있다. 

![스크린샷 2023-12-20 21.58.22](https://hackmd.io/_uploads/r1_eSPeDp.png)
