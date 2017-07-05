# Framework 만들기

https://www.raywenderlich.com/126365/ios-frameworks-tutorial

위의 튜토리얼에 잘 설명되어 있다. 기존의 프로젝트에서 공통적으로 사용될 코드를 추출하여 이를 프레임워크로 만들고, 이 프레임워크를 다시 CocoaPods로 만드는 과정이 나와있다.



### 재사용 가능한 코드를 모아 프레임워크화 하기

1. framework project를 하나 생성한다.
2. 위에서 생성한 프로젝트의 xcodeproj파일을 원본 프로젝트의 sub-project로 추가.
3. 원본 프로젝트에 있는 공통의 소스코드를 빼내어 이 서브프로젝트에 복사함. (원본에서는 해당파일 삭제)
4. 서브 프로젝트의 product폴더에서 framework파일을 찾아냄(빨간색일것임) 이를 드래그하여 원본 프로젝트의 target> general tab > embedded binaries에 추가시킴. 
5. 서브프로젝트로 가서 외부에 공개할 코드를 public으로 만들어야 함. 주요 api 메소드, init메소드, 주요 클래스, @IBInspectable 및 각종 프로퍼티들이 그 대상임. 
6. 이제 해당 framework를 import하여 쓰면 된다.



### Framework를 CocoaPods로 변환하기

1. 위에서 추가했던 xcodeproj파일의 위치로 이동하여 터미널을 열자.

2. ```shell
   # pod spec 작성하자. 자세한 수정법은 위의 링크에 나옴.
   > pod spec create 서브프로젝트명

   # framework 자체가 다른 서드파티 cocoapods에 의존성을 갖고있다면 다음과 같은 형식으로 작성한다.
   s.dependency "PKHUD"

   # 단, swift 버전을 설정해주는 방법은 위의 링크가 잘못되어있다.
   # podspec이 있는 동일한 path에 아래와 같이 별도의 파일을 생성해줘야 한다.
   > touch .swift-version
   # 이 파일을 에디터로 열고 간단히 버전을 기록하면 된다. 예를 들어 3.0 이라고 쓰면된다.

   # pod spec을 제대로 설정했는지 확인하려면 다음을 실행해본다.
   > pod spec lint
   ```

3. 이제 원본 프로젝트의 root 위치로 이동하여 터미널을 열자. 

   ```shell
   # Podfile을 생성하고 적당한 위치에 다음과 같이 작성.
   > pod 'Pod이름', :path => '~/로컬경로'
   ```

   위의 Pod는 우리가 일반적으로 third party에서 다운받는 Pod가 아니다. 나 혼자만 로컬에서 사용하는것이다. 이렇게 로컬에서 사용하는 소스코드를 이용하여 pod를 만들면 이때의 pod project내부의 파일들은 다 symbolic link로 생성된다. 이 말은 내가 해당 파일을 수정하면 원래의 파일도 수정되는것이다. 이 덕분에 내가 개발하는 도중에도 pod를 업데이트하는것이 가능해지는 것이다.  이런 파일들은 Development Pods라는 폴더 하에 위치하게 되는데 이 역시도 바로 개발중인 pod라는 의미다.

4. `pod install`



### 로컬 Pods를 publish해보자.

> 자세한 내용은 위의 튜토리얼에 나와있으니 참고.





# CocoaPods

라이브러리를 의존성에 맞게 설치할수 있게 도와주는 툴
<설치방법> 
​																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																																											http://cafe.naver.com/mcbugi/234080?social=1

0. 사전준비 : 최신버전의 cocoapods로 업데이트 하기.

ruby gem 업데이트: 
`sudo gem update --system`

cocoapods 버전 업데이트: 
`sudo gem install cocoapods`

pod의 버전 확인은 
`pod --version`



1. 프로젝트의 디렉토리로 감.

2. 그리고 거기에 파일명 Podfile 이라는 텍스트 파일을 만들고 아래와 같은 형식으로 원하는 라이브러리를 쓴다. (Podfile 작성법은 하단 참조)
   `touch Podfile`
   `nano Podfile`

3. pod install

4. 만약 다른 오픈소스를 추가하고싶다면 위의 Podfile을 수정한 후 pod update를 실행하면됨.

5. pod가 인스톨되어있다면 반드시 .xcworkspace 파일로 프로젝트를 열어야 한다.

## Podfile 작성법
https://www.raywenderlich.com/97014/use-cocoapods-with-swift

> You can make CocoaPods integrate to your project via frameworks instead of static libraries by specifying use_frameworks!
```shell
platform :ios, "8.0"        # project의 iOS버전
use_frameworks!             # static library 대신 프레임워크 사용 

target "TargetName" do
	inherit! :search_paths      # 이거 안써주면 unit test target에서 import가 제대로 안될수 있음.
	pod 'SBJson'
	pod 'AFNetworking'
	pod 'MBProgressHUD', '~> 0.9.1' # ~> 표의 의미는 해당 버전과 호환되는 최상위 버전까지 업데이트 하라는 뜻. 
end
```

(만약 라이브러리 이름을 모른다면 콘솔창에 pod list라고 쳐서 직접 찾거나 http://cocoapods.org 에서 검색.)



소스 버전컨트롤을 할때는 pod 파일들도 함께 commit하자. branch를 스위칭할때 혼동을 줄수있기때문에 아예 통째로 커밋하는게 낫다.



혹시 설치를 했는데도 프레임워크 인식을 못한다면
⌘+⌥+⇧+B 를 눌러 build clean을 해보자.
