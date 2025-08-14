WWDC23  
https://developer.apple.com/kr/videos/play/wwdc2023/10156

## 뷰 업데이트와 애니메이션 렌더링의 관계
### 애니메이션과 관계 없는 상태 변화
1. 처음 뷰를 렌더링할 때, SwiftUI는 뷰의 body를 읽으면서 어트리뷰트 그래프를 만든다. body는 다시 호출하지 않는다.
2. 뷰 상태가 업데이트되면 업데이트 트랜잭션이 시작되고, 뷰를 그릴 때 필요한 어트리뷰트가 비활성화된다.
3. SwiftUI는 뷰의 body를 다시 읽어 다시 렌더링한다. 트랜잭션이 닫힌다.

### 애니메이션 가능한 상태가 변경될 때
1. 뷰 상태가 업데이트되면, 업데이트 트랜잭션이 시작되고, 뷰를 그릴 때 필요한 어트리뷰트가 비활성화된다.
2. 이때, 애니메이션 가능한 상태가 withAnimation 안에서 변경된 경우, 애니메이션이 트리거된다. SwiftUI는 애니메이션 사본을 만들고, 해당 상태의 현재 시점의 값을 보간하여 업데이트한다.
3. 해당 상태에 대해, SwiftUI는 Model 값과, Presentation 값을 따로 관리한다. (실제 변경된 값, 현재 시간에 따라 노출되어야하는 값)
4. 트랜잭션이 닫힌다.
5. SwiftUI는 뷰의 body를 다시 읽어 다시 렌더링한다.

	![](attachments/Pasted%20image%2020250813163714.png)

## [Animatable](https://developer.apple.com/documentation/SwiftUI/Animatable)
### Animatable과 Animation
- Animatable: scaleEffect처럼 애니메이션을 적용할 데이터
- Animation: 시간에 따른 데이터의 실제 변화

### Animatable의 구현
- animatableData는 Vector로 기술되어야 함

![](attachments/Pasted%20image%2020250813165337.png)

![](attachments/Pasted%20image%2020250813165402.png)

- VectorArithmetic은 자릿수가 고정된 여러 수의 시퀀스일 뿐임
- 위에서 설명했던 scaleEffect는 CGFloat 값을 받고, CGFloat는 VectorArithmetic를 채택하므로 바로 다음과 같이 쓸 수 있음 => 애니메이션 가능한 데이터 Animatable
	![](attachments/Pasted%20image%2020250813165721.png)

- 보통은 Animatable을 직접 쓸 일은 없음
### Animatable을 사용하는 고급 애니메이션 ([예제](https://developer.apple.com/documentation/swiftui/composing_custom_layouts_with_swiftui))
#### AS-IS
![](attachments/Pasted%20image%2020250813165835.png)
- 아바타가 직선 경로로 애니메이션됨

#### TO-BE
![](attachments/Pasted%20image%2020250813170009.png)
- Podium 뷰가 Animatable을 채택하게 하고, animatableData를 offset으로 지정. position이 아닌 offset(Angle)을 기준으로 애니메이팅됨.

GQ. Layout이 뭐지?
GA. [Layout](https://developer.apple.com/documentation/swiftui/layout) 문서 읽어보기

![](attachments/Pasted%20image%2020250813173002.png)

=> ⚠️ Animatable을 직접 구현하면 애니메이션 모든 프레임에서 body를 호출하므로 성능 문제가 발생할 수 있음. 이게 유일한 방법일 때만 사용하는 게 좋다.

## Animation
- Animatable 데이터를 시간에 따라 보간하는 알고리즘
- Animation 분류
	- Timing curve
		![](attachments/Pasted%20image%2020250813173442.png)
		- 애니메이션 속도와 지속 시간을 결정하는 커브를 갖음
			- 커브는 비지어 곡선의 컨트롤 포인트를 통해 만듦
			- 0과 1사이의 특정한 상대적 지점에서의 값과 속도를 계산할 수 있음
				![](attachments/Pasted%20image%2020250813173713.png)
	- Spring
		![](attachments/Pasted%20image%2020250813173957.png)
		- 스프링 시뮬레이션을 통해 주어진 지점에서의 값을 게산
			- 원래 계산 방법: 질량, 강성, 감쇠
			- SwiftUI에서 도입된 방식: 기간, 바운스
			- 여기에도 커브를 사용
	- Higher order
		![](attachments/Pasted%20image%2020250813174106.png)
		- 스피드, 딜레이, 반복(횟수 + 반전) 등을 커스텀할 수 있음
	- [Custom Animation](https://developer.apple.com/documentation/swiftui/customanimation)
		![](attachments/Pasted%20image%2020250814102424.png)
		- 프로토콜 개요
			- `func animate<V>(value: V, time: TimeInterval, context: inout AnimationContext<V>) -> V?` (Required)
			- `func velocity<V>(value: V, time: TimeInterval, context: AnimationContext<V>) -> V?` (Default Implementation Provided)
			- `func shouldMerge<V>(previous: Animation, value: V, time: TimeInterval, context: inout AnimationContext<V>) -> Bool`  (Default Implementation Provided)
		- `func animate<V>(value: V, time: TimeInterval, context: inout AnimationContext<V>) -> V?` 
			- value: 애니메이션을 적용할 벡터
				- 시작 벡터와 종료 벡터의 차이를 계산해 델타 벡터가 전달된다. 이 점을 고려해서 구현할 것.
			- time: 경과 시간
			- context: 추가적인 애니메이션 상태
			- 애니메이션이 끝났으면 nil을, 아니면 현재 벡터를 반환
			![](attachments/Pasted%20image%2020250814103722.png)
			- 1.0 -> 1.5로 증가할 때의 애니메이션이지만, 실제로 델타를 계산하여 는 0.0 -> 0.5를 보간하게 됨. 이 델타 벡터가 value로 주어짐
		- `func shouldMerge<V>(previous: Animation, value: V, time: TimeInterval, context: inout AnimationContext<V>) -> Bool` 
			- 특정 애니메이션이 렌더링될 때 동일한 애니메이션이 다시 트리거된다면?
			- 이때 showMerge가 호출됨
			- 이전 애니메이션과 합쳐도 되면 true를 반환하도록 구현, 아니면 false를 반환하도록 구현
				- If `shouldMerge(previous:value:time:context:)` returns `true`, the system merges the new animation instance with the previous animation. The system provides to the new instance the state and elapsed time from the previous one. Then it removes the previous animation.
				- If this method returns `false`, the system doesn’t merge the animation with the previous one. Instead, both animations run together and the system combines their results.
			- 이를 위해서 델타 벡터를 취급하는 것이기도 함. 합칠 때 트리거 시점이 다른 여러 벡터를 정확하게 합치기 위해 델타끼리 계산한다는 것.
			![](attachments/Pasted%20image%2020250814103913.png)
			- 리니어 애니메이션과 스프링 애니메이션을 직접 구현하는 경우, shouldMerge 구현의 차이에 대해 설명해주는데 솔직히 잘 이해못함... 맹글어봐야 알 것 같음...

```swift
struct MyLinearAnimation: CustomAnimation {
    var duration: TimeInterval

    func animate<V: VectorArithmetic>(
        value: V, time: TimeInterval, context: inout AnimationContext<V>
    ) -> V? {
        if time <= duration {
            value.scaled(by: time / duration)
        } else {
            nil // animation has finished
        }
    }

    func velocity<V: VectorArithmetic>(
        value: V, time: TimeInterval, context: AnimationContext<V>
    ) -> V? {
        value.scaled(by: 1.0 / duration)
    }
}
```

## Transaction
- 방금까지는 트랜잭션이라는 말을, 특정 UI 업데이트 시 진행되는 태스크들의 시퀀스를 지칭하려고 사용했음
- 사실 SwiftUI에서 트랜잭션이란, 데이터 플로우 구조와 그와 관련한 API 중 하나임
- SiwftUI에서의 암시적인 데이터 플로우 구조
	- Environment
	- Preferences
	- **Transaction**
- 딕셔너리. SwiftUI가 UI 업데이트 관련 컨텍스트를 암시적으로 전파할 때 사용. 특히 애니메이션.

### 애니메이션과 트랜잭션
![](attachments/Pasted%20image%2020250814105753.png)

![](attachments/Pasted%20image%2020250814105802.png)

![](attachments/Pasted%20image%2020250814105841.png)

- 트랜잭션 데이터는 뷰 계층 아래로 흐른다.
- 트랜잭션은 직접 모디파이어로 제어도 가능하다
	![](attachments/Pasted%20image%2020250814110033.png)
- 다만 위의 방식은 매번 애니메이션을 오버라이드하기 때문에 안전하지 않음. 대신 animation 모디파이어를 쓰면 안전함
	![](attachments/Pasted%20image%2020250814110211.png)
	- 관찰할 값을 지정해 스코프를 분명히 함
	- 이렇게 고치면 withAnimation이 필요 없어짐
		![](attachments/Pasted%20image%2020250814110308.png)
- 여러 뷰에 대한 애니메이션을 각각 다른 방식으로 렌더링하고 싶은 경우
	- 이렇게?
		![](attachments/Pasted%20image%2020250814113558.png)
		- 순서는 지켜지지만 격리되지 않으므로, 동시에 트리거되었을 때 예상하지 않은 결과가 나올 수 있음
	- 개선
		![](attachments/Pasted%20image%2020250814113553.png)
		- animation 노드마다 사본이 복사됨 => 격리

### 커스텀 트랜잭션 키
- Environment에 커스텀 키를 만들 수 있는 것처럼 커스텀 Transaction Key도 만들 수 있음
	![](attachments/Pasted%20image%2020250814114020.png)

- 예시: 사용자가 버튼을 눌러서 트리거된 경우와 프로그래밍 방식으로 트리거된 경우를 구분해 애니메이팅 분기
	- 시도
		![](attachments/Pasted%20image%2020250814114143.png)
		- withTransaction으로 래핑하고 있음
		- 키와 기록할 값을 클로져와 함께 전달함
		- 괜찮아보이지만 뷰 업데이트마다 transaction 모디파이어가 불리면서 예상하지 못한 동작이 일어날 수 있음
	- 개선 1
		![](attachments/Pasted%20image%2020250814114418.png)
		- transaction 모디파이어에 value를 넘겨 업데이트 스코프를 좁혔음 
	- 개선 2
		![](attachments/Pasted%20image%2020250814114512.png)
		- 트랜잭션의 하위 계층에서 scaleEffect 갱신

 