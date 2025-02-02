WWDC24
https://developer.apple.com/videos/play/wwdc2024/10179

## Add Test Bundle
- File - New - Target - Unit Testing Bundle
- Xcode 16부터 기본이 Swift Testing이므로 바로 Finish

## Building Blocks
###  1. Test Functions

```swift
import Testing

@Test func videoMetadata() {
    // ...
}
```
 - Testing 모듈 임포트
 - @Test 어트리뷰트와 함께 함수 작성 - 테스트 함수임을 표현
 - 왼쪽에 다이아몬드 모양의 테스트 실행 버튼이 생김

정리하자면,
- 테스트 함수는 @Test 어트리뷰트가 사용된 함수
- 글로벌 함수도 가능하고, 특정 타입에 속한 메서드도 가능
- `async` 함수일 수도, `throws` 함수일 수도 있음
- 특정 글로벌 액터로 격리할 수도 있음 (에를 들어 `@MainActor`)

### 2. Expectations
- 특정 비디오의 메타데이터가 예상과 일치하는지 검증하는 테스트 케이스 예제
	```swift
	import Testing
	@testable import DestinationVideo // internal 요소에 접근하기 위해 @testable 사용
	
	@Test func videoMetadata() {
	    let video = Video(fileName: "By the Lake.mov")
	    let expectedMetadata = Metadata(duration: .seconds(90))
	    #expect(video.metadata == expectedMetadata)
	}
	```

- \#expect 매크로
	- 기대한 상태가 true인지 검증하는 매크로
	- 일반적인 표현식과 연산자 허용
	- 실패할 경우 넘겨진 소스코드와 표현식들의 값을 캡쳐해 띄워줌
		![](Pasted%20image%2020250202133301.png)
		- 일반적인 프레임워크와 달리 좌항, 우항을 구분하지 않고 하나의 표현식으로 표현했는데도 찰떡같이 표시해줌
	- 매우 유연함
		```swift
		#expect (1 == 2)
		#expect (user.name == "Alice")
		#expect (!arr.isEmpty)
		#expect (numbers.contains(3))
		```

### Required Expectations
- 다음 테스트를 진행하기 위해 반드시 성공해야 하는 기대 상태를 표현 ([문서](https://developer.apple.com/documentation/testing/require(_:_:sourcelocation:)-5l63q))
	- `#require(1 == 2) // 테스트 실패`
	- 성공하지 못하면 테스트 중단 및 실패
	- XCTest에서는 setUp()에서 `continueAfterFailer = true` 지정
- 또는 Optional 값이 반드시 nil이 아닌 값으로 언래핑되어야 다음 테스트를 진행할 수 있는 경우 ([문서](https://developer.apple.com/documentation/testing/require(_:_:sourcelocation:)-6w9oo))
	- 이 경우는 XCTUnwrap과 비교할 수 있을 것 같음.
		```swift
		let task = try #require(paymentTasks.first) // nil이면 테스트 중단 및 실패
		#expect(task.isDefault)
		```

### 3. Traits
- @Test 어트리뷰트에 Display Name을 입력해줄 수 있음. `@Test("CoreData 엔티티 저장에 성공해야 한다")` 한글도 된다.
- 테스트 네비게이터와 같이 Xcode에 이 이름이 표시됨
- 이렇게 @Test 어트리뷰트에는 다양한 Traits을 추가할 수 있음

- 테스트에 대한 설명 정보 추가
- 테스트 실행 시기 또는 실행 여부 조정
- 테스트 동작 방식 조정

![](Pasted%20image%2020250202140005.png)

### 4. Test Suite
- 관련 있는 테스트 함수 여러 개를 하나의 스위트로 그루핑할 수 있음
- 간단하게 여러 테스트 함수(`@Test func test~`)들을 감싸는 구조체를 만들면 됨 (클래스, 액터도 가능하지만 구조체를 추천한다고 함)
- 원래 @Suite 어트리뷰트를 붙여야 하지만 테스트 함수들이나 다른 테스트 스위트를 가지고 있다면 암시적으로 테스트 스위트로 취급됨

```swift
struct VideoTests {
    let video = Video(fileName: "By the Lake.mov")

    @Test("Check video metadata") func videoMetadata() {
        let expectedMetadata = Metadata(duration: .seconds(90))
        #expect(video.metadata == expectedMetadata)
    }

    @Test func rating() async throws {
        #expect(video.contentRating == "G")
    }

}
```

- Group related test functions and suites
- Annotated using @Suite
	- Implicit for types containing @Test functions or suites
- May have stored instance properties
- Use init and deinit for set-up and tear-down logic, respectively
- Initialized once per instance @Test method
	- 상태 공유 방지
	- 즉, 내부의 테스트 함수가 실행될 때마다 새로운 테스트 스위트 객체가 생성됨. 위의 테스트에서, videoMetadata()와 rating()이 각각 실행될 때 video는 서로 다른 객체임.

## Common Workflows

### 조건이 있는 테스트

- 특정 조건에서만 실행할 테스트
	```swift
	@Test(.enabled(if: AppFeatures.isCommentingEnabled))
	func videoCommenting() {
	    // ...
	}
	```
	- 런타임에 조건 실행, false면 스킵
- 비활성화할 테스트
	```swift
	@Test(.disabled("Due to a known crash"))
	func example() {
	    // ...
	}
	```
	- 주석처리 대신 비활성화
- 버그
	```swift
	@Test(.disabled("Due to a known crash"),
	      .bug("example.org/bugs/1234", "Program crashes at <symbol>"))
	func example() {
	    // ...
	}
	```
	- 이슈 관리까지 한 번에 가능
- iOS, MacOS 버전에 따른 조건부 테스트
	```swift
	// ❌ Avoid checking availability at runtime using #available
	@Test func hasRuntimeVersionCheck() {
	    guard #available(macOS 15, *) else { return }
	
	    // ...
	}
	
	// ✅ Prefer @available attribute on test function
	@Test
	@available(macOS 15, *)
	func usesNewAPIs() {
	    // ...
	}
	```
	- `@available` 어트리뷰트 사용. `guard #available(/* ... */)` 구문을 쓸 필요가 없음.

### 테스트 간 공통점 관리
- 태그 사용 가능

```swift
@Test(.tags(.formatting)) func rating() async throws {
    #expect(video.contentRating == "G")
}
```

- 다른 테스트 스위트에 위치한 서로 다른 테스트 케이스들을 태그 기반으로 연결할 수 있음
- Xcode 테스트 내비게이터에 표시됨
- 테스트 스위트에 태그를 추가할 수도 있음. 이렇게 하면 하위 테스트에 일괄 적용됨
	```swift
	@Suite(.tags(.formatting))
	struct MetadataPresentation {
	    let video = Video(fileName: "By the Lake.mov")
	
	    @Test func rating() async throws {
	        #expect(video.contentRating == "G")
	    }
	
	    @Test func formattedDuration() async throws {
	        let videoLibrary = try await VideoLibrary()
	        let video = try #require(await videoLibrary.video(named: "By the Lake"))
	        #expect(video.formattedDuration == "0m 19s")
	    }
	}
	```

### 다양한 인자로 테스트
```swift
struct VideoContinentsTests {

    @Test func mentionsFor_A_Beach() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "A Beach"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    @Test func mentionsFor_By_the_Lake() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "By the Lake"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    @Test func mentionsFor_Camping_in_the_Woods() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "Camping in the Woods"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    // ...and more, similar test functions
}
```

- 테스트 코드를 짜다보면 흔한 경우
- 데이터마다 테스트 케이스를 작성하다보면 테스트 코드 유지보수가 힘들어짐
- 이럴 때 필요한 인자를 매개변수화할 수 있음

```swift
struct VideoContinentsTests {

    @Test("Number of mentioned continents", arguments: [
        "A Beach",
        "By the Lake",
        "Camping in the Woods",
        "The Rolling Hills",
        "Ocean Breeze",
        "Patagonia Lake",
        "Scotland Coast",
        "China Paddy Field",
    ])
    func mentionedContinentCounts(videoName: String) async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: videoName))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

}
```

- 별도의 테스트케이스로 취급되어 병렬 실행
- Xcode 테스트 내비게이터에도 각각 별도의 테스트케이스인 것처럼 표시됨
- 특정 인자에 대해서만 개별 테스트를 돌리거나 할 수도 있음
- 자체적으로 하나의 테스트 함수에서 반복문을 돌리는 것보다 훨씬 이점이 많음

## Swift Testing and XCTest
### 차이점 비교
- 테스트 함수
	![](Pasted%20image%2020250202143520.png)
- 기대 검증
	- \#expect
		![](Pasted%20image%2020250202143729.png)
		![](Pasted%20image%2020250202143708.png)
		![](Pasted%20image%2020250202143758.png)
	- \#require
		![](Pasted%20image%2020250202143847.png)
- 테스트 스위트
	![](Pasted%20image%2020250202144849.png)

### 마이그레이션할 때 고려할 점
 - a single unit test target
	- Swift Testing tests can coexist with XCTests Consolidate similar XCTests into a parameterized test
- Consolidate similar XCTests into a parameterized test
- Migrate each XCTest class with only one test method to a global @Test function
- Remove redundant "test" prefix from method names

 - Continue using XCTest for tests which:
	- ﻿﻿Use Ul automation APls (such as XCUIApplication)
	- ﻿﻿Use performance testing APls (such as XCTMetric )
	- ﻿﻿Can only be written in Objective-C
	
	=> 결국 당장은 XCTest와 Swift Testing이 공존할 수밖에 없음
	
	- Avoid calling XCTAssert (...) from Swift Testing tests, or \#expect (...) from XCTests -> 교차 사용 금지

## 오픈 소스
- 오픈소스 프로젝트. 커뮤니티 드리븐 (swiftlang, Swift Forum)
- 플랫폼에 종속되지 않음. Xcode가 아니더라도 테스트 가능.
	- VSCode 익스텐션, 터미널에서 `swift test`로 테스트 실행, ...

##  추가 자료
- https://developer.apple.com/documentation/Testing
- https://github.com/swiftlang/swift-testing
