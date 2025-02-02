WWDC19
https://developer.apple.com/videos/play/wwdc2019/413

## XCTest 소개

### 개요
- Xcode에서 제공하는 빌트인 테스트 프레임워크
- 테스트의 중요성
	- 버그 발견
	- 요구사항 재정리 및 기대 검증
- 테스트 프라미드: 테스트 범위(thoroughness), 테스트 품질, 테스트 속도에 따른 차이
	- 유닛 테스트
		- 피라미드의 기반.
		- 코드의 한 단위를 검증.
		- 일반적으로 하나의 함수의 인풋에 따른 아웃풋 검증.
		- 간단하고 실행 속도 매우 빠름.
		- 가장 많은 테스트를 필요로 함.
	- 통합 테스트
		- 유닛 테스트보다 더 큰 범위를 검증.
		- 특정 하위 시스템이나 여러 클래스들이 함께 적절하게 동작하는지 검증함.
		- 유닛 테스트 위에 존재함. 적어도 컴포넌트의 각종 기능이 개별적으로 잘 동작하는지 보장되어야 다른 컴포넌트와 함께 잘 작동하는지 확인하는게 의미가 있기 때문임.
		- 보통 유닛 테스트만틈 많이 작성할 필요는 없음.
		- 실행하는데 유닛 테스트보다 조금 더 오래 걸리지만, 한 번 실행될 때 더 많은 범위를 검증함.
	- UI 테스트
		- 유저 입장에서 테스트하는 것
		- 앱이 예상한대로 실행되는 지를 최종적으로 테스트
		- 실행하는데 오래 걸리지만, 앱이 실제로 잘 동작하는 지를 시연
		- UI는 빈번하게 수정되므로 UI 테스트도 유지보수에 더 신경써야 함
	- 세 개의 층이 피라미드 형태로 적절하게 밸런스를 맞추는게 좋음
- XCTest에서의 테스트 타입
	- 유닛 테스트, UI 테스트, 성능 테스트
	- 이 발표에서는 유닛 테스트와 UI 테스트에 중점을 둠

### Testing from Scratch

### 프로젝트 생성
- 프로젝트를 만들 떄 Include Unit Tests와 Include UI Tests에 체크
- 프로젝트 네비게이터에 테스트를 위한 타겟이 자동으로 추가되어 있는 것을 확인할 수 있음

### 간단한 사용 방법
```swift
// How to use XCTest

import XCTest
@testable import MyCounterApp

class MyCounterTests: XCTestCase {
	override func setUp() {
		// Put setup code here.
	}

	override func tearDown() {
		// Put teardown code here.
	}

	func testIncrementCounter() {
		var counter = Counter()
		counter.increment()
		
		XCTAssertEqual(counter.value, 1,
			"Counter was not incremented: \(counter)")
	}
}
```
- XCTest 임포트, 테스트할 모듈을 @testable 어트리뷰트를 붙여 임포트
	- @testable을 붙이는 이유: 특정 엔티티의 접근 권한이 internal (기본값)인 경우, 원칙대로라면 해당 엔티티가 정의된 모듈이 아닌 다른 모듈에서는 해당 엔티티를 참조할 수가 없음. 
	- 그렇다면 현재 타겟이 분리되어 있기 때문에 테스트 코드에서 internal 엔티티는 참조할 수 없게 됨.
	- 그렇다고 해서 오직 테스트를 위해 open이나 public으로 변경하는 것은 캡슐화를 불가능하게 함.
	- 이렇게 테스팅을 하는 경우 테스트하고자 하는 모듈에 대해 @testable 어트리뷰트를 붙여 임포트하면 internal 레벨의 엔티티도 테스트 타겟에서 참조 가능함.
	- 참고
		- https://holyswift.app/what-is-testable-annotation-in-swift/
		- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/accesscontrol/#Access-Levels-for-Unit-Test-Targets
- 테스트 클래스는 XCTestCase를 상속하여 Xcode에서 테스트할 떄 사용할 수 있음을 나타냄
- 테스트 케이스 각각은 이 클래스의 메서드로 작성
	- 이름은 test로 시작해 테스트하고자 하는 항목을 같이 씀
	- Assertion API를 이용해 테스트 결과를 검증
- 메서드를 작성하면 오른쪽에 다이아몬드 표시가 나옴. 실행 가능하다는 의미
	- 테스트 실행 후 실패하면 빨간색 X표 다이아몬드로 변경됨
	- 수정 후 다시 실행해 성공하면 초록색 체크 다이아몬드로 변경됨
- 초기화와 클린업 - setUp() 메서드와 tearDown() 메서드를 오버라이드
	- setUp() 메서드와 tearDown() 메서드는 각 테스트 케이스 전 후에 매번 실행된다.
		![](Pasted%20image%2020250202100554.png)

### 데모
#### 단위 테스트
- 여행 관련 앱에서, 도시 이름을 두 개 입력받으면 도시 사이 거리를 마일로 계산하여 반환하는 DistanceCalculator 테스트
- 같은 파일 내의 딕셔너리로 각 도시와 위도/경도가 입력되어 있고, 추후 DB로 옮길 예정

```swift
class DistanceCalculatorTests: XCTestCase {
	var calculator: DistanceCalculator!

	override func setUp() {
		calculator = DistanceCalculator()
	}

	override func tearDown() {
	}

	func testCoordinatesOfSeattle() throws {
		let city = try XCTUnwrap(calculator.city(forName: "Seattle"))

		XCTAssertEqual(city.coordicates.latitiude, 47.61)
		XCTAssertEqual(city.coordicates.longitude, -122.33)
	}

	func testSanFranciscoToNewYork() throws {
		let distanceInMiles = try calculator.distaceInMiles(from: "San Francisco", to: "New York City")
		XCTAssertEqual(distanceInMiles, 2571, accuracy: 1)
	}

	func testCupertinoNotRecognized() throws {
		XCTAssertThrowsError(try calculator.dinstanceInMiles(from: "Cupertino", to: "New York City")) { error in
			XCTAsserEqual(error as? DistanceCalculator.Error, .unknownCity("Cupertino"))
		}
	}
}	
```

##### 팁
- 화면을 이분할 하여, 위의 에디터에 테스트 코드, 아래의 에디터에 원래 코드를 띄워놓고 작업 -> 편해 보임
- Optional을 언래핑할 떄에는 강제 언래핑하지 않고 XCTUnwrap API를 활용. nil이면 예외 발생
- 애초에 테스트 메서드 정의 시 throws 지정, 예외처리를 따로 하지 않고 예외가 발생하면 테스트 실패하도록 함
- XCTAssertEqual에 accuracy를 지정해 허용 가능 오차를 지정할 수 있음
- 의도적으로 예외가 발생하는지 테스트 하기 위해서는 XCTAssertThrowsError API 사용
- 왼쪽 패널의 다이아몬드 아이콘을 누르면 작성한 테스트 클래스, 케이스들이 보이고 조작 가능하다.

#### UI 테스트
- UI 테스트는 코드 내부 구조와 상관 없이 진행되는 블랙박스 테스트
- 화면의 요소와 인터랙션 하면서 테스트 진행

```swift
class DiscoverUITests: XCTestCase {
	let app = XCUIApplication()

	override func setUp() {
		continueAfterFailure = false // 앞에서 테스트에 이미 실패하면 더 이상 진행하지 않음
		app.launch()
	}

	override func tearDown() {
	}

	func testMilesToParis() {
		app.tabBars.buttons["Discover"].tap() // 탭바 아이템 터치
		XCTAssert(app.staticTexts["San Francisco"].isHittable) // San Francisco 문자열이 나타나 인터랙션 가능한 상태인지 체크

		let sfImage = app.image("san-francisco")
		sfImage.swipeLeft() // 이미지에 대해 스와이프
		XCTAssert(app.staticTexts["San Francisco"].isHittable)

		XCTAssert(app.staticTexts["5586 miles away"].isHittable)
	}
}
```

##### 팁
- UI 요소를 어떻게 참조할지 모르겠는 경우
	- 테스트 코드 중간에 브레이크포인트를 만들고 실행
	- 디버거가 활성화되면 po app 커맨드 입력 `(lldb) po app`
	- 이미지만 출력하고 싶은 경우 `(lldb) po app.images`
		![](Pasted%20image%2020250202111639.png)

### Test Organization

- 앱 자체로 좁혀 본다면
	![](Pasted%20image%2020250202112209.png)
- 범위를 넓혀보면
	![](Pasted%20image%2020250202112412.png)

### Code Coverage

- 테스트코드가 소스코드를 얼마나 커버하는지
- 왼쪽 패널의 말풍선 모양 -> Coverage를 보면 타겟 별 커버리지 확인 가능

## Test plans

최근 영상을 보자...

## Continuous integration workflows

