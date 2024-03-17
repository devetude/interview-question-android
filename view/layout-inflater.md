## Definition
- XML 레이아웃 파일을 파싱하여 트리 구조의 뷰 객체들로 인스턴스화를 위한 시스템 서비스
- 빌드 타임에 XML 레이아웃을 최적화하고 런타임에 `XmlPullParser` 통한 작업 수행
- 인플레이트에 사용된 컨텍스트에 따라서 생성되는 뷰의 룩앤필이 달라짐

## Usages
- 레이아웃 인플레이터 획득 방법은 크게 4가지 존재
  - 액티비티: 내부적으로 윈도우의 레이아웃 인플레이터 반환
  - 시스템 서비스: 시스템 서비스로 레이아웃 인플레이터 생성
  - 레이아웃 인플레이터 팩토리: 내부적으로 시스템 서비스 호출
  - 뷰 팩토리: 내부적으로 레이아웃 인플레이터 팩토리 호출
```kt
activity.layoutInflater

context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as layoutInflater

LayoutInflater.from(context)

View.inflate(context, R.layout.x, parent)
```
- 레이아웃을 생성 후 이용하는 방법
  - 액티비티: `onCreate` 호출 시점에 `setContent`의 인자로 레이아웃 리소스 아이디를 넘겨 인플레이트 이후 윈도우의 데코 뷰에 부착
  - 프래그먼트: `onCreateView`의 반환 값으로 생성한 뷰를 넘겨 프래그먼트 컨테이너에 부착
  - 뷰 홀더: `onCreateViewHolder` 호출 시점에 뷰 홀더 생성자의 인자로 생성한 뷰를 넘겨 리사이클러 뷰에 부착
```kt
override fun onCreate(savedInstanceState: Bundle?) {
  onCreate(savedInstanceState)
  setContentView(R.layout.x)
}

override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View =
  inflater.inflate(R.layout.x, container, false /* attachToRoot */)

override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): SearchAppealItemViewHolder {
  val inflater = LayoutInflater.from(parent.context)
  val view = inflater.inflate(R.layout.x, parent, false /* attachToRoot */)

  return XViewHolder(view)
}
```
