## Definition
- 운영체제의 작업 스케쥴링 단위
- 메모리에 적재되어 실행 중인 프로그램

## Features
- CPU 시간, 주소 공간, [메모리 영역](https://github.com/devetude/interview-question-android/blob/master/process/memory-area.md)을 운영체제로부터 할당 받음
- 독립된 주소 공간에서 실행되어 다른 프로세스가 또 다른 프로세스의 메모리 영역 접근 불가
- 다른 프로세스의 자원에 접근하기 위해서 [IPC](https://github.com/devetude/interview-question-android/blob/master/process/ipc.md) 필요
- 운영체제에 의해 컨텍스트 스위칭 가능
- 최소 한개 이상의 스레드를 관리
- 기본적으로 앱은 하나의 프로세스와 스레드 보유
- 앱의 설정에 따라 다수의 프로세스로 구성 가능
- 안드로이드 운영체제는 메모리를 회수할 필요가 있을 경우에만 우선순위에 따라 프로세스 종료

## Levels
- 포그라운드 프로세스
  - 사용자가 현재 진행중인 작업에 필요한 프로세스
    - onResume 상태의 액티비티
    - onReceive 상태의 브로드캐스트 리시버
    - onCreate와 onDestory 상태 사이의 서비스
- 가시적 프로세스
  - 사용자가 현재 진행중이지 않지만, 실행중인 사실이 인지되는 프로세스
    - onPause 상태의 액티비티
    - startForegroundService로 실행된 서비스
- 서비스 프로세스
  - 사용자에게 보이지 않고 실행중인 프로세스
    - startService로 실행된 서비스
  - 중요도가 낮아지면 백그라운드 프로세스 레벨로 이전
- 백그라운드(캐시) 프로세스
  - 현재 필요하지 않은 프로세스
  - 메모리가 부족할 경우 우선순위에 따라 운영체제에 의해 종료 가능
    - onStop 상태의 액티비티
    - 오랜 시간동안 동작하는 백그라운드 서비스
