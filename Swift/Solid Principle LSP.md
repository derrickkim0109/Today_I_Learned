<h1><center> Solid Principle - LSP </center></h1>

###### tags: `TIL`,`SOLID`
###### date: `2024-01-3017:21:33.284Z`

> [color=#724cd1][name=데릭]
> [SOLID - 위키백과](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))
> [참신러닝](https://leechamin.tistory.com/518)

# 개요

이전 글에서 SRP, OCP에 대해 학습했다. 오늘은 LSP(Liskov substitution principle)에 대해 알아보자.

## LSP(리스코프 치환 원칙)

> 특정 객체를 하위 타입의 인스턴스로 변환해도 프로그램이 유지되야 한다는 걸 말한다. 

다시말해서, 컴퓨터 프로그램에서 S타입 객체가 T타입의 하위 타입이라면 T타입 객체를 S 객체로 교체해도 변경 없이 사용할 수 있어야 한다는 말이다. 

## 리스코프 치환 원칙을 위배한 경우

```swift 
class Rectangle {
    var width: Float = 0
    var height: Float = 0
    
    var area: Float {
        return width * height
    }
}

class Square: Rectangle {
    override var width: Float {
        didSet {
            height = width 
        }
    }
}

func printArea(of rectangle: Rectangle) {
    rectangle.height = 5
    rectangle.width = 2
    print(rectangle.area)
}

let rectangle = Rectangle()
printArea(of: rectangle) // 10

// -------------------------------

let square = Square()
printArea(of: square) // 4
```

하위 객체에서 상위 객체의 width값을 구할 수 없기 때문에 리스코프 치환 원칙을 위배한 것이라 할 수 있다. Rectangle를 사용하는 경우 width setter 때문에 의도치 않은 결과가 나온다. 

## 리스코프 치환 원칙 위배를 해결

Protocol을 사용하자.

```swift 
protocol Polygon {
    var area: Float: { get }    
}

class Rectangle: Polygon {
    private let width: Float
    private let height: Float
    
    init(
        width: Float,
        height: Float,
    ) {
        self.width = width
        self.height = height
    }
    
    var area: Float {
        return width * length
    }
}

class Square: Polygon {
    private let side: Float
    
    init(side: Float) {
        self.side = side
    }
    
    var area: Float {
        return pow(side, 2)
    }
}

func printArea(of polygon: Polygon) {
    print(polygon.area)
}

let rectangle = Rectangle(width: 2, length: 5)
printArea(of: rectangle) // 10

let square = Square(side: 2)
printArea(of: square) // 4
```

이렇게 protocol을 채택해서 각각 area를 구해서 해결할 수 있다.