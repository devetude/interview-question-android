## Definition
- 코루틴이 실행될 때 필요한 여러가지 실행 환경 정보들의 집합
- `Job`, `Dispatcher`, `CoroutineName`, `CoroutineExceptionHandler`, key-value 형태의 기타 요소를 결합하여 구성

## Features
- 내부적으로 `CoroutineContext.Element` 즉, key-value 쌍의 집합으로 구성
- 존재하지 않는 키에 값을 추가하면 해당 키로 값이 추가되며, 이미 있는 키에 다른 값을 추가하면 덮어씀
- `+` 연산자를 통해 요소를 결합하는 형태로 구성할 수 있음

```context.kt
val context = Job() + SupervisorJob() + Dispatchers.Main + CoroutineName("devetude")
```

- 각각의 코루틴은 각각의 컨텍스트를 지님
- 부모 컨텍스트 안에서 `launch` 함수를 통해 실행된 자식 코루틴의 경우 기본적으로 부모의 컨텍스트를 상속
- 추가로 특정 요소에 대한 부분을 덮어쓸 수 있음

```context.kt
val job = launch(Dipatcher.Default + CoroutineName("parent")) {
  // Dipatcher.Default + CoroutineName("parent")

  launch(CoroutineName("child")) {
    // Dispatcher.IO + CoroutineName("child")
  }
}
```
