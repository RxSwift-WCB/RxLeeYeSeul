## Ch3.Subjects - 예슬✨

### 1. 시작하기
```swift
public func example(of description: String,
                    action: () -> Void) {
  print("\n--- Example of:", description, "---")
  action()
}


example(of: "PublishSubject"){
    let subject = PublishSubject<String>() //Publish Subject : 수정사항 받으면 구독자들한테 열심히 알려주는 애들
    
    subject.onNext("Is anyone listening?") //받았다! 근데 구독자 없어서 아무일도 일어나지 않음.

    let subscriptionOne = subject.subscribe(onNext: { string in print(string)})
    //subject를 구독하는 구독자 생겼다! 그리고 발생하는 스트링 형태의 이벤트에 대해서 출력할거라고 정의해줬다.
    
    subject.on(.next("1")) // 1 출력됨
    subject.on(.next("2")) // 2 출력됨
    // observable은 철저히 sequential한 형태. 구독되기 전 이벤트는 알 수 없다.
    
    //.on(.next(_:))는 새로운 .next 이벤트를 subject에 삽입하고 이벤트 내의 값들을 파라미터로 통과시킨다.
    //.on(.next(_:)) 와 .onNext(_:) 는 똑같은 것~!
    
    

}
```
1) 
    * PublishSubject : 이름처럼 받은 정보를 구독자한테 뿌리는 친구
    * <String>이기 때문에 그 정보의 타입이 String이다.
2) 
    * subject에게 스트링이 담긴 next 이벤트를 준다.
    * 그러면 바로 구독자들한테 새로운 내용을 알려줘야하는데 전달할 구독자가 없다 -> 아무것도 출력되지 않음
3) 
    * subject.subscribe()를 통해 subscriptionOne이라는 구독을 생성한다. 그리고 발생하는 스트링을 담은 이벤트에 대해 print(string) 할 것이라고 정의해줬다
    * 그리고 subject에 이벤트를 준다 -> 출력된다!
    * observable은 sequential하다는 점을 기억해야함!! 구독 전의 이벤트는 알 수 없다.

### 2. Subject란?
* Subject는 observable과 observer 역할을 동시에 한다.
* 위의 코드와 같이 subject는 **.next 이벤트** sequence를 받으면서 변경사항을 subscriber에게 바뀐 사항을 방출한다.
* RxSwift에는 4 가지 타입의 subject가 있다.
	* **PublishSubject** 빈 상태로 시작하여 새로운 값만을 subscriber에 방출한다. (앞에서 본 예제)
	* **BehaviorSubject** 하나의 **초기값**을 가진 상태로 시작하여, 새로운 subscriber에게 **초기값 또는 최신값을 방출**한다.
	* **ReplaySubject** 버퍼를 두고 초기화하며, 버퍼 사이즈 만큼의 값들을 유지하면서 새로운 subscriber에게 방출한다.
	* **Variable** `BehaviorSubject`를 래핑하고, 현재의 값을 상태로 보존한다. 가장 최신/초기 값만을 새로운 subscriber에게 방출한다.

### 3. Publish Subjects
* 
![image](https://user-images.githubusercontent.com/42545818/97672009-ae02c600-1acc-11eb-8bb6-60a49e5332dc.png)
```swift
 let subscriptionTwo = subject.subscribe{ event in
        print("2)", event.element ?? event)
    }
    subject.onNext("3")
```
를 추가하면 어떻게 출력될까?
* subscriptionOne에 의해 3이 먼저 출력되고 ()
* subscriptionTwo에 의해 2)3이 출력된다.
* 여기에 subject.onNext("4")도 추가한다면? 어떤 결과가 나올 지 생각해봅시당.
* 위의 그림을 보면 알 수 있듯이 먼저 구독한 subscriptionOne에 의해 4 먼저 출력되고 2)4가 출력된다.

```swift
subject.onCompleted() //subject의 observable sequence를 중단한다.
subject.onNext("5") // 새로운 이벤트를 준다. 2)completed가 출력된다.
    
subscriptionTwo.dispose() //구독취소호
    
let disposeBag = DisposeBag()
    
subject
    .subscribe{
    print("3)", $0.element ?? $0) //모든 subject는 한번 중단되면 미래의 구독자에게 stop event를 재방출한다. 중요중요
    }
    .disposed(by: disposeBag)
subject.onNext("?") //자동으로 dispose 되었으므로 구독자는 없고 아무런 출력이 나오지 않는다.
```
* subject가 종료되었을 때에 존재하는 구독자에게만 종료 이벤트를 줄 뿐만 아니라 그 이후에 구독한 subscriber에게도 종료 이벤트를 알려주는 특성이 있다.

**어떨 때 쓸 수 있을까?**

* 시간에 민감한 데이터를 모델링할 때. (예. 실시간 경매 앱)
* (10:00 am이 경매시간이라고 가정하고,) 10:01am에 들어온 유저에게, 9:59am에 기존의 유저에게 날렸던 알람 "서두르세요. 경매가 1분 남았습니다." 을 계속 보내는 것은 아주 무의미하다.
---
### 4. Behavior Subjects
* Publish Subject와 비슷하지만 Behavior Subjects는 새로운 구독자한테 가장 최근의 .next event를 방출한다.
* 초기값을 갖는다.
![IMG_6B6D4B3B4003-1](https://user-images.githubusercontent.com/42545818/97677980-6ed97280-1ad6-11eb-805b-1e9d0a1bc9ea.jpeg)
* 첫번째 구독자가 구독을 시작했을 때 가장 최근의 1이라는 이벤트가 있으므로 첫번째 구독자에게 이벤트 1을 방출한다.
```swift
example(of: "Replay Subject"){
    // 1
    let subject = ReplaySubject<String>.create(bufferSize: 2) // .create로 사이즈가 2인 버퍼 생성!
    let disposeBag = DisposeBag()
    
    // 2
    subject.onNext("1")
    subject.onNext("2")
    subject.onNext("3")
    
    // 3
    subject
        .subscribe {
            print(label: "1)", event: $0) // 버퍼 사이즈가 2이므로 2와 3이 남아있다!
        }
        .disposed(by: disposeBag)
    
    subject
        .subscribe {
            print(label: "2)", event: $0) // 버퍼 사이즈가 2이므로 2와 3이 위의 구독으로 인해 남아있다!
        }
        .disposed(by: disposeBag)

    
    subject.onNext("4") // 구독자 1) 2)에 의해 각각 4 출력
    
    subject
        .subscribe{
            print(label: "3)",event: $0) //버퍼에 남아있는 3,4
        }
        .disposed(by: disposeBag)
}
```
**어떨 때 쓸 수 있을까?**
* `BehaviorSubject`는 뷰를 가장 최신의 데이터로 미리 채우기에 용이하다.
* 예를 들어, 유저 프로필 화면의 컨트롤을 `BehaviorSubject`에 바인드 할 수 있다. 이렇게 하면 앱이 새로운 데이터를 가져오는 동안 최신 값을 사용하여 화면을 미리 채워놓을 수 있다.

### 5. Replay Subjects
 * `ReplaySubject`는 생성시 선택한 특정 크기까지, 방출하는 최신 요소를 일시적으로 캐시하거나 버퍼한다. 그런 다음에 **해당 버퍼를 새 구독자에게 방출**한다.
 * 버퍼는 비어있을 수 있으므로 초기값이 반드시 있을 필요 없다.
 * **(주의)** `ReplaySubject`를 사용할 때 유념해야할 것이 있다. 바로 이러한 버퍼들은 메모리가 가지고 있다는 것이다.
	* 이미지나 array 같이 메모리를 크게 차지하는 값들을 큰 사이즈의 버퍼로 가지는 것은 메모리에 엄청난 부하를 준다.
 ![image](https://user-images.githubusercontent.com/42545818/97713953-deb22200-1b03-11eb-8dc8-45438d1dee99.png)
 ```swift
example(of: "Replay Subject"){
    // 1
    let subject = ReplaySubject<String>.create(bufferSize: 2) // .create로 사이즈가 2인 버퍼 생성!
    let disposeBag = DisposeBag()
    
    // 2
    subject.onNext("1")
    subject.onNext("2")
    subject.onNext("3")
    
    // 3
    subject
        .subscribe {
            print(label: "1)", event: $0) // 버퍼 사이즈가 2이므로 2와 3이 남아있다!
        }
        .disposed(by: disposeBag)
    
    subject
        .subscribe {
            print(label: "2)", event: $0) // 버퍼 사이즈가 2이므로 2와 3이 위의 구독으로 인해 남아있다!
        }
        .disposed(by: disposeBag)

    
    subject.onNext("4") // 구독자 1) 2)에 의해 각각 4 출력
    // subject.onError(MyError.anError) -> 3)까지 추가했을 때 출력 결과 1)4 2)4 1)anError 2)anError 3)3 3)4 3)anError
    subject
        .subscribe{
            print(label: "3)",event: $0) //버퍼에 남아있는 3,4
        }
        .disposed(by: disposeBag)
}
 ```
 ![image](https://user-images.githubusercontent.com/42545818/97719147-158b3680-1b0a-11eb-8186-654b7ce35c0b.png)
 * 3) 구독자를 추가하기 전에 subject.onError(MyError.anError)를 추가해보자
 ```swift
 subject.onNext("4") // 구독자 1) 2)에 의해 각각 4 출력
    subject.onError(MyError.anError) 
    subject
        .subscribe{
            print(label: "3)",event: $0) //출력 결과 1)4 2)4 1)anError 2)anError 3)3 3)4 3)anError
        }
        .disposed(by: disposeBag)
```
![image](https://user-images.githubusercontent.com/42545818/97721510-dc07fa80-1b0c-11eb-87fc-3fd503d4bd9d.png)

 * subject가 `error`를 통해 완전 종료되었음에도 불구하고 새 구독자`3)`에게 버퍼에 있는 값들을 보내주고 있다.
	* subject가 종료되었어도 버퍼는 여전히 돌아다니고 있기 때문에 이런 결과가 가능하다. 따라서 `error`를 추가한 다음에는 반드시 dispose를 하여 이벤트의 재방출을 막을 수 있다.

**어떨 때 쓸 수 있을까?**
* 만약에 `BehaviorSubject`처럼 최근의 값외에 더 많은 것을 보여주고 싶다면 어떻게 해야할까? 예를 들어 검색창같이, 최근 5개의 검색어를 보여주고 싶을 수 있다. 이럴 때 `ReplaySubject`를 사용할 수 있다.

### 6. Variables(deprecated)
* Variable은 Bahavior Subject의 특징을 갖고(it wraps a behavior subject) 현재 값을 state로 저장한다. 
* 현재값은 'value' 프로퍼티로 접근 할 수 있다.
* 새로운 element를 variable에 설정하는데도 쓸 수 있다. -> onNext를 사용할 필요가 없다.
* variable의 underlying behavior subject에 접근하려면 asObservable()을 호출한다.
```swift
example(of: "Variable") {
    // 1
    let variable = Variable("Initial value")
    let disposeBag = DisposeBag()
    // 2
    variable.value = "New initial value"
    // 3
    variable.asObservable()
        .subscribe {
            print(label: "1)", event: $0) //초기값 출력
        }
        .disposed(by: disposeBag)
    // 1
    variable.value = "1" //현재값 1 출력
    // 2
    variable.asObservable()
      .subscribe {
        print(label: "2)", event: $0) //이전값 1 출력
      }
      .disposed(by: disposeBag)
    // 3
    variable.value = "2" // 현재값 1)2 2)2 각각 출력
}
```

* value에는 .error나 .completed를 추가할 수 없다.
```swift
// These will all generate errors
variable.value.onError(MyError.anError)
variable.asObservable().onError(MyError.anError)
variable.value = MyError.anError
variable.value.onCompleted()
variable.asObservable().onCompleted()
```

### 7.Relays
    * PublishRelay
    * BehaviorRelay

* `PublishRelay` wraps `PublishSubject`
* `BehaviorRelay` wraps `BehaviorSubject`

```swift
example(of: "PublishRelay") {
  let relay = PublishRelay<String>()
  
  let disposeBag = DisposeBag()

  relay.accept("똑똑 미니 있나요?") //새로운 value 추가
  
  relay
  .subscribe(onNext: { // 구독자 생성, variable과 달리 asObservable 없이 바로 .scribe
    print($0)
  })
  .disposed(by: disposeBag)
  
  relay.accept("1") //1출력
}
```
* 마찬가지로 error나 completed event add 할 수 없음.



