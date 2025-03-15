WWDC19
https://developer.apple.com/videos/play/wwdc2019/237/

## 레이아웃
### 들어가며
```swift
struct ContentView : View {
	var body: some View {
		Text("Hello World")
	}
}
```

![](Pasted%20image%2020250315164402.png)
- 파란색 선은 텍스트 뷰의 bounds(경계)이다.
- 레이아웃이란 스크린 위의 여러 바운드를 어떻게 결정하는 지에 대한 문제이다.

![](Pasted%20image%2020250315164643.png)
- 자세히 들여다 보면 세가지 뷰가 있다.
- body를 가지는 상위 레이어는 레이아웃 중립이라고 하고, body의 bounds에 따라 자신의 bounds가 결정된다.

![](Pasted%20image%2020250315165527.png)
- 그렇다면 ContentView는 레이아웃 중립이므로, 위와 같이 ContentView를 제외하고 레이아웃 작업에 영향을 미치는 두 개의 뷰로 추려볼 수 있다.

### 레아아웃 과정
1. 부모가 자식에게 부모 자신의 사이즈를 알려준다.
	- 자식의 바운드를 부모가 정하는게 아니라, 부모 뷰의 남은 공간 크기를 알려주는 것이다.
	- 위의 도식에서는 Root View가 Text에게 Safe Area를 제외한 공간 크기를 알려주게 된다.
2. 자식이 자신의 크기를 정한다.
	- 부모는 자식의 사이즈를 강제할 수 없고 자식의 결정을 존중한다.
3. 부모가 자식을 부모 좌표 공간의 어딘가에 배치한다. 2에서 자식이 정한 사이즈대로.

즉, 특정한 뷰의 정의에 자신의 사이즈를 정하는 작업이 포함된다. (2)

4. SwiftUI는 좌표를 근처 픽셀에 딱 맞게 반올림 한다. 안티앨리어싱되지 않는다.


### 예시 1
- Text에 초록색 배경을 모디파이어를 통해 추가한다면, Root View와 Text 사이에 뷰가 하나 늘어나는 것과 같다. 이때 Background는 Text와 같은 바운드를 갖는다.

![](Pasted%20image%2020250315171220.png)

- 패딩을 추가해본다면, 자동으로 적당한 패딩을 추가해준다.

![](Pasted%20image%2020250315171333.png)

![](Pasted%20image%2020250315171437.png)

- 전체 과정
	- Root View가 Background View에게 전체 사이즈를 알려준다.
	- Background View는 레이아웃 중립으로, Root View가 알려준 사이즈를 그대로 Padding View에게 알려준다.
	- Padding View는 얼마 만큼의 padding이 추가되어야 하는지를 알고, 그만큼을 제외한 사이즈를 Text에게 전달한다. (ex> padding을 10pt 만큼 주고자 했다면 전체 스크린 사이즈보다 가로 세로 10pt 씩을 줄여서 알려주는 것)
	- Text는 전달받은 사이즈를 토대로 자신의 사이즈를 정하고 Padding에게 다시 알려준다.
	- Padding은 Text의 사이즈로부터 padding 만큼을 더해 Background View에 알려준다.
	- Background View는 레이아웃 중립이어서 전달 받은 사이즈를 Root View에게 알려주면 되지만, 그 전에 Color View에게 사이즈를 알려준다. 역시 Color View도 그대로 따른다.
	- Root View는 자식 뷰를 자신의 좌표 공간에 배치한다.

### 예시 2
- 20\*20 사이즈의 이미지를 갖는 Image는 고정된 사이즈를 갖게 된다.

![](Pasted%20image%2020250315173038.png)

- 여기에 frame 모디파이어를 사용해도 이미지의 크기 자체는 고정되어 변하지 않는다. 그렇지만 프레임 뷰가 위에 생겨 bounds는 커진다.

![](Pasted%20image%2020250315173136.png)

- 프레임의 크기와 이미지의 크기가 다른데 이것은 의도된 동작이다. Frame은 그저 상위의 뷰일 뿐이고 이미지의 크기는 Image 뷰가 결정한다.
- Frame이 오토레이아웃의 제약조건이라고 착각해서는 안된다.
- (추가) 이미지를 리사이징하고 싶다면 [resizable(capInsets:resizingMode:)](https://developer.apple.com/documentation/swiftui/image/resizable(capinsets:resizingmode:))

## 스택 뷰
### HStack과 VStack
![](Pasted%20image%2020250315174248.png)

- 4개의 스택 만으로 간단하게 만들었다.

### 스택의 레이아웃
- 스택 뷰는 하나의 공간을 여러 뷰가 나눠 할당받기 때문에 위와는 조금 다르다.

![](Pasted%20image%2020250315174551.png)

1. 스택은 할당받은 공간에서 간격 만큼을 빼고 자식들의 개수로 나누어 자식들의 공간을 확보해둔다.
2. 자식 중 가장 유연성이 작은 뷰에게 나눈 공간을 제안한다.
	- Image는 고정 크기를 가지므로 가장 먼저 제안하고, Image가 자신의 크기를 결정한다. 스택은 Image가 결정한 크기만큼 여유 공간에서 제외한다.
	- 다른 Text에 대해서도 반복한다. (Delicious -> Avocaodo Toast)

- 다만 이렇게 되면 HStack에 주어진 공간이 작아지는 경우 순서에 따라 "Avocaodo Toast"가 "Avocaodo ..."와 같이 줄어든다. 
- Delicious 보다는 Avocaodo Toast가 더 중요한 주제이므로 다음과 같이 우선순위를 둘 수 있다.

![](Pasted%20image%2020250315175616.png)

- 정렬 기준을 명시할 수 있다.
- 다만 .bottom으로 명시하면 다음과 같이 행간이 포함되어 정렬이 잘 안 된 것처럼 보이는 문제가 있다.

![](Pasted%20image%2020250315175951.png)

- 이 때는 .lastTextBaseline를 사용하면 된다.

![](Pasted%20image%2020250315180031.png)

- 더 복잡하게 계산해줄 수도 있다.

![](Pasted%20image%2020250315180200.png)

- 또는 커스텀 정렬 조건을 구현할 수도 있다. AlighmentID를 상속한 이넘을 VerticalAlignment에 정의한다.

![](Pasted%20image%2020250315180447.png)

![](Pasted%20image%2020250315180417.png)

짜잔

## 그래픽

- SwiftUI에서는 Shape도 뷰이다

![](Pasted%20image%2020250315204053.png)

- 다양한 모디파이어로 커스텀 가능

![](Pasted%20image%2020250315204143.png)

- 다양한 스타일

![](Pasted%20image%2020250315204254.png)

- 그라데이션도 간단하게

![](Pasted%20image%2020250315204337.png)

![](Pasted%20image%2020250315204356.png)

- 커스텀 쉐입

![](Pasted%20image%2020250315204713.png)

![](Pasted%20image%2020250315204806.png)

![](Pasted%20image%2020250315204840.png)
베지어패스로 호를 그린다

![](Pasted%20image%2020250315204925.png)
애니메이션에 사용할 속성들
시스템이 보간하여 애니메이션을 구현할 수 있는 부동소수점수들

이후 ZStack에서 함께 렌더링

![](Pasted%20image%2020250315205423.png)

탭하면 삭제되도록 .tapAction 모디파이어 사용
트랜지션 애니메이션은 아래에서 계속 구현 (scaleAndFad)


![](Pasted%20image%2020250315205552.png)
애니메이션 구현을 위해서는 두 가지 상태 정의가 필요 (isActive == true || false)

![](Pasted%20image%2020250315205716.png)
![](Pasted%20image%2020250315205736.png)

-  drawing group을 이용해 전체 쉐입을 하나의 NSView로 그려 성능 문제 최적화
