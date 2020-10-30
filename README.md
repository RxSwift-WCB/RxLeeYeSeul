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
1. 
    * PublishSubject : 이름처럼 받은 정보를 구독자한테 뿌리는 친구
    * <String>이기 때문에 그 정보의 타입이 String이다.
2. 
    * subject에게 스트링이 담긴 next 이벤트를 준다.
    * 그러면 바로 구독자들한테 새로운 내용을 알려줘야하는데 전달할 구독자가 없다 -> 아무것도 출력되지 않음
3. 
    * subject.subscribe()를 통해 subscriptionOne이라는 구독을 생성한다. 그리고 발생하는 스트링을 담은 이벤트에 대해 print(string) 할 것이라고 정의해줬다
    * 그리고 subject에 이벤트를 준다 -> 출력된다!
    * observable은 sequential하다는 점을 기억해야함!! 구독 전의 이벤트는 알 수 없다.

### 2. Subject란?
* Subject는 observable과 observer 역할을 동시에 한다.
* 위의 코드와 같이 subject는 .next 이벤트 sequence를 받으면서 변경사항을 subscriber에게 바뀐 사항을 방출한다.
* RxSwift에는 4 가지 타입의 subject가 있다.
    * **PublishSubject** 빈 상태로 시작하여 새로운 값만을 subscriber에 방출한다. (앞에서 본 예제)
    * **BehaviorSubject** 하나의 초기값을 가진 상태로 시작하여, 새로운 subscriber에게 초기값 또는 최신값을 방출한다.
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
    print("3)", $0.element ?? $0) //모든 subject는 한번 중단되면 미래의 구독자에게 stop event를 재방출한다.
    }
    .disposed(by: disposeBag)
subject.onNext("?") //자동으로 dispose 되었으므로 구독자는 없고 아무런 출력이 나오지 않는다.
```

### 4. Behavior Subjects
* Publish Subject와 비슷하지만 Behavior Subjects는 새로운 구독자한테 가장 최근의 .next event를 방출한다.
![IMG_6B6D4B3B4003-1](https://user-images.githubusercontent.com/42545818/97677980-6ed97280-1ad6-11eb-805b-1e9d0a1bc9ea.jpeg)





