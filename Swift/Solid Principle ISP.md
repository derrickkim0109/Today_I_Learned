<h1><center> Solid Principle - ISP </center></h1>

###### tags: `TIL`,`SOLID`
###### date: `2024-01-3017:21:33.284Z`

> [color=#724cd1][name=데릭]
> [SOLID - 위키백과](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%ED%96%A5_%EC%84%A4%EA%B3%84))

## ISP(Interface segregation principle)

클라이언트에서 Interface를 채택할 때, 불필요한 메서드가 있으면 안된다는 원칙이다. 즉, 작은 단위들로 분리시킴으로써 클라이언트가 꼭 필요한 메서드들만 이용할 수 있게 설계해야 한다는 말이다. 

```swift 
protocol PhotoRepositoryInterface {
    func fetchAlbum() -> [AlbumInfoEntity]
    func save(_ photo: PhotoInfoEntity)
    func fetchCameraAuthorization()
}
```
PhotoRepositoryInterface에 앨범 리스트를 호출, 사진을 저장, 카메라의 권한을 요청하는 메서드 3 개가 있다고 가정해보자. 

```swift 
final class PhotoRepository: PhotoRepositoryInterface {
        func fetchAlbum() -> [AlbumInfoEntity] {

        }
    
    func save(_ photo: PhotoInfoEntity) {
    }
    
    func fetchCameraAuthorization() {
    }
}
```
PhotoRepository는 사진과 관련이 있는 기능이 들어가야 한다는 것을 알 수 있다. 각 메서드들을 보면 save메서드만이 Photo와 연관지을 수 있을 것이다.

그래서 아래와 같이 수정해봤다.

```swift 
protocol AlbumRepositoryInterface {
    func fetchAlbum() -> [AlbumInfoEntity]
}

protocol PhotoSaveInterface {
    func save(_ photo: PhotoInfoEntity)
}

protocol CameraAuthorizationInterface {
    func fetchCameraAuthorization()
}
```

이렇게 하면 코드의 유연성과 확장성이 향상되고, 클라이언트는 필요한 기능만 사용할 수 있다.