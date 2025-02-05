WWDC22
https://developer.apple.com/videos/play/wwdc2022/10128
관련 세션
	[ARKit 6 소개](ARKit%206%20소개.md)

## Object Capture recap
- 여러 앵글의 사진을 통해 3D 모델을 만들 수 있는 기능 (Photogrammetry API)
- Create 3D models with Object Capture (WWDC21) 참고

## ARKit camera enhancements
- 촬영한 이미지의 해상도가 높을수록 3D 모델의 품질이 좋다
- ARKit의 High-resolution background photos 업데이트와 연계해 향상된 사진 촬영 경험 제공
	- Captures photos at full camera resolution
	- Uninterrupted video stream of current ARSession
	- Provides EXIF tags in photos

```swift
if let hiResCaptureVideoFormat = ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing {
    // Assign the video format that supports hi-res capturing.
config.videoFormat = hiResCaptureVideoFormat
}
// Run the session.
session.run(config)

session.captureHighResolutionFrame { (frame, error) in
   if let frame = frame {
      // save frame.capturedImage 
      // …   
   }
}
```

- 초점, 화이트밸런스, 노출 등 수동 설정이 필요한 경우 AVCaptureDevice에 접근해 정밀 설정

## Best practice guidelines
- 스캔할 물체 고르기
	- Good object characteristics
		- ﻿﻿Sufficient texture detail
		- ﻿﻿Minimal reflective surfaces
		- ﻿﻿Rigid (when flipping)
		- ﻿﻿Limited fine structure
- 촬영 조건
	- Ideal environment characteristics
		- ﻿﻿Diffuse, consistent lighting
		- ﻿﻿Enough space around object

- 출력 품질
	![](Pasted%20image%2020250205123935.png)
- USDZ 포맷
## End-to-end workflow
- 실제 나무로 체스 말을 만들고, Object Capture를 통해 스캔
- 3D 체스 작성
