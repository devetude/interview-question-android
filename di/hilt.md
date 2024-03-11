## Hilt
- 안드로이드 종속 항목 삽입 라이브러리
- 프로젝트의 안드로이드 클래스의 컨테이너를 제공하고 수명 주기를 관리하여 종속 항목 제공
- 빌드 시간에 안드로이드를 위한 대거 구성요소를 생성 및 유효성 검사
- 런타임에 객체에 필요한 종속 항목을 자동으로 생성 및 삽입

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
- `@AndroidEntryPoint` 어노테이션으로 액티비티, 프래그먼트, 뷰, 서비스, 브로드캐스트 리시버의 수명주기를 따르는 컨테이너 생성
- `Hilt_X`와 같이 주입을 위한 추상 클래스가 자동으로 생성되고 힐트 그래들 플러그인에 의해 빌드 시간에 원본 클래스가 상속하는 형태로 변경
  - 종속 항목 주입을 위한 `inject` 함수 자동 생성
  - 뷰 모델 생성에 필요한 `getDefaultViewModelProviderFactory` 함수가 자동 생성
- `@Inject` 어노테이션으로 종속 항목을 주입
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
```
```kt
@AndroidEntryPoint
class LogsFragment : Fragment() {
  @Inject
  lateinit var logger: LoggerLocalDataSource

  @Inject
  lateinit var dateFormatter: DateFormatter
}
```

## View Model Container
- `@HiltViewModel` 어노테이션으로 뷰 모델 수명주기를 따르는 컨테이너 생성
- 생성에 필요한 `X_Factory` 클래스가 자동으로 생성
- `@Inject` 어노테이션으로 종속 항목을 주입
  - 종속 항목은 `Provider`를 이용하는 형태로 뷰 모델 생성 시점에 주입
```kt
@HiltViewModel
class MainViewModel @Inject constructor(private val repository: Repository) : ViewMode()
```

## Provide Dependencies
- 생성자에 `@Inject` 어노테이션으로 인스턴스 제공 방법(결합) 정의
- `@Singleton` 어노테이션으로 인스턴스 앱의 수명주기로 범위 정의
```kt
@Singleton
class LoggerLocalDataSource @Inject constructor(private val logDao: LogDao)
```

## Module
- 프로젝트에 포함되지 않은 클래스는 코드를 수정할 수 없기 때문에 인스턴스 제공 방법 정의가 어려움
- 인터페이스는 생성자가 없기 때문에 인스턴스 제공 방법 정의가 어려움
- `@Module` 어노테이션으로 모듈 컨테이너 정의
- `@InstallIn` 어노테이션으로 모듈 컨테이너의 연결된 구성요소를 정의
- 빌더 패턴은 `@Provide` 어노테이션으로 함수를 통해 인스턴스 제공 방법 정의 가능
  - 대표적으로 레트로핏 객체나 룸 데이터베이스 객체를 제공해야 할 경우
- `X_YFactory` 클래스가 자동 생성되어 필요한 곳에서 `get` 함수를 통해 종속 항목을 가져감
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

@ActivityComponent
class AppNavigatorImpl @Inject constructor() : AppNavigator
```

## Qualifiers
- 한정자를 이용하여 동일한 유형에 대해 여러 결합을 정의 가능
  - 특정 유형에 대해 여러 결합이 정의되어 있을 때 그 유형의 특정 결합을 식별하는 데 사용하는 어노테이션
  - 예를들어 2가지 다른 레트로핏 객체를 제공해야할 경우 제공하는 쪽과 제공받는 쪽 모두 불분명
  - 한정자 어노테이션을 정의하여 제공하는 쪽과 제공받는 쪽에 사용하면 의도를 분명하게 전달 가능
```kt
// RetrofitQualifier.kt
@Qualifier
@Retention(AnnotationRetention.BINARY)
internal annotation class ARetrofit

@Qualifier
@Retention(AnnotationRetention.BINARY)
internal annotation class BRetrofit
```
```kt
// RemoteSourceModule.kt
@Provides
@Singleton
@ARetrofit
fun provideARetrofit(): Retrofit

@Provides
@Singleton
@BRetrofit
fun provideBRetrofit(): Retrofit
```
```kt
// RemoteSource.kt
@Singleton
class RemoteSource @Inject constructor(
  @ARetrofit private val aRetrofit: Retrofit,
  @BRetrofit private val bRetrofit: Retrofit
)
```

## Predefined Qualifiers
- 사전에 정의된 앱 혹은 액티비티 컨텍스트 한정자를 이용하여 주입 가능
- 앱 컨텍스트의 경우 `@ApplicationContext` 어노테이션, 액티비티 컨텍스트의 경우 `@ActivityContext` 어노테이션 이용
  - 모듈에서 `@Provides` 어노테이션으로 빌더로 객체를 생성할 때 컨텍스트를 주입해야 하는 경우
  - 클래스에서 `@Inject` 어노테이션으로 생성자로 객체를 생성할 때 컨텍스트를 주입해야 하는 경우
```kt
@Provides
@Singleton
fun provideMemoryDatabase(@ApplicationContext context: Context): MemoryDatabase =
    Room.inMemoryDatabaseBuilder(context, MemoryDatabase::class.java).build()
```
```kt
class DeviceManager @Inject constructor(@ApplicationContext private val context: Context)
```

## Scopes
- 안드로이드 클래스의 수명 주기에 따라 생성된 구성요소 클래스의 인스턴스를 자동으로 만들고 제거

|        **컴포넌트**       |     **생성 시점**     |             **소멸 시점**             |
|:-------------------------:|:---------------------:|:-------------------------------------:|
|     SingletonComponent    | Application#onCreaate |         Application destroyed         |
| ActivityRetainedComponent |  Activity#onCreate()  | Activity#onDestroy() (구성 변경 제외) |
|     ViewModelComponent    |   ViewModel created   |          ViewModel destroyed          |
|     ActivityComponent     |  Activity#onCreate()  | Activity#onDestroy() (구성 변경 포함) |
|     FragmentComponent     |  Fragment#onAttach()  |          Fragment#onDestroy()         |
|       ViewComponent       |      View#super()     |             View destroyed            |
| ViewWithFragmentComponent |      View#super()     |             View destroyed            |
|      ServiceComponent     |   Service#onCreate()  |          Service#onDestroy()          |
