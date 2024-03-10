<h1><center> What’s new in UIKit(3) </center></h1>

###### tags: `💻 WWDC 스터디`
###### date: `2023-12-27T15:12:33.284Z`

> [color=#724cd1][name=데릭]
> [WWDC23 What's new in UIKit](https://developer.apple.com/videos/play/wwdc2023/10055/)

> WWDC 2023 Session 중 하나인 `What’s new in UIKit`에 대해 알아보자

# 개요

이전 [What’s new in UIKit(2)](https://hackmd.io/Bcn0jcbMR1mCddu9-MfMcg)와 달리 이번 시간은 `Improvements for iPad`에 대해 알아본다.
> 13:32부터 시작하는 내용이다.

## Improvements for iPad

- Window dragging interaction
- Enhanced sidebar behavior in Stage Manager
- Keyboard scrolling support
- Advancements in building document-centric apps
- The new Apple Pencil features

### Window dragging interaction

iOS 17에서는 dragging 동작이 시작될 수 있는 영역을 확장하여 Stage Manager의 window dragging 기능을 업데이트했다.

![스크린샷 2023-12-27 오후 3.19.53](https://hackmd.io/_uploads/rkrGfHFwa.png)

- Users can now pan on UINavigationBar
- Alternatively use UIWindowSceneDragInteraction
- Can set up gesture relationships
- Works with Mac Catalyst

이제 UINavigationBar 내부의 아무곳이나 드래그하면 Window가 이동된다. 이 제스처는 팬 또는 스와이프 제스처와 같이 앱에 있을 수 있는 다른 제스처 Recognizer와 잘 작동한다.

앱의 UINivigationBar를 사용하지 않는 경우 UIWindowSceneDragInteraction을 채택하여 모든 뷰에 추가할 수 있다.

충돌이 없는지 확인하기 위해 앱의 다른 팬 제스처와 제스처 관계를 설정할 수도 있다.

기본적으로 Mac Catalyst에서도 작동한다.

### Sidebar behavior in Stage Manager

Column-style UISplitViewController는 Stage Manager로 인해 새롭고 우아한 resizing behavior를 얻게 되었다고 한다..?

Sidebar는 필요한 경우 자동으로 숨겨지며, 특별히 표시하도록 요청할 때까지 숨겨진 상태를 유지한다.

만약에 사이드바가 더 좁은 width로 요청되면 UISplitViewController는 필요에 따라 overlay 또는 재배치 동작을 사용하게 된다.

![스크린샷 2023-12-27 20.13.53](https://hackmd.io/_uploads/r13lPYYDa.png)

Window의 크기가 더 커지더라도 overlay된 사이드바는 유지된다. 

![스크린샷 2023-12-27 20.14.45](https://hackmd.io/_uploads/rygVPKKPT.png)

더 넓은 너비로 해제했다가 다시 불러오면 tile로 다시 표시된다.

Mail(?)과 같은 Triple-column 분할 뷰 컨트롤러도 비슷하게 동작한다고 한다.

- New behavior for column-style UISplitViewController only
- Tiled sidebars

이 새로운 동작은 double-column or triple-column style로 생성된 UISplitViewController에서 발생한다. 

요약하면, 자동 behavior는 가능할 때마다 column을 tiling하고, 너비가 줄어들 때 필요에 따라 사이드바를 숨긴다.
사이드바 버튼을 탭했을 때 타일링한 공간이 충분하지 않은 경우, 보조 column을 사이드바로 overlay하거나 대체하며, PreferredDisplayMode 및 PreferredSplitBehavior를 사용하여 앱에서 동작을 재정의할 수 있다.

### Advancements in building document-centric apps

### Improvements for document-centric apps

- New UIDocumentViewController base class
    - Standard appearance for document apps
- UIDocument supports automatic renaming

UIKit은 콘텐츠 뷰 컨트롤러의 기본 클래스 역할을 하는 새로운 UIDocumentViewController를 제공한다. 
 system default experience를 보장하고 타이틀 메뉴 자동 구성, 공유, drag and drop, key commands 등과 같은 기능들을 추가 채택없이 제공한다. 
 
또한, UIDocument는 이제 UINavigationItemRenameDelegate를 준수하고 뷰 컨트롤러의 navigation item의 이름을 바꾸는 experience를 제공한다.

-> document management에 중점을 둔 세션은 `Build better document-based app`이니까 확인해라.


## Keyboard scrolling support

- UIScrollView can be scrolled using keyboards shortcuts
- Page Up, Page Down, Home, and End
- Enabled by default for iOS 17
- Can override behavior using `allowsKeyboardScrolling`

## General enhancements

- Collection view improvements
- Spring animation parameters
- Text interactions
- Default status bar style
- Drag and drop enhancements
- ISO HDR image support
- Page Control
- Paltte menus


### Collection view performance

![스크린샷 2023-12-27 20.31.38](https://hackmd.io/_uploads/rk4msFYvp.png)

이 그래프는 많은 수의 item으로 작업을 수행할 때 iOS 17에서 CollectionView가 얼마나 빠른지 보여준다.

- 애니메이션 없이 업데이트를 수행하면 성능이 더욱 향상된다.


#### Collection view layout enhancements

**Compositional Layout**

아래는 iPad Health app이다. favorite section은 2개의 item row를 가지는 compositional layout을 사용하고 있다. 그리고 NSCollectionLayoutDimension.estimated를 사용하여 자체 크기를 조정한다. Medication Cell의 높이가 옆에 있는 셀의 높이와 일치하지 않는 것을 확인해보자.

![스크린샷 2023-12-27 20.35.47](https://hackmd.io/_uploads/r1k7nFtwT.png)


- NSCollectionLayoutDimension.uniformAcrossSiblings(estimate:)
    - Work with self-sizing

NSCollectionLayoutDimension.uniformAcrossSiblings(estimate:)이 기능을 사용하면 레이아웃 self-sizing item이 가장 큰 크기를 가지느 item의 크기에 따라 일관된 크기를 가질 수 있게 한다.

위의 예제처럼 셀의 높이가 일지하지 않는 경우 estimated size를 uniformAcrossSiblings로 바꾸면 된다.

이 기능을 사용할 때는 가장 큰 item의 사이즈를 결정하기 위해 다른 모든 item들을 만든 후에 조정해야 한다는 것을 잊으면 안된다.

![스크린샷 2023-12-27 20.39.56](https://hackmd.io/_uploads/rkPfTKKD6.png)

## Spring animation parameters

![스크린샷 2023-12-27 20.40.26](https://hackmd.io/_uploads/HkNVpYYw6.png)

duration은 애니메이션이 완전히 완료되는 데 걸리는 시간이 아니라 Spring Animation이 안정화되는 것을 인식할 때까지의 시간을 정의하며 bounce와는 무관하다.

애니메이션의 바운스를 제어하는데 있어, zero에서 바운스를 증가시키면 애니메이션에 바운스 효과가 더해지지만, 이 과정에서 애니메이션의 총 지속 시간은 변하지 않습니다. 다시 말해, 애니메이션이 느껴지는 시간은 그대로 유지되면서 바운스 효과만 증가하는 것입니다

And increasing the bounce from zero adds bounce to the animation, without changing how long the animation feels.

```swift 

// Using the new UIView spring animation API

UIView.animate(springDuration: 0.5, bounce: 0.0) {
    circle.center.x += 100
}


// Optional이라서 animate만 사용함.
UIView.animate {
    circle.center.x += 100
}

```

-> `Animate with Springs`를 봐라.

## Text interactions

**Text cursor improvements**

![스크린샷 2023-12-27 20.48.01](https://hackmd.io/_uploads/Hynx19FwT.png)

- Accessories indicate input mode and dictation.

- Redesigned text loupe and selection handles
- Custom implementations can use system provided UI without UITextInteraction

- word processor와 같은 뷰를 만들 때 UITextInteraction 채택하지 않고도 system-provided view를 사용할 수 있다.


**Text item actions and menus**

![스크린샷 2023-12-27 20.50.50](https://hackmd.io/_uploads/rkPiJ9FDa.png)

- New API for text item interactions.
- Change the primary action or menu content.

![스크린샷 2023-12-27 20.51.30](https://hackmd.io/_uploads/H1h61qKPp.png)

- Tag custom ranges of text for interaction

Tag를 지정하여 interaction을 활성화할 수 있으며 Linkrk가 아닌 텍스트에 action이나 메뉴를 더 쉽게 추가할 수 있다.

-> "What's new with text and text interactions"를 보자.

## Default status bar style

```swift

// Default status bar style

override var preferredStatusBarStyle: UIStatusBarStyle {
    return .default
}

```

![스크린샷 2023-12-27 20.54.48](https://hackmd.io/_uploads/B1Q5x9tD6.png)

status bar style은 앱이나 뷰 컨트롤러의 dark or light mode에 따라 전환되는 기본 스타일이였다.

![스크린샷 2023-12-27 20.56.22](https://hackmd.io/_uploads/HJxgZ5twp.png)

iOS 17부터 default 스타일이 앱의 콘텐츠에 따라 지속적으로 조정되고 대비되는 것을 유지하기 위해 dark와 light 스타일을 자동으로 변경한다.


## Drag-and-drop enhancements

![스크린샷 2023-12-27 20.57.45](https://hackmd.io/_uploads/ryNSWcFv6.png)

![스크린샷 2023-12-27 20.58.12](https://hackmd.io/_uploads/rJgwZ9KDp.png)

- Launch apps by dropping supported content onto app icons.

- Specify CFBundleDocumentTypes in Info.plist

- Open via UIScene delegate callbacks

## ISO HDR image support

- Supported by
    - UIImageView
    - UIGraphicsImageRenderer
    - UIImageReader

-> "Upport HDR images in your app"

## Page control

- Represent fractional page progress
- UIPageControlProgress
- UIPageControlTimerProgress


```swift 

// Setting up a UIPageControlTimerProgress

let timerProgress = UIPageControlTimerProgress(preferredDuration: 10)

pageControl.progress = timerProgress

timerProgress.resumeTimer()

```

![스크린샷 2023-12-27 21.03.00](https://hackmd.io/_uploads/r1eKf5YD6.png)

무한 스크롤 된다..

## Palette menu

![스크린샷 2023-12-27 21.03.35](https://hackmd.io/_uploads/Bkg-ofqKP6.png)

![스크린샷 2023-12-27 21.03.46](https://hackmd.io/_uploads/HynszqFD6.png)

![스크린샷 2023-12-27 21.04.13](https://hackmd.io/_uploads/rkwaG9KPT.png)
