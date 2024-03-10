<h1><center> Solid Principle - DIP </center></h1>

###### tags: `💻TIL`,`SOLID`
###### date: `2024-01-3017:21:33.284Z`

> [color=#724cd1][name=데릭]
> [SOLID - 위키백과](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))

## DIP(Dependency Inversion principle

> 상위 모듈은 하위 모듈에 의존하면 안 되며, 두 모듈 모두 추상화에 의존해야 한다는 원칙이다.

1. 상위 모듈이 하위 모듈에 의존하지 않아야 한다.

- 전통적인 접근 방식에서는 상위 모듈이 하위 모듈에 직접 의존하여 결합도가 높은 코드를 구현하게 된다. DIP는 이 결합도가 높은 코드에서 발생할 수 있는 문제를 방지하기 위한 원칙이다. 결합도가 높은 경우, 변경이 발생할 때마다 상위 모듈도 변경해야 하는 문제가 있을 수 있다.

2. 둘 다 추상화에 의존해야 한다.

- 상위 모듈과 하위 모듈이 추상화에 의존해야 한다는 말은 protocol에 의존하는 것을 말한다. 이를 통해 변경이 발생했을 때, protocol만 업데이트하면 되므로, 상위 모듈과 하위 모듈 간의 결합도가 낮아지고 유연성과 확장성이 향상된다. 

### DIP를 위배한 코드

```swift 
// 하위 수준 모듈
class LightBulb {
    func turnOn() {
        print("LightBulb is ON")
    }
    
    func turnOff() {
        print("LightBulb is OFF")
    }
}

// 상위 수준 모듈
class Switch {
    private let bulb = LightBulb()
    
    func operate() {
        bulb.turnOn()
        // 또는 bulb.turnOff()
    }
}
```

### DIP를 적용한 코드

```swift 
// 추상화된 인터페이스
protocol Switchable {
    func turnOn()
    func turnOff()
}

// 하위 수준 모듈
class LightBulb: Switchable {
    func turnOn() {
        print("LightBulb is ON")
    }
    
    func turnOff() {
        print("LightBulb is OFF")
    }
}

// 상위 수준 모듈
class Switch {
    private let device: Switchable
    
    init(device: Switchable) {
        self.device = device
    }
    
    func operate() {
        device.turnOn()
        // 또는 device.turnOff()
    }
}
```