WWDC22
https://developer.apple.com/kr/videos/play/wwdc2022/10126/
관련 세션
	[Bring your world into augmented reality](Bring%20your%20world%20into%20augmented%20reality.md)

## 4K 비디오
- 기본: 비디오 캡쳐 시 4K 해상도로 캡쳐한 후, 비닝(2\*2 픽셀을 취해 색상 평균값을 낸 뒤 하나의 픽셀로 취급)하여 FHD 급으로 제공
- 변경: 옵션에 따라 비닝을 거치지 않고 원본 4K 해상도로 다룰 수 있음

## 카메라 개선
- 고해상도 배경 이미지: AVSession에서 기본 해상도의 이미지를 따로 촬영 가능. 비디오 캡쳐가 중단되지 않고 동시 수행됨
- HDR: HDR 이미지 캡쳐 가능
- AVCaptureDevice 접근: 노출, 화이트밸런스 등 조정 가능
- Exif 태그 획득

## 평면 앵커
- 평면 앵커와 기본 평면 기하를 보다 명확하게 분리해야 한다는 의견이 많았음
- ARKit 6에서 완전히 분리됨

## 모션 캡쳐
- Ear joint tracking (2D)
- Improved pose detection (2D)
- More temporal consistency (3D)
- Better occlusion handling (3D)

## LocationAnchors
- 몇 개의 도시 추가
- ﻿﻿Vancouver, Toronto, Montreal (Canada)
- ﻿﻿Singapore
- ﻿﻿Fukuoka, Hiroshima, Osaka, Kyoto, Nagoya, Yokohama, Tokyo (Japan)
- ﻿﻿Melbourne, Sydney (Australia)

- Available later this year (2022)
- ﻿﻿Auckland (New Zealand)
- ﻿﻿Tel Aviv-Yafo (Israel)
- ﻿﻿Paris (France)

