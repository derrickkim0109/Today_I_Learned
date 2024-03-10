<h1><center> CALayer </center></h1>

###### tags: `TIL`,`SOLID`
###### date: `2023-12-3017:21:33.284Z`

> [color=#724cd1][name=데릭]
> [CALayer](https://developer.apple.com/documentation/quartzcore/calayer)

# 개요

Apple Documentation를 보고 CALayer에 대해 정리해보자

## CALayer

> 이미지 기반의 content를 관리하고 content에 애니메이션을 수행할 수 있는 객체이다. 

### Overview

Layer는 뷰에 대한 백업 store를 제공하는 데 자주 사용되지만, display하기 위한 뷰가 없이도 사용할 수 있다. 그리고 Layer의 main job은 제공하는 시각적인 content를 관리하는 것이지만, Layer 자체에는 background, color, border, shadow 속성이 있어 관리할 수 있다. 

그리고 시각적인 content를 관리하는 것 외에도, Layer는 content를 View에 표시하는 데 사용되는 content의 geometry(ex: position, size, transform)에 대한 정보도 유지한다. Layer의 프로퍼티들을 수정하는 것은 Layer의 content 혹은 geometry에서 애니메이션을 시작하는 방법이다. Layer 객체는 Layer의 타이밍 정보를 정의하는 데 CAMediaTiming Protocol을 채택하여 Layer와 애니메이션의 지속 시간과 속도를 캡슐화한다. 

Layer 객체가 뷰에 의해 생성된 경우, 일반적으로 뷰는 자동으로 Layer의 Delegate로 할당되므로 이 관계를 변경하면 안된다. 내가 직접 생성한 Layer의 경우, Delegate 객체를 할당하고 해당 객체를 사용하여 Layer의 content를 동적으로 제공하여 다른 작업을 수행할 수 있다. Layer에는 하위 뷰의 Layout을 별도로 관리하기 위해 LayoutManager 객체가 있을 수 있다.