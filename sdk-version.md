## Min SDK Version
- 앱이 실행하는 데 필요한 최소 API 수준
- 기기에 설치되어있는 API 수준이 최소 API 수준보다 낮은 경우 앱 설치 불가
- 낮게 설정할 경우 하위 호환성이 증가하나 상위 API를 사용할 경우 보다 많은 코드 분기 필요
- 높게 설정할 경우 상위 API를 위한 코드 분기는 줄어드나 설치할 수 있는 사용자가 줄어듬

## Max SDK Version
- 앱이 실행하는데 필요한 최대 API 수준
- 과거 낮은 버전의 안드로이드에서 유효성 검사를 위해 사용되었으나 현재는 사용하지 않음

## Target SDK Version
- 앱이 실행될 때 사용할 API 수준
- 개발자가 해당 API 수준에서 앱이 잘 동작하는 것을 의미
- API 수준을 변경할 경우 앱의 룩앤필, 동작, 보안적 변경점이 반영
- 구글 플레이 스토어에 앱을 게시하기 위해서는 주기적으로 최신 API 수준을 사용하도록 해야 함

## Compile SDK Version
- 앱을 컴파일할 때 사용할 API 수준
- 개발자가 사용 불가능한 API를 실수로 사용하는 부분을 검증하는 용도
- 보통 Target SDK 버전과 동일하게 설정

![sdk_version](https://github.com/devetude/interview-question-android/blob/master/img/sdk-version.jpg?raw=true)
