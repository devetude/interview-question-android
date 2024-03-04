## Definition
- 클래스가 다른 클래스를 참조해야 할 경우 필요한 클래스를 종속 항목이라 함
- 필요한 클래스를 참조하는 방법을 디펜던시 인젝션이라 함
  - 필요한 종속 항목 생성
  - 매개변수 전달
  - 다른 객체(서비스 로케이터 등)로 주입
```kt
class Car {
  // 직접 생성
  private val engine: Engine = Engine()

  fun start() = engine.start()
}
```
```kt
// 매개변수 전달
class Car(private val engine: Engine) {
  fun start() = engine.start()
}
```
```kt
object ServiceLocator {
  fun getEngine(): Engine = Engine()
}

class Car {
  // 다른 객체로 주입
  private val engine: Engine = ServiceLocator.getEngine()

  fun start() = engine.start()
}
```

## Limitations
- 직접 필요한 종속 항목을 생성하면 의존성이 강해지고 재사용성 및 확장성이 떨어짐
- 다른 객체로부터 가져오는 방법을 이용하더라도 싱글톤 사용으로 인해 테스트하기 어렵게 만듬

## Hilt Library
- 종속 항목 구현을 쉽게 교체할 수 있는 형태로 만들 수 있음
- 종속 항목은 검증 가능한 요소가 되어 컴파일 시간에 확인 가능
- 주입을 받는 클래스는 종속 항목을 관리하지 않아 다양한 구현을 전달하여 테스트 해볼 수 있음
