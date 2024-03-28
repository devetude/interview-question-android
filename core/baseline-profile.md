## Objective
- 앱 또는 라이브러리에서 핵심 코드의 베이스라인 프로파일 수집
- 수집된 베이스라인 프로파일을 기반으로 AOT 컴파일을 통해 목적 코드로 컴파일
- 앱 최초 실행(설치 및 업데이트)시 런타임에 수행되는 JIT 컴파일 단계를 회피하여 성능 개선
- 최적화 된 코드는 별도의 `classes.dex` 파일에 배치

## Advantages
- 앱 시작, 화면 이동, 스크롤 등 사용자가 앱을 처음 시작할 때 자주 사용하는 기능에 대한 성능 개선
- 클라우드 프로파일(구글에서 자동 집계하는 프로파일)에만 의존하여 최적화하지 않고 보다 빠르게 최적화 가능
  - 프로파일을 수집하기 위해서는 보통 1주 이상의 시간이 필요
  - 프로파일이 생성되기 전에 앱 설치 또는 업데이트하는 경우가 많기 때문에 최적화 기회를 놓칠 수 있음
  - 런타임에 JIT 컴파일을 통한 최적화 또는 유휴 상태에서 최적화된 형태의 dex 파일로 변경되긴 함

![baseline-profile](https://github.com/devetude/interview-question-android/blob/master/img/baseline-profile.jpg?raw=true)

## Create Baseline Profiles
1. 기준 프로파일 모듈 생성
- `File > New > New Module > Templates > Baseline Profile Generator`
```kts
// settings.gradle.kts
include ":baseline-profile"
```
```kts
// build.gradle.kts
plugins {
  id("androidx.baselineprofile")
}

dependencies {
    baselineProfile(project(":baseline-profile"))
}
```
2. 사용자가 자주 사용할 기능에 대한 JUnit 테스트 추가
```kt
@LargeTest
@RunWith(AndroidJUnit4::class)
class BaselineProfileGenerator {
    @get:Rule
    val rule = BaselineProfileRule()

    @Test
    fun generate() = rule.collect(
        packageName = InstrumentationRegistry.getArguments().getString("PACKAGE_NAME")
            ?: throw Exception("targetAppId not passed as instrumentation runner arg"),
        includeInStartupProfile = true
    ) {
        pressHome()
        startActivityAndWait()

        // TODO: Write more interactions to optimize advanced journeys.
    }
}
```
3. 테스트 실행 결과로 만들어진 프로파일을 `src/main` 디렉토리 밑으로 이동
- 결과 저장 경로: `src/<variant>/generated/baselineProfiles/baseline-prof.txt`
