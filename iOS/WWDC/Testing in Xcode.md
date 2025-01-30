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

## Test plans


## Continuous integration workflows

