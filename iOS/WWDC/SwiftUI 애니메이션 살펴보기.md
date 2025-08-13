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

	![](Pasted%20image%2020250813163714.png)

## [Animatable](https://developer.apple.com/documentation/SwiftUI/Animatable)
### Animatable과 Animation
- Animatable: scaleEffect처럼 애니메이션을 적용할 데이터
- Animation: 시간에 따른 데이터의 실제 변화

### Animatable의 구현
- animatableData는 Vector로 기술되어야 함

![](Pasted%20image%2020250813165337.png)

![](Pasted%20image%2020250813165402.png)

- VectorArithmetic은 자릿수가 고정된 여러 수의 시퀀스일 뿐임
- 위에서 설명했던 scaleEffect는 CGFloat 값을 받고, CGFloat는 VectorArithmetic를 채택하므로 바로 다음과 같이 쓸 수 있음 => 애니메이션 가능한 데이터 Animatable
	![](Pasted%20image%2020250813165721.png)

- 보통은 Animatable을 직접 쓸 일은 없음
### Animatable을 사용하는 고급 애니메이션 ([예제](https://developer.apple.com/documentation/swiftui/composing_custom_layouts_with_swiftui))
#### AS-IS
![](Pasted%20image%2020250813165835.png)
- 아바타가 직선 경로로 애니메이션됨

#### TO-BE
![](Pasted%20image%2020250813170009.png)
- Podium 뷰가 Animatable을 채택하게 하고, animatableData를 offset으로 지정. position이 아닌 offset(Angle)을 기준으로 애니메이팅됨.

GQ. Layout이 뭐지?
GA. [Layout](https://developer.apple.com/documentation/swiftui/layout) 문서 읽어보기

![](Pasted%20image%2020250813173002.png)

=> ⚡ 애니메이션 모든 프레임에서 body를 호출하므로 성능 문제 발생 가능. 이게 유일한 방법일 때만 사용하는 게 좋다.

## Animation
- Animatable 데이터를 시간에 따라 보간하는 알고리즘
- Animation 분류
	- Timing curve
		![](Pasted%20image%2020250813173442.png)
		- 애니메이션 속도와 지속 시간을 결정하는 커브를 갖음
			- 커브는 비지어 곡선의 컨트롤 포인트를 통해 만듦
			- 0과 1사이의 특정한 상대적 지점에서의 값과 속도를 계산할 수 있음
				![](Pasted%20image%2020250813173713.png)
	- Spring
		![](Pasted%20image%2020250813173957.png)
		- 스프링 시뮬레이션을 통해 주어진 지점에서의 값을 게산
			- 원래 계산 방법: 질량, 강성, 감쇠
			- SwiftUI에서 도입된 방식: 기간, 바운스
			- 여기에도 커브를 사용
	- Higher order
		![](Pasted%20image%2020250813174106.png)
		- TODO: 마저 보기