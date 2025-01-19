- 문서: [Foundation - NotificationCenter](https://developer.apple.com/documentation/foundation/notificationcenter)
- 개요: 등록된 Observer들에게 정보를 브로드캐스트할 수 있는 도구. 앱 내에서의 전달에 한하며, 프로세스간 통신을 위해서는 [DistributedNotificationCenter](https://developer.apple.com/documentation/foundation/distributednotificationcenter)를 활용하라고 한다.

## 방법
1. 유니크한 키를 정하고, 옵저버를 추가한다. (`NotificationCenter::addObserver(_:selector:name:object:)`)
	- 기본적으로 NotificationCenter.default가 전역에 제공된다.
	- selector에 알림을 받을 핸들러 메서드를 지정해준다. 아무 인자도 받지 않거나, Notification 객체 한 개를 인자로 받는 메서드여야 한다. 이 조건을 지키지 않으면 런타임에서 NSInvalidArgumentException 예외가 발생한다.
	- 핸들러에서 Notification 객체를 인자로 받는 경우, 알림의 키(`name`)와 발행자(`object`), 추가 정보(`userInfo`)를 해당 객체로부터 참조할 수 있다.
	- 아래에서 간단한 클래스를 작성해보았다.
	```swift
	final class Listener {
	    let id: Int
	    
	    init(_ id: Int) {
	        self.id = id
	        
	        let nc = NotificationCenter.default
	        nc.addObserver(self, selector: #selector(handler), name: Notification.Name("SomeNotification"), object: nil)
	    }
	    
	    @objc func handler(_ notification: Notification) {
	        if let userInfo = notification.userInfo,
	           let contents = userInfo["contents"] as? String {
	            print("[Listener#\(id)] 알림을 수신했습니다. name: \(notification.name) contents: \(contents)")
	        }
	    }
	}
	```

2. 같은 키를 이용해 알림을 발송한다. (`NotificationCenter::post(name:object:userInfo:)`)
	- name: 알림의 키
	- object: 알림을 보낸 객체. nil을 지정해도 된다.
	- userInfo: 알림에 실어보낼 추가 정보. `[AnyHashable : Any]?` 타입이다. 즉 nil을 지정해도 된다.
	```swift
	final class Publisher {
	    func post(_ contents: String) {
	        let nc = NotificationCenter.default
	        nc.post(name: Notification.Name("SomeNotification"), object: self, userInfo: ["contents": contents])
	    }
	}
	
	let listener1 = Listener(1)
	let listener2 = Listener(2)
	
	let publisher = Publisher()
	publisher.post("첫번쨰 알림")
	publisher.post("두번쨰 알림")
	publisher.post("세번쨰 알림")
	```

	![](Pasted%20image%2020250119132207.png)

## 활용
- 반응형 프레임워크를 사용할 떄와 달리 알림을 보내는 쪽과 받는 쪽이 서로를 몰라도 된다. 대신 NotificationCenter.default를 참조한다.
- 알림을 불특정 다수의 객체가 받아야하는 경우 유용할 것 같다. (로그인 등)
- 또한 앱 라이프사이클 변경에 따른 알림을 받는다든지 시스템에서 미리 제공하는 알림을 받을 수도 있다.
	- ex> https://www.hackingwithswift.com/example-code/system/how-to-detect-when-your-app-moves-to-the-background

## 참고
- addObserver로 등록한 옵저버를 나중에 수동으로 제거해줘야 하는가?
	- 답: 최신 OS를 타겟으로 한 앱이라면 시스템이 자동으로 관리해준다.
		- "If your app targets iOS 9.0 and later or macOS 10.11 and later, and you used [`addObserver(_:selector:name:object:)`](https://developer.apple.com/documentation/foundation/notificationcenter/1415360-addobserver) to create your observer, you do not need to unregister the observer. If you forget or are unable to remove the observer, the system cleans up the next time it would have posted to it."
		- 출처: [removeObserver - Discussion](https://developer.apple.com/documentation/foundation/notificationcenter/1407263-removeobserver#discussion)

## 참고자료
https://www.hackingwithswift.com/example-code/system/how-to-post-messages-using-notificationcenter
https://medium.com/@nimjea/notificationcenter-in-swift-104b31f59772

