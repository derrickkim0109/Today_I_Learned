<h1><center> What’s new in UIKit(1) </center></h1>

###### tags: `💻 WWDC 스터디`
###### date: `2023-12-27`

> [color=#724cd1][name=데릭]
> [WWDC23 What's new in UIKit](https://developer.apple.com/videos/play/wwdc2023/10055/)

> WWDC 2023 Session 중 하나인 `What’s new in UIKit`에 대해 알아보자

# 개요

> 오늘은 WWDC 2023에 새롭게 UIKit에 추가된 viewIsAppearing에 대해 공부했었다. 하지만, 조금 더 깊은 학습이 필요하다고 생각되어 WWDC를 보게 되었다. 

viewIsAppearing은 2023년도 부터 새롭게 추가된 메서드이다. viewWillAppear 메서드와 viewWillLayoutSubviews메서드 사이에 추가된 메서드이다. 자세한 내용에 대해 공부해 보자.

이번에 UIKit의 주요 아키텍처 개선 사항을 도입하여 앱을 더 쉽게 만들 수 있을 뿐만 아니라 SwiftUI와의 통합을 획기적으로 향상시켰다고 한다.

## Five key features

- Xcode previews
- View controller lifecycle updates
- Advances in the trait system
- Animated symbol images
- Empty states

### Xcode previews

> UIKit에서 Xcode previews를 직접 사용할 수 있다.

이 기능을 활용하려면 `#Preview`매크로를 사용하여Preview의 이름을 정하고, viewController를 반환하면 된다.

```swift 
class LibraryViewController: UIViewController {
    // ...
}

#Preview("Library") {
    let controller = LibraryViewController()
    controller.displayCuratedContent = true
    return controller
}
```

또한 뷰 컨트롤러가 없이 UIView만 볼 수도 있다. 

```swift 
class SlideshowView: UIView {
    // ...
}

#Preview("Memories") {
    let view = SlideshowView()
    view.title = "Memories"
    view.subtitle = "Highlights from the pas year"
    view.images = ...
    return view
}
```

Preview는 UI Components를 시각화하고 코드를 반복할 때 즉각적인 피드백을 제공하는데 도움이 된다.


### View controller lifecycle updates

ViewController를 present하거나 NavigationController를 통해 push or pop을 할 때와 같이 화면 전환 중에 ViewController의 Lifecylce에 대한 개선 사항을 다룬다. 


**New view Controller callback**

```
UIViewController.viewIsAppearing(_:)

Called between viewWillAppear(_:) and viewDidAppear(_:)

Best place to update UI when view appears
- View controller and view trait collections updated
- View added to hierachy, hac accurate geometry

Back-deploy to iOS 13

```

viewWillAppear 호출 이후, viewDidAppear 메서드가 호출되기 전에 viewIsAppearing이라는 새로운 ViewController Callback을 만들었다.

**viewIsAppearing**은 뷰가 나타날 때마다 작업을 수행하기 **best place**이다. 이 메서드가 호출되면, 뷰 컨트롤러와 뷰 모두 최신의 trait collection을 갖게 된다. 또한, 뷰가 계층 구조에 추가되어 있는 상태로 super View에 배치되어 있다.

이로 인해 viewIsAppearing은 size를 포함하여 뷰의 초기 geometry에 따라 달라지는 코드를 실행하기 이상적인 Callback이 된다.

마지막으로, viewIsAppearing 메서드는 iOS13에서부터 사용할 수 있다. 

![image](https://hackmd.io/_uploads/B1uWiWFvp.png)

뷰가 계층 구조에 추가되기 전과 레이아웃이 시작되기 전에 viewWillAppear가 어떻게 호출되는지 살펴보자. 

아직 계층구조에 추가되기 전이라, view의 size라던가 geometry, trait collection 등이 정확하지 않다. 

그래서, 여기서 trait collection을 사용하거나, 뷰의 size, geometry가 필요한 작업을 수행하기에는 너무 이른 시점이다.

![image](https://hackmd.io/_uploads/BylzsbtPa.png)

다음으로 애니메이션이 발생한 후 마지막에 별도의 CATransaction에서 viewDidAppear가 어떻게 호출되는지 확인하자.

viewDidAppear에서 변경한 사항은 transition이 완료될 때까지 보여지지 않으므로 transition 중에 무언가를 변경하고 보여지길 원하기에는 너무 늦다.

![image](https://hackmd.io/_uploads/Bk2ZibtDT.png)

![image](https://hackmd.io/_uploads/rkDfi-tDT.png)

반면에, viewIsAppearing은 viewWillAppear와 동일한 Transaction 내에서 호출된다. 즉, viewWillAppear, viewIsAppearing callback에서의 변경 사항들은 transition의 첫 번째 frame에서부터 사용자에게 보이게 된다는 의미이다.

```
This means any changes you make in either of those callbacks become visible to the user at the same time, right from the very first of the transition.
```

viewWillAppear와 viewIsAppearing의 타이밍은 비슷하지만, 주요 차이점이 있다.

![image](https://hackmd.io/_uploads/SJ5GjWKP6.png)

Layout callback들은 layoutSubviews를 실행할 때마다 생성된다. 이는 transition 중에 여러 번 발생할 수 있거나, 뷰가 보여지는 중에 언제든지 호출될 수 있다. 

**그러나!**
viewIsAppearing은 appearance transition 중에 한 번만 호출되며, 뷰에 레이아웃이 필요하지 않은 경우에도 호출된다.

### Trait system enhancements

- Define custom traits
- Easily change trait values in hierarchy
- Flexible trait change registration
- UIKit and SwiftUI interoperability

Trait은 앱의 계층 구조를 통해 데이터를 자동으로 전달한다. UITraitCollection에는 사용자 인터페이스 스타일, horizontal and vertical size class들, 선호하는 콘텐츠 사이즈 Category 같은 system trait이 포함되어 있다. 

이제 custom trait을 정의하여 UITraitCollection에 내 데이터를 추가할 수 있다. 이 방법은 앱 내의 뷰, 뷰 컨트럴러에 데이터를 전달하는 완전히 새로운 방식!이다.

또한, 뷰 또는 뷰 컨트롤러에서 Trait 값을 쉽게 수정할 수 있도록 override API를 추가했다고 한다. 

또한, 하위 클래스에서 traitCollectionDidChange를 재정의 할 필요 없이 Trait이 값을 변경할 때, 콜백을 수신하도록 하는 유연한 API를 채택할 수도 있다.

마지막으로, custom UIKit Trait을 custom SwiftUI Environment Key와 연결하여 앱의 UIKit과 SwiftUI Component간에 데이터를 원활하게 전달할 수 있다.

-> 새롭게 개선된 사항들을 더 확인하고 싶으면 `Unleash the UIKit trait system`을 확인하라.

### Animated symbol images

SF Symbol을 사용하면 toolbar icon, navigation bar, other UI Element를 일관되게 보여줄 수 있다. 

Text Label과 자동으로 정렬되도록 설계되었으며 앱 디자인에 맞게 weight, scale, appearance를 쉽게 설정할 수 있다.

![스크린샷 2023-12-27 오후 12.48.44](https://hackmd.io/_uploads/r1BsRMYD6.png)

iOS 17에서는 UIKit은 new Symbol effect API를 통해 애니메이션이 되는 Symbol 지원한다. 이런 Effect는 symbol Image, custom Image 둘다 적용할 수 있다.

**symbol effect**

```swift 

// Bounce the symbol once
imageView.addSymbolEffect(.bounce)

// Adding indefinite effects

// Add a variable color effect, which repeats
imageView.addSymbolEffect(.variableColor.iterative)

// Sometime later, remove the effect
imageView.removeSymbolEffect(ofType: .variableColor)

// Adding content transition effects

// Change the image, using Replace effect
imageView.setSymbolImage(
    pauseImage, 
    contentTransition: .replace.offUp
)

```

- New universal animations for all symbols
- Unified API across UI frameworks
- Composite layer annotations for custom sysmbols

-> 자세한 내용은 `Animate symbols in your app`을 확인하라.

## Empty states

Empty state는 앱에서 표시할 콘텐츠가 없는 순간을 말한다. 

![스크린샷 2023-12-27 오후 12.55.55](https://hackmd.io/_uploads/ryULemKwp.png)

`They tipically occur upon the first launch of your app, as no content has been created.`

Empty state는 보통 앱을 처음 실행할 때 발생하는데, 아직 콘텐츠가 생성되지 않았기 때문이다. 또한, 인터넷 연결이 되지 않는 제한 사항으로 인해 앱이 콘텐츠를 생성할 수 없는 경우에도 발생한다.


![스크린샷 2023-12-27 오후 1.00.40](https://hackmd.io/_uploads/BkVO-XKPa.png)

```swift 

// Represent no favorites empty state

var configuration = UIContentUnavailableConfiguration.empty()

configuration.image = UIImage(systemName: "star.fill")
configuration.text = "No Favorites"
configuration.secondaryText = "Your favorite translations will appear here."
viewController.contentUnavailbleConfiguration = configuration
```

UIContentUnavailableConfiguration은 Empty State에 대한 구성 가능한 설명이며, 이미지나 텍스트와 같은 placeholder 콘텐츠와 함께 제공될 수 있다.

![스크린샷 2023-12-27 오후 1.03.30](https://hackmd.io/_uploads/SkbmfXYvT.png)

```swift 

let configuration = UIContentUnavailableConfiguration.loading()
viewController.contentUnavailbleConfiguration = configuration

```

.empty() 외에도 UIKit은 준비 중인 콘텐츠를 나타내기 위해 .loading()을 제공한다.

![스크린샷 2023-12-27 오후 1.06.52](https://hackmd.io/_uploads/HycyQXFvp.png)

```swift

let cofiguration = UIHostingConfiguration { 
    VStack {
        ProgressView(value: progress)
        Text{ "Downloading file..." }
        .foregroundStyle(.secondary)
    }
}

viewController.contentUnavailbleConfiguration = configuration

```

또한, UIHostingConfiguration API를 활용하여 SwiftUI View로 앱의 Empty state를 나타낼 수도 있다.

> Cell에 UIHostingConfiguration을 사용하는 것과 동일하다.

viewController의 content unavailable configuration을 업데이트 하기 가장 좋은 장소는 새롭게 나온 메서드인 updateCOntentUnavailableConfiguration(using: state)이다. 

아래 예제는 결과를 반환하지 않는 쿼리를 위해 설계된 .search() Configuration을 사용한다.

![스크린샷 2023-12-27 오후 1.27.48](https://hackmd.io/_uploads/SJzAw7YDT.png)

```swift 

// Represent no search results empty state. 

override func updateContentUnavailableConfiguration(
    using state: UIContentUnavailableConfigurationState
) {
    var configuration: UIContentUnavailableConfiguration?
    
    if searchResults.isEmtpy {
        configuration = .search()
    }
    
    contentUnavailableConfiguration = configuration
}

// update search results for query

searchResults = backingStore.results(for: query)
setNeedsUpdateContentUnavailableConfiguration()

```

이 configuration은 Search Controller에서 삽입된 쿼리와 함께 localized된 primary와 secondary Text를 제공한다.

뷰 컨트롤러의 the available of content가 변경될 때마다 setNeedsUpdateContentUnavailableConfiguration을 호출하여 업데이트를 요청하라.