# DI-and-Koin
이번 파트는 DI가 무엇인지 정확히 기록하고 그에 사용할 Koin에 대한 설멍을 담은 파트이다.
***
## :wrench: Android개발자가 본 DI(Dependency Injection) 란??
우리는 단일클래스로 개발을 할 수 없기때문에 클래스간의 참조는 필수적으로 일어나며 이때 의존성이 발생합니다. 예를들어 Car라는 클래스(이하 Car)는 Engine이라는 클래스(이하 Engine)를 참조합니다.(엔진이 없는 차는 없기때문에) 이때 Engine을 얻는 방법은 3가지정도로 구분됩니다. 다른말로 Engine과 Car간의 Dependency를 생성하는 방법은 3가지정도로 구분된다라고 할 수 있습니다.

1. Car 내부에서 Engine을 생성합니다.
2. 다른 곳에 저장되어 있는 객체를 가져옵니다.
3. 외부에서 생성된 객체를 매개변수로 제공받습니다.(DI)

#### 1.Car 내부에서 Engine을 생성하는 경우
```kotlin
fun main{
    var car = Car()
    car.start()
}

class Car {
    private val engine = Engine()

    fun start() {
        engine.start()
    }
}

class Engine {
    fun start() {}
}

```
<img src = "https://user-images.githubusercontent.com/48902047/142577004-bdebb030-d843-4fff-8056-08b7d3fddc3b.png">
어떤 단점이 있을까요?? Car는 Engine을 항상 가지고 있습니다. 즉 Car는 Engine에 강하게 종속되어 있으며 Car만 따로 분리하여 테스트가 어렵게 만듭니다.(Engine을 다른 가상의 값으로 넣어가며 테스트가 불가능합니다.) 우리는 Car와 Engine간의 결합도를 낮춰 테스트 더블이 가능하도록 만들고 싶습니다.

#### 2.Car 내부에서 getEngine통해 Engine를 가져오는 경우

```kotlin
fun main{
    var car = Car()
    car.start()
}

class Car {
    private val engine = context.getEngine()

    fun start() {
        engine.start()
    }
}

class Engine {
    fun start() {}
}
```
우리는 context객체에서 getSystemService(서비스명)을 호출하여 특정 서비스 객체를 가져오는 코드에 익숙합니다. 이것의 단점은 뭘까요?? 일반적인 네이밍 규칙에 따르면 get(ClassName)은 SingleTone Pattern을 뜻합니다. 즉 getSystemService로 생성된 SystemService클래스는 사용하지 않더라도 계속 메모리에 남게되며 메모리 누수가 발생할 가능성이 있습니다. 편하고 빠를수도 있다는 장점이 있지만 “1.Car 내부에서 Engine을 생성하는 경우”와 같은 단점도 가지고 있습니다.

##### *다들 아시겠지만 참고로 onClick, onCreate 등등 앞에 on이 붙으면 콜백함수를 뜻합니다.*

#### 3.Engine를 외부에서 선언한 후 Car에 넘겨주는 경우

```kotlin
fun main{
    var engine = Engine()
    var car = Car(engine)
    car.start()
}

class Car(private var engine : Engine) {
    fun start() {
        engine.start()
    }
}

class Engine {
    fun start() {}
}
```

<img src = "https://user-images.githubusercontent.com/48902047/142577709-d09036dc-b284-4ad1-83bd-7cb23aee818b.png"> 

Car는 Engine을 외부에서 받아옴으로써 Engine에서 자유롭습니다. 즉 Car의 Engine값에 다양한 값을 전달할 수 있으며(Electro Engine, Bio Engine등) 내가 원하는 Mock Engine들이 Car에 전달하여 테스트가 가능합니다. 즉 테스트 더블이 가능해졌다라고 할 수 있습니다.

### 테스트 더블(Test double)이란?

테스트 더블은 한국말로 ‘대역 가능 테스트’ 정도로 해석 할 수 있습니다. 좀 더 쉽게 ‘아무거나 대신 다른걸로 넣어볼게 테스트’ 정도로 해석 할 수 있습니다.
Car내부에서 Engine을 생성한 경우 우리는 항상 Engine을 이용해서만 테스트를 해야합니다.(다른 엔진을 받아올 수 없습니다.) 이로인해 Car를 테스트 할 수 있는 Engine의 범위는 Engine단 하나로 제한됩니다.
하지만 우리가 테스트할때는 실제 Engine이 아닌 가상의 값을 넣어 Car를 테스트 하고싶습니다. 이때 Engine을 매개변수로 받는다면 여러 다른 하위 Engine을 테스트 할 수 있으며 Engine을 대신할 대역(대역 배우같은)을 넣어 테스트가 가능해집니다.
Car에 전달되는 대역 인자값을 Stub/Mock/Fake 등으로 표현되며, 이러한 방식을 테스트 더블이라고 합니다.

즉, 가상의 다양한 Engine값을 넣어 Car를 테스트 할 수 있다면 테스트 더블이 가능하다라고 할 수 있겠습니다.

##### *참고: 스턴트 대역배우를 영어로 stunt double이라고 합니다.*

여태까지 우리는 수동으로 의존성 주입을 하는 방법에 대해서 알아봤습니다. 그런데 아직 우리는 뭔가 좀 부족합니다. Car같은경우 여러가지 객체를 매개변수로 받을 수 있습니다. 엔진, 타이어, 보닛, 핸들등등 너무나 많은 외부 객체가 있습니다. 이런경우 수동으로 저런 클래스들을 생성해서 넣어준다면 개발자는 아래와 같은 코드를 생산합니다. 외부 생성 객체가 100개라면 어떻게 될까요???

```kotlin
fun main{
    var engine = Engine()//상용구 코드
    var tier = Tier()//상용구 코드
    var bonnet = Bonnet()//상용구 코드
    var handle = Handle()//상용구 코드
    var car = Car(engine, tier, bonnet, handle)
    car.start()
}

class Car(private var engine : Engine, private var tier : Tier
          , private var bonnet : Bonnet, private var handle : Handle) {
    fun start() {
        engine.start()
    }
}

class Engine {
    fun start() {}
}
```
var engine = Engine()같은 상용구 코드( boilerplate code와 같은말입니다)가 많아지며, 많아진 상용구 코드는 개발자를 피로하게 하며(계속 쳐야되니까) 수정 관리가 용이하지 않게 만듭니다.(찾아가며 바꿔줘야되니까)

##### *반복되고 강조되는 코드는 개발자를 불안하게 해요.(feat. 개통령 아저씨)*

#### 자동으로 DI하는 방법은 없을까?

DI라이브러리는 수동으로 DI하는 것을 자동으로 DI를 가능하게 하며, 이를 제어역전이라고 합니다.(사실 제어역전중의 일부가 DI입니다.) 자동으로 DI를 하는 라이브러리는 Dagger, Dagger2, Koin, Hilt등이 있으며 상기 개념에 대해 이해도가 낮은상태에서 접근시 굉장히 어려움을 겪습니다. 저 또한 그랬습니다. 이후에 알아보도록 하겠습니다.

#### 제어역전(IoC)이란?

항상 개발에서는 용어가 발목을 잡습니다.

Inversion of control이란 뭘까요??
역전이라고 하는걸 이해하기전에 일반적인 제어가 무엇인지 알 필요가 있습니다.
일반적인 제어란, 개발자가 원할때 원하는 코드를 명시적으로 생성, 호출, 생명주기 통제등을 하는 행위를 이야기 합니다. 원할때 원하는 코드를 사용하려면 다량의 제어문(if, when, try-catch)이 들어갈 수 있으며 이또한 오류 가능성을 높이고, 개발자의 피로도를 증가시킵니다.
제어 역전이란 개발자가 클래스의 생성, 호출, 생명주기 통제등을 제어하는게 아니라 위임받은 라이브러리(ex:Dagger, Koin, Spring 등)가 코드의 생성, 호출, 생명주기 통제등의 행위를 의미합니다.
즉, **개발자가 할일을 위임하면 라이브러리가 대신 해준다.** 정도로 이해하시면 될것같습니다.

##### *안드로이드 개발자는 나도모르게 IoC에 익숙해져 있습니다.startActivity(Intent(Activity))를 호출하면 언제 어디서 Activity의 onCreate가 호출되는지 모르지만 우리는 사용할 수 있습니다.*

## Koin
koin이라는 라이브러리가 생소할 수 있습니다. Koin은 kotlin이라는 안드로이드 공식 언어 개발자들이 개발한 DI 프레임워크이며 순수 코틀린으로만 제작되었습니다. 개인적으로 다른 프레임워크(Dagger같은..)와의 가장 큰 차이점은 경량화가 되어있고, 러닝커브다 낮다라는 장점이 있습니다.

#### Koin은 DSL이다.

제가 생각하는 가장 중요한 요소인것 같습니다. 시작부터 어려운 단어가 튀어 나옵니다. 그것은 바로 DSL…하지만 여러분 모두 알고있는 개념이니 안심하셔도 됩니다.

#### DSL(Domain Specific Language)

Koin은 도메인 특화 언어로써 SQL을 생각하면 이해가 쉬울지 모르겠습니다. SQL에서 특정 데이터를 갱신시키려면 update문을 써야만 하며, 특정 데이터를 조회하기 위해서는 select문을 써야 합나다. 또한 삭제는 delete를 써야하죠. select문으로는 delete를 할 수 없습니다. 이처럼 하나의 도메인당 역할이 주어져 있으며, 그에 맞는 도메인을 사용하면 된다라는 장점이 있습니다. *module*을 만들고 싶으면 modules(…)를 사용하면 됩니다. 마치 SQL create 처럼 말이죠.

Koin DSL키워드를 요약하면 아래와 같습니다.(물론 더 많습니다.)

+ *startKoin{}* - 어플리케이션을 생성하고 괄호안에 있는 도메인들을 등록하고 Koin GlobalContext안에 등록합니다.
+ *koinApplication{}* - 어플리케이션을 생성하고 괄호안에 있는 도메인들을 등록합니다.
+ *modules{}* - 사용될 모듈을 정의합니다.
+ *single{}* - 싱글톤 패턴으로 인스턴스를 생성합니다.(get Instance)
+ *factory{}* - 새로운 인스턴스를 생성합니다.(new Class)
+ *get{}* - 구성요소들 간의 종속성을 자동으로 매핑시켜줍니다.
+ *viewmodel{}* - MVVM의 viewmodel전용으로 이용됩니다.
+ *androidLogger()* - 안드로이드 전용 로그를 작성합니다.
+ *logger()* - 로그를 작성합니다.
+ *properties()* - 주어진 맵을통하여 properties를 선언합니다.

중복된게 있다라고 느껴지는 이유는 android만을 위한 라이브리가 아니기 때문입니다. Android 전용이 있고, 그 외에 용이 있다는 말이죠. 우리는 안드로이드 개발자이기때문에 Android 전용만 골라서 쓰도록 하겠습니다. 왠지 전용이라고 하니까 특별해진 기분입니다.

##### *제발 글로 설명하지말고 코드를 보여주세요. 저는 한글보다 코드읽는게 더 쉬워요.-개발자 이모씨*

### Gradle

gradle 세팅을 알아보겠습니다. 먼저 module수준의 gradle file에 koin-android를 추가해 주겠습니다. **버전은 업데이트가 진행중이므로 사용시점에 최신버전을 검색하여 적용하는것이 좋겠습니다.**
koin은 공식문서에는 나와있지는 않지만, 제공되는 샘플코드 검토결과 최소 sdk버전을 21이상으로 설정해야 하는것으로 보입니다. 21버전이면 범용성은 상당히 괜찮은편 입니다.
```kotlin
defaultConfig {
        minSdkVersion 21
        ...
    }
...
dependencies {
...
    implementation "org.koin:koin-android:2.1.1"
...
}
```

### Package 구성하기

“Android개발자가 본 DI 란??”에서 설명드렸었던 Car와 Engine의 DI를 자동으로 하는 방법으로 코드를 구현해보도록 하겠으며 패키지 구조는 아래와 같습니다.(기본 개념 이해를 돕기위하여 최대한 간단하게 구성해보았습니다.)
<img src = "https://user-images.githubusercontent.com/48902047/142581977-38e157c9-dff0-44b2-bdce-1fea57fdae1f.png">

### Car Engine 클래스 생성하기

Car는 Engine을 매개변수로 받습니다. 이때 의존성이 발생하는데 의존성 주입을 이때 이용할 수 있습니다. 수동으로 하던것을 어떻게 자동으로 하는지 알아보겠습니다.

```kotlin
class Car(var engine: Engine) {
    var TAG = javaClass.simpleName

    fun startCar() {
        Log.d(TAG, "start Car")
        engine.start()
    }
}

class Engine {
    var TAG = javaClass.simpleName

    fun start() {
        Log.d(TAG, "Start Engine")
    }
}
```
### Application에서 의존성 주입하기

Application에서 의존성을 주입하는 방법이 Koin라이브러리의 핵심이라고 할 수 있겠습니다. 코드는 아래와 같으며 설명은 코드를 보신 다음에 하도록 하겠습니다.
```kotlin
class MainApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        // start Koin context
        startKoin {
            androidContext(this@MainApplication)
            androidLogger()
            modules(appModule)
        }
    }
}

val appModule = module {
    single { Engine() }
    factory { Car(get()) }

}
```
Application()은 앱 전역에서 선언하는것은 다 선언합니다. 예를 들면 SharePreference가 있습니다.

val appModule = module{} 로 사용할 모듈들 정의합니다. single로 Engine을 생성하고, factory로 Car를 생성합니다. 이때 특이한점은 Car의 인자값으로 get()를 이용한다는 것 입니다.

get()는 상기 Car Class에서 정의한 매개변수(Engine)를 찾아서 자동으로 매핑시켜 줍니다. 아주 편하죠? 이때 인자값이 다르거나, 매개변수 값이 다르면 컴파일 오류가 나기때문에 런타임 오류는 걱정하지 않으셔도 됩니다.

##### *single은 싱글톤으로 객체를 생성하며, factory는 새객체를 생성합니다. 나눈 이유는 여러가지 도메인이 있다는것을 보여주기 위함이며 필요에 의해서 사용하면 되겠습니다. ex> repository는 single로 생성, viewmodel은 factory로 생성*

싱글톤은 대부분 무거운 친구이면 선언을 합니다.

이렇게 만들어진 appModule은 Application에서 startKoin{}을 통해서 주입시켜 줍니다. startKoin은 설명안해도 느낌 오시지 않나요? Koin을 사용하겠다 이런 뜻이며 사용시 필요한 내용들을 넣어주면 됩니다.(context, log, modules 등등)

### MainActivity에서 사용하기
사전준비는 모두 마쳤으니 MainActivity에서 실제로 한번 사용해보도록 하겠습니다.
```kotlin
class MainActivity : AppCompatActivity() {
    val car: Car by inject()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        car.startCar()
    }
}
```
여러가지 상용구 코드(boilerplate)가 사라졌습니다. “Android개발자가 본 DI 란??”에서 인용하면 아래와 같은 코드죠.
<img src = "https://user-images.githubusercontent.com/48902047/142582938-d5f981e8-1f81-4fbf-bb05-41d083d35ba4.png">

by inject()를 통하여 제어역전이 되었으며,(우리는 val car 인스턴스가 언제 어떻게 생기는지 모르지만 사용할 수 있습니다.) 자동으로 의존성을 주입하는 코드가 완성되었습니다.

## Koin Library 사용하여 MVVM 사용하기

### Gradle

gradle세팅에서 중요한 내용은 minSdkVersion입니다. 공식 문서에는 나와있지 않지만, 제공되는 샘플로 미루어보아 21버전 이상을 설정해주시는게 좋을 것 같습니다.

최신 koin-android 를 추가하고, koin으로 의존성 주입을 할 수 있는 viewmodel 도메인을 사용하기 위해 koin-viewmodel을 추가해 줍니다.

또한 MVVM에 꼭 필요한 lifecycle라이브러리를 추가해 줍니다.

databinding이용을 위해 databinding도 enable 시켜줍니다.

Viewmodel, LiveData, Databinding 같은경우 별도의 포스팅에서 설명하고 있습니다. 같이 봐보시면 좋을 것 같습니다.
```kotlin
defaultConfig {
    minSdkVersion 21
    ...
}
dataBinding {
    enabled = true
}
...
dependencies {
    ...
    implementation "org.koin:koin-android:2.2.2"
    implementation "org.koin:koin-android-viewmodel:2.2.2"
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
    ...
}
```
### Package 구성하기
패키지 구성은 아래와 같습니다. 실제 프로그램에서는 CarRepository에서 데이터베이스(Room) 또는 서버통신(Retrofit)을 통해서 데이터를 가져와야하지만, 내용이 복잡할것 같아 model패키지 안에 재료들을 만들어 놓았습니다.

#### *Room에 대한 사용법에 대해서는 링크 남겨놓도록 하겠습니다.*
<img src = "https://user-images.githubusercontent.com/48902047/142718957-91e6196a-01b8-41bd-ba29-d3a5ebc4d7f2.png">
권장 아키택쳐를 그림으로 표현하면 아래와 같습니다.(해당 패키지는 사실상 Repository까지만 구현하였습니다.)
<img src = "https://user-images.githubusercontent.com/48902047/142720358-880ba381-27ff-46d0-a476-7300ec3d9aa8.png">

### Car 클래스 구성요소 추가하기
저번 포스팅에서는 Engine만을 매개변수로 받았지만 Car에 대한 다른 여러 매개변수도 추가하였습니다.(보닛, 핸들, 타이어)

여러 매개변수를 받을때 상용구 코드가 많이 생성될 수 있지만, 이를 Koin을 이용하여 제거할 수 있습니다.

각 클래스는 가독성을 높이기 위해서 별다른 내용을 넣지 않았습니다.
```kotlin
class Car(var engine: Engine, var bonnet: Bonnet, var handle: Handle, var tier: Tier) {
    var TAG = javaClass.simpleName

    fun startCar(): String {
        engine.startEngine()
        bonnet.doingSomething()
        handle.doingSomething()
        tier.doingSomething()

        Log.d(TAG, "started Car")
        return "started Car"
    }
}

class Engine {
    var TAG = javaClass.simpleName

    fun startEngine() {
        Log.d(TAG, "Start Engine")
    }
}

class Bonnet {
    fun doingSomething() {
        //doingSomething
    }
}

class Handle {
    fun doingSomething() {
        //doingSomething
    }
}

class Tier {
    fun doingSomething() {
        //doingSomething
    }
}
```
### 테스트 더블이 가능한 Repository만들기
**여기서 특이한점은 CarRepository를 상속받아 구현하는 구현채인 CarRepositoryImpl을 만들었다는 점입니다.**

이는 테스트 더블을 위하여 구현하였습니다. 전기차에 대한 Repository를 만들고 싶다면 CarRepository를 상속받아 구현하는 구현채인 ElectronicCarRepositoryImpl을 사용하면 되며, 여러 다른 테스트를 할때도 StubCarRepositoryImpl등으로 구현하여 테스트가 가능해집니다.
```kotlin
interface CarRepository {
    fun getCar(): Car
}

class CarRepositoryImpl(car: Car) : CarRepository {
    var TAG = javaClass.simpleName
    val mCar: Car = car

    override fun getCar(): Car {
        //get Car class from Room Database or Retrofit
        //but we don't have any Room Database and Retrofit
        //then I received Car class from Model class

        return mCar
    }
}
```
### Viewmodel 만들기
Viewmodel은 별다른 기능없이 Car를 시작하여(시동을 거는 느낌으로) 그 결과를 화면에 ObservableField를 통하여 세팅하는 클래스로 구성하였습니다.
```kotlin
class CarViewModel(val mainRepo: CarRepository) : ViewModel() {
    var carResult = ObservableField<String>()
    var car = mainRepo.getCar()

    fun startCar() {
        var result = car.startCar()
        carResult.set(result)
    }
}
```
### Application에서 의존성 주입 준비하기(가장 중요)
이부분은 설명을 조금 길게 해볼까 합니다. 준비 되셨나요? 심호흡 한번 하고 전체 코드를 보신 후 같이 상세 코드로 넘어가보도록 하겠습니다.
```kotlin
class MainApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // start Koin context
        startKoin {
            androidContext(this@MainApplication)
            androidLogger()
            modules(localDataModule, repoModule, viewModelModule)
        }
    }
}

val localDataModule = module {
    factory { Engine(); Bonnet(); Handle(); Tier() }
    factory { Car(get(), get(), get(), get()) }
}

val repoModule = module {
    single<CarRepository> {
        CarRepositoryImpl(get())
    }
}

val viewModelModule = module {
    viewModel {
        CarViewModel(get())
    }
}
```
먼저 아래의 코드를 보면 localDataModule을 선언하고 그 안에 facytory를 2개 선언하였습니다.

첫번째 factory에는 Engine(), Bonnet(), Handler(), Tier()어떤것도 인자값으로 받지 않고, 재료가 되는 클래스들을 생성하였습니다.

두번째 factory에는 Car(get()…)을 통하여 Car에 필요한 요소들을 넣어주었으며 이때 get()은 Car에 필요한 인자값들이 선언(factory, single등의 방법으로)이 되어있는지 자동으로 탐색하게 됩니다.

**아래의 코드만 설정해 놓으면 사용시 언제 어떻게 어디서 인자값을 전달받았는지 알 수 없지만(*제어 역전*) 우리는 Car에서 Engine, Bonnet등의 값을 사용할 수 있습니다.**

<img src = "https://user-images.githubusercontent.com/48902047/142719121-e35a7d62-ff1a-484d-aa0c-6a5c76eb9deb.png">
<img src = "https://user-images.githubusercontent.com/48902047/142719136-857b7254-aa13-4369-8114-2746ef47b3c9.png">

그 다음 repoModule은 repository전용으로 구성하였고 repository는 여러 군대에서 지속적으로 사용하므로 single(SingleTone)으로 구성하였습니다.

CarRepositoryImpl(get())을 통하여 CarRepositoryImpl에 필요한 인자값(Car)을 자동으로 탐색하여 매핑시켜줍니다.

**아래의 코드만 설정해 놓으면 우리는 CarRepositoryImpl에서 Car을 사용할 때 언제 어떻게 어디서 초기화되고 인자를 전달받았는지 모르지만(*제어 역전*) Car을 사용할 수 있습니다.**

<img src = "https://user-images.githubusercontent.com/48902047/142719160-c1809fbe-8141-4ebf-8ae9-5581ed1cd6bd.png">
<img src = "https://user-images.githubusercontent.com/48902047/142719167-34c0d54b-bd8d-4cc4-a348-c533c1a2ca17.png">
그 다음 viewModelModule은 viewmodel전용 module로 구성하였고 내부에 viewmodel전용 도메인인 viewmodel{}을 선언하였습니다.

CarViewModel은 CarViewModel(get())을통해 필요한 인자값(CarRepositoryImpl)을 자동으로 받을 수 있습니다. CarViewModel은 테스트더블이 가능하도록 하기 위하여 CarRepository의 구현체를 인자로 받습니다.

제어역전이 되었다라는걸 다시한번 작성하면 눈이 아플수도 있으니 생략하겠습니다. :)

<img src = "https://user-images.githubusercontent.com/48902047/142719186-47fc807f-c0cf-43e8-803b-4c64eb2d2228.png">
<img src = "https://user-images.githubusercontent.com/48902047/142719188-5e158e0a-4c98-4d3f-8113-50f615a5e0aa.png">

MainActivity에서 어떻게 활용되는지 살펴보고 마무리하겠습니다.

### MainActivity에서 의존성 주입하기

by inject()를 통해 CarViewModel에 의존성 주입하였으며 연쇄작용으로 Repository, Model수준의 의존성도 Application에서 설정한 대로 자동으로 주입되었습니다.

우리는 ClassName(get())을 통하여 어떻게 의존성 주입을 해줬으면 좋겠어 라는 설정만 했을 뿐이지 실제로 선언을 한다거나 하지는 않았습니다.

하지만 carVM.startCar()를 통하여 Car의 기능을 모두 사용할 수 있습니다.

또한 상용구 코드들이 제거되어 MainActivity가 매우 깔끔해보입니다.
```kotlin
class MainActivity : AppCompatActivity() {
    val carVM: CarViewModel by inject()
    lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        carVM.startCar()

        binding.viewmodel = carVM
    }
}
```
이로써 안드로이드 MVVM에서 강조하는 관심사 분리가 되어있고, 테스트가 용이한 간단한 MVVM아키택쳐가 완성되었습니다.
