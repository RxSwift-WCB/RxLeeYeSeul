## Ch.4 Observables and Subjuects in Practice - 예슬✨

### 1. ViewController에서 Variable 이용하기

> 다음 코드를 **MainViewController.swift**의 `MainViewController` 내에 입력합니다.

	```swift
	private let bag = DisposeBag()
    private let images = BehaviorRelay<[UIImage]>(value: [])
	```
	* 다른 클래스끼리 통신하지 않는다면 `private`로 정의할 것

* 뷰컨이 dispose bag을 소유하기 때문에, 뷰컨이 released 되자마자 모든 observable이 disposed 된다. (메모리 관리에 👍)
* 이것은 dispose bag에서 구독을 취소하는 것이 뷰컨에서도 deallocated 되도록 도와준다.
* 하지만 rootViewController 같은 것은 앱이 꺼질 때 까지 released 되지 않기 때문에 이 챕터 나중에 효율적인 Dispose에 대해 나옴

> actionAdd()에 다음을 추가한다.
```swift
let newImages = images.value //images의 현재값에 image 추가하고 그걸 accept를 통해 새로운 값으로 준다.
      + [UIImage(named: "IMG_1907.jpg")!]
    images.accept(newImages)
```

> actionClear()에 다음을 추가한다.
```swift
images.accept([])
```
> viewDidLoad에서 images의 구독을 생성한다. Relay이므로 바로 subscirbe할 수 있다.
```swift
images
  .subscribe(onNext: { [weak imagePreview] photos in
    guard let preview = imagePreview else { return }

    preview.image = photos.collage(size: preview.frame.size)
  })
  .disposed(by: bag)
```
* 이 챕터에선 구독을 viewDidLoad에서 하지만 나중에 클래스를 새로 생성하고 MVVM 아키텍쳐로 구조화할것이다.
* + 버튼을 누를때마다 사진이 하나씩 추가되는 것을 알 수 있다.

### 2. 복잡한 뷰컨 UI 다루기
* UI는 다음과 같은 방법으로 UX를 개선할 수 있다.
	* 만약 아직 아무 사진도 추가하지 않았거나, `Clear`버튼을 누른 직후라면, `Clear`이 작동하지 않게 할 수 있다.
	* 같은 상황에서 `Save` 버튼 역시 필요없다.
	* 빈 공간을 남기고 싶지 않다면, 홀수 개의 사진이 추가되었을 때 `Save` 버튼이 작동하지 않게 할 수 있다.
	* 사진을 6개까지만 추가하도록 제한할 수 있다.
	* ViewController가 현재 선택 개수를 보여줄 수 있다.
* 이걸 Reactive 하지 않은 기존의 방식으로 하려면 얼마나 긴 코드를 작성해야 할까요? 하지만 Rx에서는 매우 간단합니다.

> viewDidLoad()에 다음 코드를 추가하세요.
```swift
images
  .subscribe(onNext: { [weak self] photos in
      self?.updateUI(photos: photos)
  })
  .disposed(by: bag)
```
```swift
private func updateUI(photos: [UIImage]) {
  buttonSave.isEnabled = photos.count > 0 && photos.count % 2 == 0
  buttonClear.isEnabled = photos.count > 0
  itemAdd.isEnabled = photos.count < 6
  title = photos.count > 0 ? "\(photos.count) photos" : "Collage"
}
```
* 이 코드는 위에서 나열한 모든 개선을 반영한다. (헐..😳)
* 각각의 로직은 한줄로 표현되어있으며 이해하기 쉽다.
* 지금부터 Rx가 iOS 앱에 적용되었을 때 진짜 어떤점이 좋은지 알 수 있다.

### 3. Subject를 통해 다른 View Controller와 통신하기

* 여기서 할일은 유저가 카메라롤에 있는 임의의 사진을 선택할 수 있도록 `MainViewController`와 `PhotosViewController`를 연결하는 것이다.
* `PhotosViewController`로 push 하기 위해, 
> **MainViewController.swift** 내의 `actionAdd()`에 하단의 코드를 추가한다.기존에 입력했던 `IMG_1907.jpg` 만을 사용하게 하는 코드는 주석처리 한다.
```swift
let photosViewController = storyboard!.instantiateViewController(
  withIdentifier: "PhotosViewController") as! PhotosViewController

navigationController!.pushViewController(photosViewController, animated: true)
```
* 기존의 Cocoa 프레임워크를 다음에 해야할 일은 `photosViewController`의 사진들을 `mainViewController`로 서로 통신하기 위해 delegate 프로토콜을 쓰는 것일 것이다. 하지만 이건 매우 Rx 답지아나!
* RxSwift에서는 두개의 **어떠한** 클래스라도 연결할 수 있는 아주 universal 한 방법이 있다. 바로 `Observable`이다! 어떠한 프로토콜도 정의할 필요없다. 왜냐하면 `Observable`은 어떤 종류의 메시지라도 자신을 구독하는 Observer에게 전달할 수 있기 때문이다.

### 4. 선택한 사진에서 Observable 만들기

* 유저가 카메라롤에 있는 사진을 탭할 때마다 `.next` 이벤트를 방출하는 subject를 `PhotosViewController`에 만들 것이다.
> **PhotosViewController.swift**내에 `import RxSwift`를 하자.
`import RxSwift`

> 하단의 코드를 `PhotosViewController`에 추가한다.

```swift
	private let selectedPhotosSubject = PublishSubject<UIImage>()
	    var selectedPhotos:Observable<UIImage> {
	        return selectedPhotosSubject.asObservable()
	    }
```
* 선택된 사진을 방출할 private한 `PublishSubject`와 subject의 observable을 방출할 `selcectedPhotos` 프로퍼티를 만들었다.
	* 이 프로퍼티를 구독하는 것이 `MainViewController`에서 다른 간섭/변경 없이 사진 sequence를 관찰하는 방법이다.  

### 5. 선택한 사진들에 대한 Sequence 관찰하기
> *MainViewController.swift**로 돌아가서 `actionAdd()`내 navigation 관련 동작을 구현한 코드 다음에 다음과 같은 코드를 작성한다.
* 선택한 사진들에 대한 Sequence 관찰*을 할 수 있는 코드를 작성하는 것이다.


	```swift
	photosViewController.selectedPhotos
      .subscribe(
        onNext: { [weak self] newImage in
          
        },
        onDisposed: {
          print("Completed photo selection")
        }
      )
      .disposed(by: bag)
	```
> `onNext`클로져 안에 이미지를 추가하는 코드 생성
```swift
guard let images = self?.images else { return }
images.accept(images.value + [newImage])
```
### 6. 구독 취소하기(dispose)

* 여기까지보고 앱을 구동해보면 아주 잘 작동하는 것처럼 보이지만 한가지 간과한 것이 있다.
* 상기 코드를 보면 분명히 disposed 되었을 때 `"completed photo selection"` 메시지가 콘솔에 프린트되도록 해놓았다. 하지만 콘솔을 확인해보면 해당 메시지는 보이지 않는다. 이건 아직 해당 subject가 dispose 되지 않았다는 뜻이다.
* 당연하다. 왜냐하면 dispose bag 을 통해 dispose 되도록 명령해놓았고, `MainViewController`가 완전히 할당 해제 되어야만 dispose bag이 dispose 시킬 것이기 때문이다. 이 것이 싫다면 `.complated` 또는 `.error` 이벤트를 방출하므로써 완전 종료될 수 있을 것이다.
* 따라서, `PhotosViewController`가 사라질 때, 해당 이벤트를 방출하도록 하면 될 것이다. 
> 아래의 코드를 `PhotosViewController`의 `viewWillDisappear(_:)` 에 추가한다.

	```swift
	selectedPhotosSubject.onCompleted()
	```

### 7. 커스텀한 Observable 만들기
* 기존의 Apple API를 이용하면, `PHPhotoLibrary`에 대한 extension을 추가할 수 있을 것이다.
* 하지만 여기선 `PhotoWriter`라는 명칭의, 완전히 새로운 커스텀 클래스를 만들 것이다.
* 사진 저장을 쉽게 해줄 수 있는 `Observable`을 만들 것이다.
	* 이미지가 디스크에 성공적으로 읽혀졌다면 해당 이미지의 assetID를 방출하거나 `.completed` 또는 `.error` 이벤트를 방출할 수도 있을 것이다.

* **PhotoWriter.swift**를 열고 `import RxSwift` 한다.
* 다음의 코드를 작성한다.

	```swift
	    //1
	    static func save(_ image: UIImage) -> Observable<String> {
	        return Observable.create({ observer in

	            // 2
	            var savedAssetId: String?
	            PHPhotoLibrary.shared().performChanges({

	                // 3
	                let request = PHAssetChangeRequest.creationRequestForAsset(from: image)
	                savedAssetId = request.placeholderForCreatedAsset?.localIdentifier
	            }, completionHandler: { success, error in

	                // 4
	                DispatchQueue.main.async {
	                    if success, let id = savedAssetId {
	                        observer.onNext(id)
	                        observer.onCompleted()
	                    } else {
	                        observer.onError(error ?? Errors.couldNotSavePhoto)
	                    }
	                }
	            })

	            // 5
	            return Disposables.create()
	        })
	    }
	```
    * 주석을 따라 하나씩 살펴보자(사실 이해안됨)
		* 1) `save(_:)` 함수를 만든다. 해당 함수는 `Observable<String>`을 리턴할 것이다. 왜냐하면 사진을 저장한 다음에는 생성된 하나의 요소를 방출할 것이기 때문이다.
			* `Observable.create(_)`는 새로운 `Observable`을 생성할 것이기 때문에 어떤 Observable을 생성할 것인지를 이 클로저 내부에서 구현해야 한다.
		* 2) `performChanges(_:completionHandler:)`의 첫 번째 클로저 파라미터에서 제공된 이미지를 통해 콜라주된 사진을 생성할 것이다. 그리고 두 번째 클로저 파라미터에서 assetID 또는 `.error` 이벤트를 방출하게 될 것이다.
		* 3) `PHAssetChangeRequest.creationRequestForAsset(from:)`을 통해 새로운 사진세트를 만들 수 있고 이건 `savedAssetId`에 있는 해당 id로 저장할 것이다.
		* 4) 만약 성공 리스폰스를 받고 `savedAssetId`가 유효한 assetID 라면 `.next`와 `.completed` 이벤트를 방출할 것이다. 그렇지 않다면 `.error` 이벤트를 통해 에러를 방출할 것이다.
		* 5) `Disposible`이 리턴되도록 한다. (`.create`)의 리턴 값

### 8. RxSwift trait 연습하기

### 8-1. Single

<img src = "https://github.com/fimuxd/RxSwift/blob/master/Lectures/04_ObservablesAndSubjectsInPractice/4.%20single.png?raw=true" height = 100>

* Single은 `.success(Value)` 이벤트 또는 `.error` **이벤트를 한번만 방출**한다.
* `success` = `.next` + `.completed`
* 파일 저장, 파일 다운로드, 디스크에서 데이터 로딩 같이, 기본적으로 값을 산출하는 비동기적 모든 연산에도 유용하다.

#### 사용 예시
* `PhotoWriter.save(_)`에서 처럼, 정확히 한가지 요소만을 방출하는 연산자를 래핑할 때
	* **`Observable` 대신 `Single`을 생성**하여 `PhotoWriter`의 `save(_)` 메소드를 업데이트 할 수 있다.
* signle sequence가 둘 이상의 요소를 방출하는지 구독을 통해 확인하면 error가 방출될 수 있다.
	* 이 것은 아무 Observable에 `asSingle()`를 붙여 `Single`로 변환시켜서 확인할 수 있다.

### 8-2. Maybe

* `Maybe`는 `Single`과 비슷하지만 유일하게 **다른 점은 성공적으로 complete 되더라도 아무런 값을 방출하지 않을 수도 있다**는 것이다.

<img src = "https://github.com/fimuxd/RxSwift/blob/master/Lectures/04_ObservablesAndSubjectsInPractice/5.%20maybe.png?raw=true" height = 100>

* 사진을 가지고 있는 커스텀한 포토앨범앱이 있다. 그리고 그 앨범명은 UserDefaults에 저장될 것이고 해당 ID는 앨범을 열고 사진을 저장할 때마다 남을 것이다. 
* 이 때 `open(albumId:) -> Maybe<String>` 메소드를 통해 다음과 같은 상황을 관리할 수 있다.
	* 주어진 ID가 여전히 존재하는 경우, `.completed` 이벤트를 방출한다.
	* 유저가 앨범을 삭제하거나, 새로운 앨범을 생성하는 경우 `.next` 이벤트를 새로운 ID 값과 함께 방출시킨다. 이렇게함으로써 UserDefaults가 해당 값을 보존할 수 있도록.
	* 뭔가 잘못 되었거나 사진 라이브러리에 엑세스할 수 없는 경우, `.error` 이벤트를 방출한다.
* `asSingle`처럼, 어떤 Observable을 `Maybe`로 바꾸고 싶다면, `asMaybe()`를 쓸 수 있다.

### 8-3. Completable

* `Completable`은 `.completed` 또는 `.error(Error)`만을 방출한다.

<img src = "https://github.com/fimuxd/RxSwift/blob/master/Lectures/04_ObservablesAndSubjectsInPractice/6.%20completable.png?raw=true" height = 100>

* 하나 기억해야 할 것은, **observable을 completable로 바꿀 수 없다는 것**이다.
* observable이 값요소를 방출한 이상, 이 것을 completable로 바꿀 수는 없다.
* completable sequence를 생성하고 싶으면 `Completable.create({...})`을 통해 생성하는 수 밖에 없다. 이 코드는 다른 observable을 `create`를 이용하여 생성한 방식이랑 매우 유사하다.
* `Completeble`은 어떠한 값도 방출하지 않는다는 것을 기억해야 한다. 솔직히 이런게 왜 필요한가 싶을 것이다.
	* 하지만, 동기식 연산의 성공여부를 확인할 때 `completeble`은 아주 많이 쓰인다.
* 작업했던 `Combinestagram` 예제를 통해 생각해보자.
	* 유저가 작업할 동안 자동저장되는 기능을 만들고 싶다.
	* background queue에서 비동기적으로 작업한 다음에, 완료가되면 작은 노티를 띄우거나 저장 중 오류가 생기면  alert을 띄우고 싶다.
* 저장 로직을 `saveDocumet() -> Completable` 에 래핑했다고 가정해보자. 다음과 같이 표현할 수 있다.

	```swift
	saveDocument()
		.andThen(Observable.from(createMessage))
		.subscribe(onNext: { message in
			message.display()
		}, onError: { e in
			alert(e.localizedDescription)
		})
	```

	* `andThen` 연산자는 성공 이벤트에 대해 더 많은 completables이나 observables를 연결하고 최종 결과를 구독할 수 있게 합니다.

## G. 커스텀한 Observable 구독하기

* `PhotoWriter.save(_)` observable은 새로운 asset ID를 한번만 방출하거나 에러를 방출한다. 따라서 이건 아주 좋은 `Single` 케이스가 될 수 있다.
* **MainViewController.swift**를 열고 `actionSave()`에 아래의 코드를 추가한다. 이 것은 Save 버튼을 눌렀을 때 실행될 액션에 대한 것이다.

	```swift
	guard let image = imagePreview.image else { return }

	        PhotoWriter.save(image)
	            .asSingle()
	            .subscribe(onSuccess: { [weak self] id in
	                self?.showMessage("Saved with id: \(id)")
	                self?.actionClear()
	                } , onError: { [weak self] error in
	                    self?.showMessage("Error", description: error.localizedDescription)
	            })
	            .disposed(by: bag)
	```

	* 상기 코드는 현재 콜라주를 저장하기 위해 `PhotoWriter.save(image)`를 호출한 것이다.
	* 그런 다음에 구독이 하나의 요소를 받을 때, 리턴된 `Observable`을 `Single`로 전환한다.
	* 이 후 해당 메시지가 성공인지 에러인지를 표시한다.
	* 추가적으로, 만약 이미지가 성공적으로 저장되면 콜라주 화면을 클리어한다.