---
layout: post
title: "[iOS] 이미지 리사이징을 활용한 메모리 최적화"
description: "UIGraphicsImageRenderer & Image I/O"
author: "kyujin.kim"
date: 2020-07-12
categories: [iOS]
bigimg: assets/images/downsampling/render-after-downsampling.png
tags: [iOS, UIImage, ImageIO, Memory]
---

최근, 앨범에서 가져온 사진을 인코딩 후에 업로드하는 기능의 리팩토링을 진행했습니다. 기능 분석 중 메모리가 튀는 현상을 발견했고, 이를 해결하기 위해 사용한 방법과 경험을 기록하기 위해 이 글을 작성하게 됐습니다.

아래 그림은 리팩토링 전/후의 메모리 그래프입니다. 대충 봐도 오른쪽의 그래프가 안정적으로 보이죠? (같은 코드를 가지고 테스트 앱을 만들어 측정했습니다)

![memory-graph](/assets/images/downsampling/memory-compare.png){: width="600"}{: .center-image}

그럼 이제부터 시스템이 이미지를 처리하는 방법, 적절한 API 사용과 리사이징을 통해 안정적인 메모리 그래프를 만들 수 있는 방법을 알아보겠습니다.  
<br/>

## 이미지와 메모리
---
이미지 파일은 **JPEG**, **PNG** 등의 포맷으로 인코딩 되어있는 데이터입니다. 이 데이터는 우리에게 매우 익숙한 **UIImage**를 사용해 로드할 수 있습니다. 이미지 데이터는 디코딩을 거쳐 이미지 버퍼(메모리)에 저장되고, **UIImageView**를 통해 화면에 그려집니다.

![render-before-downsampling](/assets/images/downsampling/render-before-downsampling.png){: .center-image}

이때 **이미지 버퍼의 크기**는 뷰의 크기 또는 파일의 용량이 아닌 **이미지의 픽셀 크기와 색 영역에 따라 달라집니다**. 왜냐하면 디코딩을 거친 이미지는 아래와 형태로 메모리에 저장되기 때문이죠.

![after-decoding](/assets/images/downsampling/after-decoding.png){: width="600"}{: .center-image}

큰 이미지를 처리하는 데에는 CPU와 메모리가 많이 사용됩니다. CPU와 메모리의 점유율이 높아지면, UI를 처리하기 위한 자원이 부족하게 되어 앱이 버벅거리게 되고, 배터리도 더 많이 사용하게 됩니다. 결국 우리 앱은 삭제당하겠죠... 😭

그렇다면 작은 크기의 이미지와, 필요한 색 영역을 사용하면 자원을 절약할 수 있지 않을까요? 이제부터 그 방법을 알아보러 가시죠.  
<br/>

### 리사이징
아래 그림은 이미지 리사이즈를 거치기 전의 이미지 파이프라인입니다. 이미지 버퍼에는 원본 이미지의 데이터가 그대로 들어가 있는데, 이미지 뷰의 크기가 다소 작네요. 어찌 됐든 이미지 뷰의 크기에 맞게 그려지긴 하겠지만 불필요하게 많은 메모리를 사용하게 됩니다.

![render-before-downsampling](/assets/images/downsampling/render-before-downsampling.png){: .center-image}

두 번째 그림은 리사이즈를 거친 뒤의 이미지 파이프라인입니다. 이미지 뷰가 필요로 하는 크기에 맞추어 리사이즈를 했더니, 이미지 버퍼에 딱 필요한 만큼만 픽셀 정보가 들어가 있는 것을 볼 수 있습니다.

![render-after-downsampling](/assets/images/downsampling/render-after-downsampling.png){: .center-image}

이렇게 적절한 크기로 리사이즈만 해줘도 메모리를 많이 절약할 수 있습니다. 이미지 리사이즈는 [Image I/O](https://developer.apple.com/documentation/imageio)를 사용합니다. 

아래 샘플 코드는 이미지 데이터를 기반으로 리사이즈된 이미지(섬네일)를 생성하는 코드입니다. 여러 가지 옵션이 있으니 [레퍼런스](https://developer.apple.com/documentation/imageio/cgimagesource/image_source_option_dictionary_keys)를 꼭 확인해보는 것을 추천합니다.

```swift
func downsample(imageAt imageURL: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    let imageSource = CGImageSourceCreateWithURL(imageURL as CFURL, imageSourceOptions)!
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions = [kCGImageSourceCreateThumbnailFromImageAlways: true,
                             kCGImageSourceShouldCacheImmediately: true,
                             kCGImageSourceCreateThumbnailWithTransform: true,
                             kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels] as CFDictionary
    let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions)!
    return UIImage(cgImage: downsampledImage)
}
```

이쯤에서 리사이즈에 대한 설명을 마무리하고, 색 영역에 대해서 알아보겠습니다.  
<br/>

### 색 영역(Color Space)
이미지에는 색 영역이라는 개념이 있고, 아래 사진과 같이 색 영역에 따라 메모리 사용량이 달라집니다. 앱 개발 시 보통 SRGB format을 사용하며, 최근 출시된 애플의 기기에서는 Wide format도 사용됩니다.

![color-space](/assets/images/downsampling/color-space.png){: width="600"}{: .center-image}

다양한 색 영역 중 적절한 포맷을 선택하기 위해 Apple에서는 iOS 10에서 **UIGraphicsImageRenderer**를 추가했습니다. 항상 SRGB 포맷을 사용하던 **UIGraphicsBeginImageContextWithOptions**대신 사용할 수 있습니다.

아래 코드는 **UIGraphicsBeginImageContextWithOptions**를 사용해 원을 그리는 코드입니다. **black Color**만 사용하고 있는데, SRGB 포맷으로 이미지를 생성하면 불필요하게 많은 메모리를 사용하게 됩니다.

```swift
let bounds = CGRect(x: 0, y: 0, width:300, height: 100)
UIGraphicsBeginImageContextWithOptions(bounds.size, false, 0)
 
// Drawing Code
UIColor.black.setFill()
let path = UIBezierPath(roundedRect: bounds,
                        byRoundingCorners: UIRectCorner.allCorners,
                        cornerRadii: CGSize(width: 20, height: 20))
path.addClip()
UIRectFill(bounds)

let image = UIGraphicsGetImageFromCurrentImageContext()
UIGraphicsEndImageContext()
```
---
그렇다면 새로 추가된 **UIGraphicsImageRenderer**를 사용해보는 것은 어떨까요? 

```swift
let bounds = CGRect(x: 0, y: 0, width:300, height: 100)
let renderer = UIGraphicsImageRenderer(size: bounds.size)
let image = renderer.image { context in
// Drawing Code
    UIColor.black.setFill()
    let path = UIBezierPath(roundedRect: bounds,
                            byRoundingCorners: UIRectCorner.allCorners,
                            cornerRadii: CGSize(width: 20, height: 20))
    path.addClip()
    UIRectFill(bounds)
}
```

**UIGraphicsImageRenderer**는 **black Color**만 사용한다는 것을 알아채고 적절한 색 영역을 선택해서 우리의 메모리를 절약해줄 겁니다. 앞으로는 **UIGraphicsBeginImageContextWithOptions** 대신 **UIGraphicsImageRenderer**를 사용하는 습관을 들여야겠습니다.  
<br/>

## 마무리
---
이 글은 아래 링크된 WWDC 세션을 바탕으로 작성되었습니다. 이 글에서 소개하지 않았지만 알아두면 매우 유용한 시스템의 메모리 관리 방법과, 부드러운 스크롤링을 위한 꿀팁이 있으니 꼭 한번 보시는 것을 추천합니다. 👍  
<br/>

## 참고자료
---
- [iOS Memory Deep Dive (WWDC 2018)](https://developer.apple.com/videos/play/wwdc2018/416/)
- [Image and Graphics Best Practices (WWDC 2018)](https://developer.apple.com/videos/play/wwdc2018/219)
- [Image Resizing Techniques (NSHipster)](https://nshipster.com/image-resizing/)
 
