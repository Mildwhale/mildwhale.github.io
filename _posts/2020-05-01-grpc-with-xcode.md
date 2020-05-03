---
layout: post
title: "[iOS] gRPC의 사용법 알아보기"
description: "gRPC를 Xcode에서 사용하는 방법"
author: "kyujin.kim"
date: 2020-05-01
categories: [iOS]
tags: [iOS, gRPC]
---

이 글은 gRPC를 Xcode에서 사용하기 위해 겪었던 삽질을 기록하고 공유하기 위해 작성되었습니다. gRPC의 이론적인 내용은 따로 하지 않을 것이니 궁금하신 분은 [gRPC.io](https://grpc.io)를 참고해주세요.

![grpc-logo](/Assets/images/grpc/grpc-icon-color.png){: width="400"}{: .center-image}  
<br/>

## 컴파일러와 플러그인 설치
---
gRPC는 [Google ProtoBuffer](https://developers.google.com/protocol-buffers/)를 사용하여 proto파일에 기능을 정의하고, 원하는 언어로 변환하여 사용합니다. 하나의 proto를 사용하기 때문에 플랫폼이 다르더라도 동일한 인터페이스로 통신할 수 있는 장점이 있습니다.

우리는 iOS 앱을 만들어야 하기 때문에, **proto**파일을 **swift**로 변환해야 합니다. 컴파일러와 swift 플러그인은 **swift-protobuf**, grpc 플러그인은 **grpc-swift**에서 설치할 수 있습니다.  

먼저 **Brew**를 사용해 swift-protobuf를 설치합니다. 터미널을 열고 아래 명령어를 실행해주세요. proto 파일의 컴파일러인 **protoc**와 swift로 변환해주는 플러그인 **protoc-gen-swift**가 설치됩니다. 
```
brew install swift-protobuf
``` 

다음으로 **grpc-swift** 플러그인을 설치합니다. grpc-swift는 gRPC 스펙으로 정의된 proto파일을 swift로 변환해주는 플러그인입니다.

[**grpc-swift**](https://github.com/grpc/grpc-swift) 소스를 checkout 하고, 폴더로 이동하여 아래 명령어를 실행합니다.
```
make plugins
```

make 작업이 완료되면 **protoc-gen-swift**, **protoc-gen-grpc-swift** 파일이 생성됩니다. 필요한 파일은 `protoc-gen-grpc-swift`입니다. 이 파일을 편하게 사용하기 위해 `/usr/local/bin` 폴더에 복사해 넣습니다. 파일을 복사하는 대신, 환경변수($PATH)에 경로를 추가해도 됩니다.  
<br/>

## proto를 swift로 변환하기
---
이제 proto 파일을 swift로 변환해볼 차례입니다. 터미널을 열고 **grpc-swift** 프로젝트의 `/Sources/Examples/Echo/Model` 경로로 이동합니다. 먼저, `echo.proto`파일을 제외한 나머지 파일을 삭제하고, 아래 명령어를 실행합니다.
```
protoc echo.proto --swift_out=. --grpc-swift_out=.
```

컴파일이 성공하면 `echo.grpc.swift`와 `echo.pb.swift`파일이 생성됩니다. 👏

다음으로 Xcode 프로젝트를 열고 SPM 의존성에 `grpc-swift`를 추가합니다. 그리고 위에서 컴파일했던 echo.*.swift 파일들을 프로젝트에 추가하고, Build를 눌러 설정이 잘 되었는지 확인해봅니다. 설정이 잘 되었다면 빌드가 성공해야 합니다.  
<br/>

## Trouble Shooting
---
### 파일명 중복
protoc는 proto파일의 이름을 기준으로 swift파일을 생성하기 때문에, 같은 이름의 swift파일이 존재할 수 있습니다. swift파일의 이름이 같으면 Xcode의 빌드가 되지 않을 수 있는데, 다행히도 protoc 플러그인은 이런 문제를 해결할 수 있는 옵션을 제공합니다.
```
protoc echo.proto --swift_out=. --grpc-swift_out=. --swift_opt=FileNaming=PathToUnderscores
```

`--swift_opt=FileNamin=PathToUnderscores` 옵션은 변환되는 파일의 이름에 해당 proto 파일이 위치한 경로를 언더스코어로 구분하여 추가해줍니다. 예를 들어 아무런 옵션이 없을 때 파일명이 `echo.pb.swift` 였다면, 위 옵션을 추가했을 때는 `a_b_echo.pb.swift`와 같은 형태가 된다는 것입니다.
<br/>

### import "#path#" was not found
proto파일은 다른 proto파일을 import 할 수 있습니다. 따라서 import 한 파일을 찾을 수 없는 경우, 컴파일이 되지 않습니다. 이럴 때에는 `--proto_path`를 추가하고 proto파일의 최상위 경로를 지정하면 해결할 수 있습니다.
```
protoc echo.proto --swift_out=. --grpc-swift_out=. --proto_path=/protoc-service-repo/protobuf
```

이와 같이 protoc 플러그인은 다양한 옵션을 제공합니다. 전체 옵션은 [이곳](https://github.com/grpc/grpc-swift/blob/master/docs/plugin.md)에서 확인하실 수 있습니다.  
<br/>

## 마무리
---
지금까지 proto 파일을 swift 파일로 변환하고, Xcode에 추가하여 사용하는 기본적인 방법을 소개했습니다. gRPC API를 호출하는 방법은 [이곳](https://github.com/grpc/grpc-swift/blob/master/docs/basic-tutorial.md)을 참고하시면 됩니다.

gRPC를 적용하는 내내, 자동화에 대한 고민을 계속했습니다. 그래서 최초 설정과 protoc 컴파일에 대한 스크립트를 작성하여 이 부분을 자동화했습니다. 하지만 변환된 파일을 프로젝트에 추가하는 과정은 아직 손으로 직접 해야 하는데요, 이 부분은 [XcodeGen](https://github.com/yonaskolb/XcodeGen)을 사용하면 해결할 수 있을 것 같습니다. 만약 XcodeGen을 적용하게 된다면 다음 글의 주제가 될 것 같네요.

저도 아직 모르는 부분이 많기 때문에, 더 좋고 효율적인 방법이 있다면 꼭 알려주세요!  
읽어주셔서 감사합니다! 😉  
<br/>

## 참고자료
---
- [https://github.com/apple/swift-protobuf](https://github.com/apple/swift-protobuf)
- [https://github.com/grpc/grpc-swift](https://github.com/grpc/grpc-swift)
- [https://nesoy.github.io/articles/2019-07/RPC](https://nesoy.github.io/articles/2019-07/RPC)