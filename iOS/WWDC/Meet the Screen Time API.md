WWDC21
https://developer.apple.com/videos/play/wwdc2021/10123/

## 구성
1. Managed Settings: 제한
2. Family Controls: 인가
3. Device Activity: 사용량

## 데모 앱
1. Family Control을 설정하고 인가 받음
2. 사용하지 못하게 하고 싶은 앱을 지정
3. 시간 사용 조건을 만족하면 제한을 해제

## Family Control
Xcode의 Signing & Capabilities에서 Family Control을 추가

![](Pasted%20image%2020250519230421.png)

- Family Control 활성화 시,
	- iCloud 로그아웃 불가능
	- 온디바이스 웹 컨텐트 필터가 작동. 앱이 웹 트래픽을 감시할 수 있음

## Device Activity
- Device Activity를 통해앱 사용 제한 등 코드를 백그라운드로 실행 가능
- 특정 액티비티에 대한 시작, 끝 핸들링
	![](Pasted%20image%2020250519231003.png)
	- 우선 Device Activity 스케쥴 설정 전까지 구현하지 않고 넘어감
- DeviceActivityName은 해당 액티비티에 대한 구별자
	![](Pasted%20image%2020250519231030.png)
		- 스케쥴을 만들고 daily에 대해 지정한 후 모니터링 시작

## 뷰 구현
- 뷰 단에서는 .familyActivityPicker 모디파이러를 통해 제한할 앱을 고르는 화면을 띄울 수 있음
	![](Pasted%20image%2020250519231447.png)
	![](Pasted%20image%2020250519231554.png)
	- 피커에서는 선택하면 해당 카테고리를 가리키는 토큰이 반환됨

## ManagedSettings
![](Pasted%20image%2020250519231703.png)
- ManagedSettingsStore을 생성
- shield의 applications를 핸들링하여 제한할 앱을 추가하거나 추가된 앱을 제한 해제

## DeviceActivityEvent
- 반대로 DeviceActivityEvent를 통해 앱을 제한하는게 아니라 사용할 수 있게 함

	![](Pasted%20image%2020250519232257.png)


	![](Pasted%20image%2020250519232327.png)
## 실드 화면은 커스텀 가능함
![](Pasted%20image%2020250519232432.png)

## ShieldConfiguration

![](Pasted%20image%2020250519232532.png)

![](Pasted%20image%2020250519232621.png)