## Hilt
- 안드로이드 종속 항목 삽입 라이브러리
- 프로젝트의 안드로이드 클래스의 컨테이너를 제공하고 수명 주기를 관리하여 종속 항목 제공

## Gradle Settings
- 프로젝트의 그래들 설정에 KSP와 힐트 플러그인 추가
```kts
// project/build.gradle.kts
plugins {
  id("com.google.devtools.ksp") version "1.9.22-1.0.17" apply false
  id("com.google.dagger.hilt.android") version "2.50" apply false
}
```
- 앱 그래들 설정에 KSP와 힐트 플러인 활성화
- 힐트는 자바8 이상 기능을 요구하기 때문에 그 이상으로 컴파일 옵션 설정
- 힐트와 컴파일러 라이브러리 추가
```kts
// app/build.gradle.kts
plugins {
  id("com.google.devtools.ksp")
  id("com.google.dagger.hilt.android")
}

android {
  compileOptions {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
  }
}

dependencies {
  implementation("com.google.dagger:hilt-android:2.50")
  ksp("com.google.dagger:hilt-android-compiler:2.50")
}
```

## App Container
- `@HiltAndroidApp` 어노테이션으로 앱의 수명주기를 따르는 컨테이너 생성
- 앱 기본 클래스 및 모든 힐트 코드 생성 트리거
```kt
@HiltAndroidApp
class LogApplication : Application()
```

## Android Entry Container
- `@AndroidEntryPoint` 어노테이션으로 액티비티와 프래그먼트 수명주기를 따르는 컨테이너 생성
- `@Inject` 어노테이션으로 종속 항목을 주입 받을 수 있음
  - 액티비티의 종속 항목은 `onCreate` 시점에 주입
  - 프래그먼트의 종속 항목은 `onAttach` 시점에 주입
  - `private` 한정자를 사용할 수 없음
- 프래그먼트 컨테이너를 사용하기 위해서는 반드시 해당 프래그먼트를 포함하는 액티비티도 컨테이너로 만들어야 함
```kt
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
  @Inject
  lateinit var navigator: AppNavigator
}

@AndroidEntryPoint
class LogsFragment : Fragment() {
  @Inject
  lateinit var logger: LoggerLocalDataSource
}
```

## Provide Dependencies
- 생성자에 `@Inject` 어노테이션으로 인스턴스 제공 방법(결합) 정의
- `@Singleton` 어노테이션으로 인스턴스 앱의 수명주기로 범위 정의
```kt
@Singleton
class LoggerLocalDataSource @Inject constructor(private val logDao: LogDao)
```

## Module
- 프로젝트에 포함되지 않은 클래스도 코드를 수정할 수 없기 때문에 인스턴스 제공 방법 정의가 쉽지 않음
- 인터페이스는 생성자가 없기 때문에 인스턴스 제공 방법 정의가 쉽지 않음
- `@Module` 어노테이션으로 모듈 컨테이너 정의
- `@InstallIn` 어노테이션으로 모듈 컨테이너의 연결된 구성요소를 정의
- 빌더 패턴은 `@Provide` 어노테이션으로 함수를 통해 인스턴스 제공 방법 정의 가능
```kt
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
  @Provides
  @Singleton
  fun provideDb(@ApplicationContext context: Context): AppDatabase =
      Room.databaseBuilder(context, AppDatabase::class.java, "logging.db").build()

  @Provides
  fun provideDao(db: AppDatabase): LogDao = db.logDao()
}
```
- 인터페이스는 `@Binds` 어노테이션으로 추상 함수와 구현 객체를 인자로 하여 인스턴스 제공 방법 정의 가능
```kt
@Module
@InstallIn(ActivityComponent::class)
abstract class NavigationModule {
    @Binds
    abstract fun bindNavigator(impl: AppNavigatorImpl): AppNavigator
}
```
