<h1><center> Solid Principle - OCP </center></h1>

###### tags: `TIL`,`SOLID`
###### date: `2023-12-2917:21:33.284Z`

> [color=#724cd1][name=데릭]
> [SOLID - 위키백과](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))

> [블로그] (https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-OCP-%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84-%EC%9B%90%EC%B9%99)

# 개요

오늘은 SOLID 원칙의 OCP(Open-Closed Principle)에 대해 학습해보자. 

## OCP(Open-Closed Principle)

개발 폐쇄 원칙(OCP, Open-Closed Principle)은 소프트웨어 객체(클래스, 모듈, 함수 등등)는 확장에 대해 열려 있어야(OPEN) 하고, 수정에 대해서는 닫혀 있어야(CLOSED) 한다는 원칙이다. 

여기서 확장은 새로운 기능이 추가되는 것을 의미한다. 

즉, 기능 추가를 할 상황일 때, 클래스를 확장을 통해 손쉽게 구현하면서, 확장에 따른 클래스 수정은 최소화 하도록 설계하는 것을 말한다. 

**확장에 열려있다**

- 새로운 코드를 추가해야 할때, 큰 힘을 들이지 않고 유연하게 코드를 추가해서 기능을 확장할 수 있다. 

**변경에 닫혀있다**

- 객체를 직접적으로 수정하는 것을 제한해야 한다는 의미이다. 
- 새로운 변경 사항이 발생했을 때 객체를 직접 수정하게 되면, 새로운 변경사항에 유연하게 대응할 수 없는 앱이라고 말한다. 
- 이는 유지보수의 비용 증가로 이어질 수 있다. 
- 따라서 객체를 직접 수정하지 않고도 변경사항을 적용할 수 있도록 설계해야 한다. 

```swift 

protocol Animal {
    func eat()
    func sleep()
}

class Dog: Animal {
    func eat() { }
    func sleep() { }
}

class Cat: Animal {
    func eat() { }
    func sleep() { }
}

class Monkey: Animal {
    func eat() { }
    func sleep() { }
}

```


즉, OCP는 다형성과 확장을 가능하게 하는 객체지향의 장점을 극대화하는 설계 원칙으로 객체를 추상화하여, 확장에 열려있고 변경에는 닫히는 유연한 구조를 만들어서 OCP를 적용할 수 있다. 

## OCP 원칙 위반 상황

```swift 

class Zoo { 
    func feed(_ animal: Animal) {
        if animal is Cat {
            print("Cat")
        } else if animal is Dog {
            print("Dog")
        }
    }
}

class Animal {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Cat: Animal {
        
}

class Dog: Animal {
    
}

let cat = Cat()
let dog = Dog()

let zoo = Zoo()

zoo.feed(cat)
zoo.feed(dog)

```

만약에 Horse 객체가 추가되면 feed 메서드는 어떻게 될까?

```swift 
    func feed(_ animal: Animal) {
        if animal is Cat {
            print("Cat")
        } else if animal is Dog {
            print("Dog")
        } else if animal is Horse {
            print("Horse")
        }
    }
```
이런식으로 추가해줘야하는 상황이 있을 수 있다. 이럴 때는, 상속, Protocol 두 가지를 통해 해결 할 수 있다.

### 상속

```swift 
class Zoo { 
    func feed(_ animal: Animal) {
        print("\(animal.name)")
    }
}

class Animal {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Cat: Animal {
    override init(name: String) {
          super.init(name: name)
    }
}

class Dog: Animal {
    override init(name: String) {
        super.init(name: name)
    }
}

let cat = Cat(name: "Cat")
let dog = Dog(name: "Dog")

let zoo = Zoo()

zoo.feed(cat)
zoo.feed(dog)

```

### Protocol 

```swift 
class Zoo { 
    func feed(_ animal: Animal) {
        print("\(animal.name)")
    }
}

protocol Animal {
    let name: String { get }
}

class Cat: Animal {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Dog: Animal {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

let cat = Cat(name: "Cat")
let dog = Dog(name: "Dog")

let zoo = Zoo()

zoo.feed(cat)
zoo.feed(dog)

```
