## 개요

현재 진행 중인 개인프로젝트를 출시하고 보니 불편한 점이 있다

1. 뷰 컨트롤러가 너무 커졌다.
    - 며칠만 코드를 안봐도 다시 볼 때 파악할 시간이 좀 필요하다.
	- Extension으로 나누어 뷰 컨트롤러를 수평적으로 분할하면서 짰지만, 오히려 하나의 컴포넌트에 대한 코드가 여러 파일로 쪼개지다 보니 수정해야할 메서드가 어디에 있는지 찾는데 시간이 더 걸린다.
2. 이런 일이 있으니 아키텍쳐 레벨에서의 개선 필요성을 많이 느꼈다. 프로젝트는 점점 더 확장될 것이므로, 레이어를 나누어야 위와 같은 불편이 줄어들 것이다.
3. 막상 무턱대고 MVVM이니 클린 아키텍쳐니 도입하려 보니, 사소한 변경에도 기존 기능에 장애가 생겼던 이전의 몇몇 케이스가 떠올랐다.
4. 장대한 욕심의 시작에는 테스트 코드가 필요하다. 리팩토링하고, 테스트하고, 안심하고, ... 적어도 이 사이클을 자신있게 돌릴 수 있어야 시작할 수 있다.
5. 리팩토링을 위한 테스트이므로 통합 테스트가 아니라 핵심 컴포넌트에 대한 단위 테스트가 필요하다. 그러나 지금 구현 상, 각종 컴포넌트끼리의 결합도가 매우 강해서 바로 테스트를 작성할 수가 없다.
	- 이런 코드 `let component = Component.shared`가 너무 많다.
6. 컴포넌트끼리의 결합도를 낮춰야 한다. DI가 필요하다.

View - ViewController - Repository - ...  
각 컴포넌트가 직접 의존 컴포넌트의 객체를 생성해 소유하는 방식은 단순하지만 결합도를 높인다. 이런 의존 관계를 느슨해지도록 끊으려면 외부에서 의존하는 컴포넌트를 주입받도록 리팩토링하면 된다. (Dependency Injection)

당연히 주입은 외부에서 누군가가 해줘야 한다. 일반적으로 여러 언어 진영과는 상관없이 다양한 프레임워크나 DI 라이브러리에는 공통적으로 Container가 있다. 이 Container에 각종 컴포넌트의 객체를 생성해 모아두고, 여기서에서 원하는 객체에 가지고 있는 다른 객체를 주입해준다.  
우리가 작성해둔 코드는 의존성 해결에 대한 책임에서 손을 떼고, 외부의 컨테이너가 이 책임을 대신하는 것이다. (Inversion of control)

백엔드 개발을 할 때에는 Spring에서 이 작업을 너무 간단히 해줘서 목적과 필요성을 잘 체감하지 못했었다. 모바일 개발을 시작해보니 너무나도 필요성을 느낀다. 공부가 되었다.

DI 라이브러리로 [Swinject](https://github.com/Swinject/Swinject)를 선택했다. [swift-dependencies](https://github.com/pointfreeco/swift-dependencies)와 같이 다른 선택지가 많았지만, Swinject가 스토리보드를 사용하는 경우에도 편리하게 DI가 되어서 선택했다. 이 경우 [SwinjectStoryboard](https://github.com/Swinject/SwinjectStoryboard)도 함께 추가해야 한다.

## 프로젝트에 패키지 추가

- [문서](https://github.com/Swinject/SwinjectStoryboard?tab=readme-ov-file#installation)를 참고
- Swift Package Manager를 지원하므로, Xcode에서 GUI로 `https://github.com/Swinject/SwinjectStoryboard.git`를 추가했다.
	![](Pasted%20image%2020250127200807.png)

## 간단한 사용 방법

### 1. 코드베이스 프로젝트

```swift
import Swinject

class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    let container: Container = {
        let container = Container()
        container.register(Animal.self) { _ in Cat(name: "Mimi") }
        container.register(Person.self) { r in
            PetOwner(pet: r.resolve(Animal.self)!)
        }
        container.register(PersonViewController.self) { r in
            let controller = PersonViewController()
            controller.person = r.resolve(Person.self)
            return controller
        }
        return container
    }()

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {

        // Instantiate a window.
        let window = UIWindow(frame: UIScreen.main.bounds)
        window.makeKeyAndVisible()
        self.window = window

        // Instantiate the root view controller with dependencies injected by the container.
        window.rootViewController = container.resolve(PersonViewController.self)

        return true
    }
}
```

- AppDelegate나, SceneDelegate를 사용한다면 SceneDelegate에서 DI 컨테이너를 만든다.
- 루트 뷰 컨트롤러가 되어야 할 뷰 컨트롤러의 인스턴스를 컨테이너의 resolve 메서드를 통해 생성하고, UIWindow 객체에 할당한다.

### 2. 스토리보드 프로젝트

```swift
extension SwinjectStoryboard {
    @objc class func setup() {
        defaultContainer.register(Animal.self) { _ in Cat(name: "Mimi") }
        defaultContainer.register(Person.self) { r in
            PetOwner(pet: r.resolve(Animal.self)!)
        }
        defaultContainer.register(PersonViewController.self) { r in
            let controller = PersonViewController()
            controller.person = r.resolve(Person.self)
            return controller
        }
    }
}
```

- 스토리보드를 사용한다면, 스토리보드가 인스턴스화되었을 때 해당 뷰 컨트롤러의 의존성이 주입되어야 할 것이다. 그렇다면 스토리보드 인스턴스화 이전에 컨테이너가 구성되어야 한다.
- 이때 SwinjectStoryboard에 대한 익스텐션으로 setup이라는 클래스 메서드를 정의하고, 여기서 DI 컨테이너를 등록한다. SwinjectStoryboard 내 defaultContainer 프로퍼티를 통해 기본 DI 컨테이너를 참조할 수 있다.
- 그럼 스토리보드가 인스턴스화되는 타이밍에 SwinjectStoryboard가 컨테이너를 통해 의존 컴포넌트를 주입해준다.

### 3. 스토리보드 프로젝트이지만 명시적으로 스토리보드를 인스턴스화

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    var container: Container = {
        let container = Container()
        container.storyboardInitCompleted(AnimalViewController.self) { r, c in
            c.animal = r.resolve(Animal.self)
        }
        container.register(Animal.self) { _ in Cat(name: "Mimi") }
        return container
    }()

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {

        let window = UIWindow(frame: UIScreen.mainScreen().bounds)
        window.makeKeyAndVisible()
        self.window = window

        let storyboard = SwinjectStoryboard.create(name: "Main", bundle: nil, container: container)
        window.rootViewController = storyboard.instantiateInitialViewController()

        return true
    }
}
```

- 1번 경우와 비슷하다.
- AppDelegate나, SceneDelegate를 사용한다면 SceneDelegate에서 DI 컨테이너를 만들고, SwinjectStoryboard를 create 메서드를 통해 생성하면서 만든 컨테이너를 넘겨준다.
- SwinjectStoryboard로부터 뷰 컨트롤러를 인스턴스화하고 루트 뷰 컨트롤러에 할당한다.

## 객체 스코프

- [여기](https://github.com/Swinject/Swinject/blob/master/Documentation/ObjectScopes.md)에서 문서를 확인할 수 있다.
- 객체 스코프란 컨테이너로부터 객체를 제공받을 때 어느 정도로 공유하게 할 것인지에 관한 개념이다. 아래에서 확인할 수 있지만 DI 컨테이너에는 객체 그 자체가 등록되는 것이 아니라, 팩토리 클로져가 등록되는 것이다. 따라서 resolve 메서드를 통해  필요한 객체를 얻을 때 해당 객체를 전역에 공유할지, 전혀 공유하지 않을 것인지 등 공유의 범위를 따로 정해줄 수 있다.
- 이렇게 register 하면서 지정하면 된다.

	```swift
	container.registe(AProtocol.self) { _ in AObject() }
    .inObjectScope(.container) // 👈
	```

- 기본 스코프는 다음과 같다.
	1. Transient: 매번 새로운 객체를 만든다.
	2. Graph (기본값): 명시적으로 컨테이너에 대해 resolve를 호출해 객체를 생성하는 경우 해당 객체는 무조건 새로 생성한다. 그러나 해당 객체를 반환하기 위해 여러 의존성을 해결하면서 생성되는 객체들은 공유한다.
		- 예를 들어,

			```
			ReadService - Repository - Logger
			            \ Logger

            WriterService - Repository - Logger
                        \ Logger
		   ```
			```swift
			let reader = container.resolve(ReadServiceProtocol.self)!
		 .  let writer = container.resolve(WriterServiceProtocol.self)!
	       ```
		   
		   - 이러한 관계가 있을 때 동일 객체를 표시하면 다음과 같다.

			```
			ReadService - Repository(A) - Logger(B)
			            \ Logger(B)

            WriterService - Repository(C) - Logger(D)
                        \ Logger(D)
		   ```

	1. Container: 싱글톤 객체로 다룬다. resolve할 때마다 객체를 만들지 않고 하나를 공유한다.
	2. Weak: 이 스코프인 상태에서 객체가 생성되면, 컨테이너에서 해당 객체가 참조되고 있을 경우 해당 객체가 공유된다. 참조가 없어진 후 resolve가 호출되면 새로 생성한다.

- 객체가 가지는 상태에 따라 다르게 지정해줘야할 것 같다. 전역적으로 상태가 공유되어야 하거나 stateless하여 매번 생성되어서는 안되거나 생성할 필요가 없으므로 Container를 쓰면 될 것이다.
- 객체가 특정 컨텍스트에서 가지는 별개의 상태를 가진다면 Transient나 Graph를 쓰면 될 것이다. 컨텍스트가 특정 컴포넌트에 국한하여 정해지는 경우라면 Graph가 맞겠다.
- 컴포넌트 간 라이프 사이클이 상응하는 것이 중요하다면 Weak이 좋겠다.
- 예제를 [여기](https://github.com/0tak2/ios-study/tree/main/self-study/swinject-study/swinject-study)에 작성해보았다.

## DI 컨테이너 분석

- Swinject의 [Container](https://github.com/Swinject/Swinject/blob/c630fcaa99cc3f3af7a48018477c9e03236a7ca9/Sources/Container.swift)를 보면 대략적인 메커니즘을 확인할 수 있다.
- register
	```swift
	public func _register<Service, Arguments>(
        _ serviceType: Service.Type,
        factory: @escaping (Arguments) -> Any,
        name: String? = nil,
        option: ServiceKeyOption? = nil
    ) -> ServiceEntry<Service> {
        syncIfEnabled {
            let key = ServiceKey(serviceType: Service.self, argumentsType: Arguments.self, name: name, option: option)
            let entry = ServiceEntry(
                serviceType: serviceType,
                argumentsType: Arguments.self,
                factory: factory,
                objectScope: defaultObjectScope
            )
            entry.container = self
            services[key] = entry

            behaviors.forEach { $0.container(self, didRegisterType: serviceType, toService: entry, withName: name) }

            return entry
        }
    }
	```

	- [여기](https://github.com/Swinject/Swinject/blob/c630fcaa99cc3f3af7a48018477c9e03236a7ca9/Sources/Container.swift#L138)에서 확인할 수 있다.
	- register 메서드에 전달된 타입 객체와 이름 등을 조합해 키를 만든다. 이 키에, 팩토리 클로져 등을 값으로 하여, `services: [ServiceKey: ServiceEntryProtocol]` 딕셔너리에 할당한다.
- resolve
	```swift
    public func _resolve<Service, Arguments>(
        name: String?,
        option: ServiceKeyOption? = nil,
        invoker: @escaping ((Arguments) -> Any) -> Any
    ) -> Service? {
        // No need to use weak self since the resolution will be executed before
        // this function exits.
        syncIfEnabled {
            var resolvedInstance: Service?
            let key = ServiceKey(serviceType: Service.self, argumentsType: Arguments.self, name: name, option: option)

            if key == Self.graphIdentifierKey {
                return currentObjectGraph as? Service
            }

            if let entry = getEntry(for: key) {
                resolvedInstance = resolve(entry: entry, invoker: invoker)
            }

            if resolvedInstance == nil {
                resolvedInstance = resolveAsWrapper(name: name, option: option, invoker: invoker)
            }

            if resolvedInstance == nil {
                debugHelper.resolutionFailed(
                    serviceType: Service.self,
                    key: key,
                    availableRegistrations: getRegistrations()
                )
            }

            return resolvedInstance
        }
    }
    
	fileprivate func getEntry(for key: ServiceKey) -> ServiceEntryProtocol? {
        if let entry = services[key] {
            return entry
        } else {
            return parent?.getEntry(for: key)
        }
    }
	```
	
	- [여기](https://github.com/Swinject/Swinject/blob/c630fcaa99cc3f3af7a48018477c9e03236a7ca9/Sources/Container.swift#L234)에서 확인할 수 있다.
	- 반환 받을 타입과 이름 등으로 키를 조합하고, services 딕셔너리로부터 해당 키에 해당하는 밸류를 찾아 반환한다.

## SwinjectStoryboard 분석

- SwinjectStoryboard는 본래 UIStoryboard의 생성자를 스위즐링한다.

	```swift
	import UIKit
	
	extension UIStoryboard {
	    static func swizzling() {
	        DispatchQueue.once(token: "swinject.storyboard.init") {
	            let aClass: AnyClass = object_getClass(self)!
	
	            let originalSelector = #selector(UIStoryboard.init(name:bundle:))
	            let swizzledSelector = #selector(swinject_init(name:bundle:))
	
	            let originalMethod = class_getInstanceMethod(aClass, originalSelector)!
	            let swizzledMethod = class_getInstanceMethod(aClass, swizzledSelector)!
	
	            let didAddMethod = class_addMethod(aClass, originalSelector,
	                                               method_getImplementation(swizzledMethod),
	                                               method_getTypeEncoding(swizzledMethod))
	
	            guard didAddMethod else {
	                method_exchangeImplementations(originalMethod, swizzledMethod)
	                return
	            }
	            class_replaceMethod(aClass, swizzledSelector,
	                                method_getImplementation(originalMethod),
	                                method_getTypeEncoding(originalMethod))
	        }
	    }
	
	    @objc class func swinject_init(name: String, bundle: Bundle?) -> UIStoryboard {
	        guard self == UIStoryboard.self else {
	            return self.swinject_init(name: name, bundle: bundle)
	        }
	        // Instantiate SwinjectStoryboard if UIStoryboard is trying to be instantiated.
	        if SwinjectStoryboard.isCreatingStoryboardReference {
	            return SwinjectStoryboard.createReferenced(name: name, bundle: bundle)
	        } else {
	            return SwinjectStoryboard.create(name: name, bundle: bundle)
	        }
	    }
	}	
	```
	
	- [여기](https://github.com/Swinject/SwinjectStoryboard/blob/7a446cc8d8880a403519a4a43e1b64c086dd265c/Sources/iOS-tvOS/UIStoryboard%2BSwizzling.swift)를 보면 확인할 수 있다.
	- 원래 생성자에 Swinject 초기화 메서드를 더해 기존 생성자를 치환하고 있다. 이를 통해 최종적으로는 UIStoryboard 대신 [SwinjectStoryboard](https://github.com/Swinject/SwinjectStoryboard/blob/7a446cc8d8880a403519a4a43e1b64c086dd265c/Sources/SwinjectStoryboard.swift)의 객체가 반환된다.
- SwinjectStoryboard는 기본 Swinject DI 컨테이너를 가지고 있고, 의존성 해결을 위한 여러 메서드를 정의하고 있어 스토리보드 프로젝트에도 DI가 가능하게 된다.
- 이를 응용하면 간단한 구현으로 스토리보드 프로젝트에 다른 DI 라이브러리도 적용할 수 있을 것 같다.
- 참고자료: https://developer.apple.com/documentation/objectivec/objective-c_runtime
