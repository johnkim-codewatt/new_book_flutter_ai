# Provider 패키지 소개
Provider 는 아마도 Flutter 생태계에서 가장 오래되고 널리 사용되는 상태 관리 패키지 중 하나일 것이다. 구글이 2019년에 공식적으로 소개한 이후, 지금까지도 많은 개발자들의 사랑을 받고 있으며, 상태 관리의 대표적인 솔루션으로 자리잡았다.

Provider는 상태관리를 위한 패키지이다. Provider 내부를 깊이 들여다보면 우리가 배웠던 InheritedWidget를 만날수 있다. 즉, Provider는 InheritedWidget의 한계를 보완하고 사용성을 개선한 래퍼(wrapper) 역할을 하며, 더 심플하고 강력한 상태 관리 방식을 제공하고 있다. 우리는 앞서 다양한 상태 관리 아이디어를 살펴보았고, 그 과정에서 InheritedWidget을 직접 사용했을 때 상태 관리가 얼마나 간편하게 사용할 수 있는지 경험해봤다. 하지만 동시에 InheritedWidget를 사용하면 코드가 장황해지고, 상태관리가 오히려 더 복잡해질수도 있다는걸 경험했다.

Provider는 이러한 문제점들을 개선, 보완하는 기능을 제공한다. 코드의 가독성을 높이고, 재사용성과 유지보수성도 향상시킬 수 있으며, 직관적으로 상태관리를 구현할 수 있는 구조를 제공한다. 

## **패키지추가**
[Provider](https://pub.dev/packages/provider)
앞서 배웠던 패키지 추가 방식을 떠올리고, 공식 패키지 배포 공간인 pubdev를 통해 패키지를 추가한다. pubspec.yaml 파일을 아래와 같이 작성해보자. (공식 문서를 참고한다.)

```dart
// pubspec.yaml
// provider 패키지를 추가해 주세요.
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.0
```

## **Provider 기본사용법**
만약 Int, String 같은 기본 자료형을 상태관리를 통해 공유하고자 한다면, Provider기본형을 작성하면 된다. 아래와 같이 간단하게 작성하며 해당 값을 읽는것도 쉽게 가능하다. 

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    Provider<String>( //단순한 String 공유를 위한 Provider 생성
      create: (_) => '안녕하세요!', //공급할 데이터 (문자)
      child: MyApp(), //마찬가지로 하위 위젯을 작성한다.
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //사용방법은 비슷하다. of를 통해 상위에 등록된 Provider의 값을 읽어온다.
    final greeting = Provider.of<String>(context); // 또는 context.watch<String>()

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Provider 기본 예제')),
        body: Center(
          child: Text(
            greeting,
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```

하나씩 살펴보도록하자. Provider를 필요한 위젯 상단에 배치하고, create에 공유할 상태값을 작성합니다. (InheritedWidget에서 처럼 상위위젯에서 상태관리를 하고, 해당 정보를 공유한다는 아이디어 자체는 계속 유효하다.)

```dart
 Provider<String>(
    create: (_) => '안녕하세요.', // 공급할 데이터
    child: MyApp(),
  ),
```

이 값을 읽을때는 아래와 같이 작성하면 된다. Provider.of 는 InheritedWidget에 작성했던 of 함수와 동일한 역할을 수행한다. Provider.of를 통해 Provider참조를 가져오고 상단에 등록된 Provider의 String 값을 꺼낼수 있다.

```dart
final greeting = Provider.of<String>(context); // 또는 context.watch<String>()
```

하지만 이 코드는 고정된 특정 값을 공유할때 아주 유용하지만, 시간에 흐름에 따라 변하는 동적인 상태값을 공유하기에는 알맞지 않다. 예를 들어 count 라는 상태 변수가 있고, 이 값이 지속적으로 변한다고 가정한다. 만약 상태가 1에서 2, 2에서 3으로 바뀔때마다 이값을 하위 위젯에서 모니터링 해야 한다면 어떻게 구현해야 할까?

## **ChangeNotifierProvider**
동적인 (변하는) 상태값을 감지하고, 그에 따른 로직을 유연하게 구현하고 싶다면 ChangeNotifierProvider를 사용하는 것이 가장 적합하다. 이 Provider는 상위에 있는 상태값이 변경되었을 때, 이를 사용하는 하위 위젯들에게 자동으로 변경 사실을 알려주어 화면을 갱신할 수 있도록 도와준다. 실제로 Flutter에서 가장 많이 활용되는 Provider 구문 중 하나이기도 하다.

그럼 먼저 count 라는 상태를 관리하기 위한 '모델 클래스' 를 하나 만들어보자. 특별한 규칙은 없으며, ChangeNotifier를 상속받기만 하면 된다.

>**[팁&노트]**
모델의 개념은 앱에서 사용할 데이터를 담는 일종의 ‘컨테이너’ 이다. 유저 정보, 카운트 숫자, 설정값 등 앱에서 보여주거나 처리해야 할 다양한 데이터를 담기위해 존재하고, 기능구현 보다는 데이터를 중심으로 class를 통해 작성한다.


```dart
// 상태정보를 보관하기 위한 모델 클래스
class CounterModel extends ChangeNotifier {
  int _count = 0; //상태변수
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // 이 함수를 호출하면 관련된 UI가 자동으로 리빌드 됨
  }
}
```

클래스는 특정 데이터들을 중심으로 작성되며, 내부에는 count라는 상태 변수와 이를 조작할 수 있는 메서드가 들어간다. 자세히 보면 notifyListeners()하는 특수한 메서드가 보인다. 이 메서드는 상태가 변경되었음을 Provider에 등록된 리스너들에게 알리는 역할을 하며, 이를 통해 해당 상태를 사용 중인 하위 위젯들이 자동으로 갱신되도록 만들어준다. (클래스를 작성할때 반드시 모델이라는 단어를 쓸필요는 없다.)

![image.png](/images/03_11_상태관리패키지_provider.png)

즉, notifyListeners()는 “이 값이 바뀌었으니, 관련된 화면을 다시 그려주세요!“ 라고 신호를 보내는 것과 유사하다. 이러한 구조 덕분에 상태값과 UI가 자연스럽게 연결되고, 보다 직관적으로 상태 기반 UI를 설계할 수 있게된다.

이어서 상단에 ChangeNotifierProvider 를 배치하고, 만들어둔 CounterModel 을 작성한다.

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CounterModel(),
      child: MyApp(),
    ),
  );
}
```

이제 하위 위젯에서는 CounterModel이라는 상태관리객체 참조를 가져다 쓰기만 하면 된다. 

### **상태읽기**
사실 상단에 등록된 Provider의 상태값을 가져다 쓰는 방법은 크게 3가지가 있다.

- *Provider.of<T>(context)*
- *context.watch<T>()*
- *context.read<T>()*

**provider.of**
먼저 기본이 되는 Provider.of 이다. 이 함수는 InheritedWidget 을 작성할때 썼던 of의 의미와 동일한 개념으로 이해하면된다. 

```dart
Provider.of<CounterModel>(context, listen: true)
```

매개변수에 있는 listen(true, false)은 상위위젯에 작성된 상태 변화에 따라 provider를 구독하는 하위 위젯 자신의 갱신여부를 결정하는 플래그이다. 여기서 listen은 암시적으로 true를 기본값으로 가져가기 때문에 생략이 가능하다. 그래서 보통은 아래와 같이 사용하면 된다. (*true면 상위상태값 변화에 따라 자동으로 하위 위젯 화면이 갱신된다.)

```dart
Provider.of<CounterModel>(context)
```

만약 listen 을 false로 작성하면, 이는 다음에 배울 context.read 와 유사하게 동작한다. (값을 읽기만함) 쉽게 말해 하위 위젯이 값만 사용 하고 “나는 갱신할 필요없어~” 와 같은 효과를 낼수있는 것이다.

```dart
Provider.of<CounterModel>(context, listen: false)
```

![image.png](/images/03_11_상태관리패키지_provider2.png)

**context.watch<T>**

context.watch<T>는 watch 라는 직관적인 명칭을 보면 알수있듯이, <T>타입의 상태변화를 감지, 감시(또는 구독)하는 구문이다. 이 확장함수를 사용하면 CounterModel의 상태변화를 감지하고 이를 사용하는 하위 위젯들은 자동으로 상태가 업데이트 된다. (setState를 호출하지 않아도 상태변화에 따라서 알아서 화면이 갱신되는 마법같은 일이 일어난다.)

```dart
context.watch<CounterModel>()
```

**context.read<T>**
위에서 설명했듯이 context.read<T> 는 provider.of 의 false 옵션을 준것과 유사하게 동작한다. 위젯에서 단순히 값을 읽기만 할때 사용하면 편리하다.

```dart
context.read<CounterModel>()
```

실제 코드작성을 통해 위에서 배운 3가지 방법을 활용해보자.

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 상태를 보관하고 변경을 알리는 모델 클래스
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); //구독자들에게 상태가 변경되었음을 알림
  }
}

void main() {
  runApp( //최상단에 ChangeNotifierProvider를 배치
    ChangeNotifierProvider( //ChangeNotifierProvider를 사용해 상태를 관리
      create: (_) => CounterModel(), // 상태 공급
      child: MyApp(),
    ),
  );
}

// 실제 UI를 구성하는 위젯
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 상태를 "구독" 한다.
    final count = context.watch<CounterModel>().count; 
    //Provider.of 를 통해 provider 참조 자체를 가져와서 활용한다.
    final provider = Provider.of<CounterModel>(context);
    
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('ChangeNotifierProvider 예제')),
        body: Center(
          child: Column(
            children: [
              Text(
                '구독 형태의 사용: $count', //상태정보 count를 표시한다. 
                style: TextStyle(fontSize: 32),
              ),
              
              Text(
                '기본 형태의 사용: ${provider.count}', //provider 참조를 통해 상태정보를 표시한다.
                style: TextStyle(fontSize: 32),
              ),
              
            ],
          )
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            //context.read<CounterModel>()를 통해 
            //상태변경 메서드를 호출한다.
            context.read<CounterModel>().increment(); 
            //provider.increment(); 
            //provider of 를 사용해도 같은 동작을 처리한다.
          },
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}

```

---

### **MultiProvider**

앱이 커지고 복잡해짐에 따라, 여러개의 Provider를 만들게 되는 경우가 있다. 이럴때는 아래와 같은 복잡한 중첩구조를 만들어야 할지도 모른다. 이 코드는 사용하는데 별 문제가 없지만, 특유의 중첩구조 때문에 가독성이 떨어지고, 익히 알고있듯이 이런 코드는 분석을 어렵게 만든다.

```dart
return ChangeNotifierProvider<A>(
  create: (_) => A(),
  child: ChangeNotifierProvider<B>(
    create: (_) => B(),
    child: ChangeNotifierProvider<C>(
      create: (_) => C(),
      child: MyApp(),
    ),
  ),
);
```

위 코드를 개선하면 아래와 같이 작성할수 있다. 같은 기능을 하는 코드이며, 훨씬더 깔끔하고 구조가 평탄화 되어있어서 코드를 분석하기도 좋다.

```dart
return MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => A()),
    ChangeNotifierProvider(create: (_) => B()),
    ChangeNotifierProvider(create: (_) => C()),
  ],
  child: MyApp(),
);
```

상위 어딘가에 Provider를 등록하고 하위 위젯에서 사용하는 아이디어는 언제나 동일하게 적용된다.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 첫 번째 상태: 사용자 정보
//ChangeNotifier 를 상속
class UserModel extends ChangeNotifier { 
  String name = "홍길동";

  void changeName(String newName) { //이름을 변경하고, 구독자에게 상태 변경을 알림
    name = newName;
    notifyListeners();
  }
}

// 두 번째 상태: 앱 설정 정보
//일종의 테마 관리를 위한 모델 클래스
class SettingsModel extends ChangeNotifier {
  bool darkMode = false;

  void toggleMode() {
    darkMode = !darkMode;
    notifyListeners();
  }
}

void main() {
  runApp(
    MultiProvider( //MultiProvider를 사용해 여러 상태를 관리
      providers: [
        ChangeNotifierProvider(create: (_) => UserModel()),
        ChangeNotifierProvider(create: (_) => SettingsModel()),
      ],
      child: MyApp(),
    ),
  );
}

// 실제 UI
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //watch를 사용해 상태를 구독하고 필요한 값을 읽어온다.
    final name = context.watch<UserModel>().name; 
    final dark = context.watch<SettingsModel>().darkMode; 

    return MaterialApp(
      //3항 연산자를 사용해 다크모드 여부에 따라 테마를 설정
      theme: dark ? ThemeData.dark() : ThemeData.light(),
      home: Scaffold(
        appBar: AppBar(title: Text('MultiProvider 예제')),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 읽어온 name으로 화면에 표시
            Text('사용자 이름: $name', style: TextStyle(fontSize: 20)),
            SwitchListTile(
              title: Text('다크모드'),
              value: dark,
              //스위치 토글에 따라 다크모드 상태를 변경, 함수를 호출
              onChanged: (_) => context.read<SettingsModel>().toggleMode(),
            ),

            ElevatedButton(
              onPressed: () {
                // 버튼 클릭시 name 변경을 위한 changeName 함수 호출
                context.read<UserModel>().changeName("김길동");
              },
              child: Text('이름 바꾸기'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### **ProxyProvider**
ProxyProvider는 특정 상태에 의존하여 다른 상태를 업데이트할때 유용하게 쓸수 있다. 아래의 코드는 생각과 달리 count 값이 업데이트 되지 않는 문제가 있다. Provider는 create 시점에 전달받은 값을 초기값(initial value) 으로 사용하여, 한 번만 객체를 생성하기 때문이다. 따라서 외부의 변수(count)가 시간에 따라 바뀌더라도 Provider 내부의 count 상태는 처음 그대로이다.

```dart
int count; //외부의 상태정보
//...
Provider( //Provider 생성시점에 count를 전달 받는다.
  create: (_) => MyModel(count),
  child: ...
)
```

이럴때 사용하는것이 ProxyProvider 이다. ProxyProvider를 사용하면 외부의 count 상태값 변화에 따라 MyModel 내부 count 값을 업데이트 할수 있다. 아래의 예제를 살펴보자.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 내부 상태를 가진 클래스
class MyModel {
  int count;
  MyModel(this.count);
}

void main() {
  runApp(MyApp());
}

// 외부 상태를 가진 StatefulWidget
class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  int externalCount = 0; //외부 상태값

  void _changeCount() { //이벤트에 의한 외부 상태값 변화
    setState(() {
      externalCount += 10;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MultiProvider( //최상단에 MultiProvider를 배치
      providers: [
        // 단순값을 전달하기 위한 provier 생성 , 외부 상태 전달
        Provider<int>.value(value: externalCount),
        
        // ProxyProvider를 통해 외부 상태(int)를 내부 모델(MyModel)로 전달
        ProxyProvider<int, MyModel>( 
          update: (_, count, __) => MyModel(count),
        ),
      ],
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('ProxyProvider 예제')),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Consumer<MyModel>(
                  builder: (_, model, __) => Text('값: ${model.count}', style: TextStyle(fontSize: 24)),
                ),
                SizedBox(height: 20),
                ElevatedButton(
                  onPressed: _changeCount,
                  child: Text('외부 count +10'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

이제 외부의 count 변수에 의존하는 provider를 업데이트 할수 있게 되었다. 이처럼 특정 상태값에 의존하는 형태의 Provider 를 작성할때 ProxyProvider는 유용하게 활용되어 진다. 코드를 자세히 보니 처음보는 코드가 보이는것 같다. 바로 Consumer 이다.


### Consumer
Flutter의 네이밍은 굉장히 직관적이다. Consumer 는 말그대로 상태를 소비하는 소비자의 역할을 한다. 우선 기존의 상태값을 사용하는 방법은 어땠는지 다시 상기해보자. Provider.of 와 같은 기본형태를 통해서 로직에 활용하거나, Watch 등을 통해 구독 처리한다고 배웠었다. 이런 방법은 편리하고 좋지만 약간의 문제가 있다. 

```dart
final counter = Provider.of<CounterModel>(context);
```

사실 이 방식은 위에서 배운 Provider 접근 방식의 코드이며, listen: true가 기본값이기 때문에 사용 중인 counter 상태 정보에 변화가 생기면, 이 코드를 사용하는 하위 위젯 전체가 재빌드 되는 방식으로 동작한다.

```dart
final counter = context.watch<CounterModel>();
```

이 코드는 좀더 의미상 명확히 구독에 촛점을 두고있는 방식이다. 하지만 상태 값이 변경되면 해당 위젯 전체가 다시 빌드되는 문제는 동일하게 발생한다. 불필요한 위젯 갱신을 줄이는 좀 더 효율적인 방식은 없을까?

너무나 쉽게 'Consumer' 를 이용하면 상태값을 사용 하는 부분과 재빌드가 일어날수 있는 부분을 최소화 할수 있다. 아래와 같이 최소한의 범위를 consumer로 감싸면 끝이다. 

아래의 코드는 기존의 context.watch 를 사용해 build 전체가 다시 호출(화면에 다시 그려짐) 되는 예제 이다.

```dart
@override
Widget build(BuildContext context) { //해당 build가 다시 호출될수 있다.
  final counter = context.watch<CounterModel>();
  
  return Column(
    children: [
      Text('값: ${counter.count}'), // 음 이 위젯만 변경되면 좋겠는데...
      MyComplexWidget(),           // 이 위젯은 사실상 갱신이 필요없다...
    ],
  );
}
```

Consumer를 통해 개선해보자. 갱신이 필요한 'Text('값: ${counter.count}');' 부분만 Consumer로 감싸고, MyComplexWidget는 그대로 둔다.

```dart
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      //Text를 consumner로 감싸면 필요한 최소부분만 리빌드 되게 할수 있다.
	    Consumer<CounterModel>(
        //변화에 따른 counter 상태가 매개변수로 전달된다.
			  builder: (context, counter, child) { 
			    return Text('값: ${counter.count}'); //업데이트된 count 값을 표시한다.
			  },
			),
      MyComplexWidget(), //이제 MyComplexWidget는 리빌드 되지 않는다.
    ],
  );
}
```

훨씬 효율적인 코드가 완성되었다. 이제 CounterModel의 상태 변화에 따라서 Text위젯만 리빌드 되도록 만들어졌다. 이렇게 작성된 코드는 성능을 유지한채, 로직을 유연하게 구성할수 있기 때문에 반드시 이해하도록 하자. Consumer 처럼 부분적인 리빌드 처리를 위한 또 다른 방법이 있다. 바로 Select 이다.

---

### Select
안타깝지만 실무에서는 예제처럼 간단한 한가지 상태값만을 가지고 있지 않다. 실제로는 굉장히 복잡하고 다양한 데이터 정보를 표시해야하는 경우가 많다. 아래와 같은 경우이다.

```dart
// 내부 상태를 가진 클래스
class MyModel {
  int state1; //여러가지 상태값1
  String state2; //상태값 2
  MyModel(this.state1, this.state2);
}
```
상태클래스 내부의 여러가지 상태정보가 있고, 누군가 이값을 구독하고 있을것이다.

```dart
Widget build(BuildContext context) {
  final myModel = context.watch<MyModel>(); // MyModel 전체를 구독
  return Text(myModel.state1); // state1을 표시
}
```

위 코드는 잘 동작하며, 별 문제없이 보이기도 한다. 하지만 자세히 들여다 보면, 해당 위젯은 state2를 사용하지 않음에도 리빌드 되는 운명을 맞이하게 된다. 기본적으로 context.watch<MyModel>() 는 MyModel 전체를 구독하는 형태이다. 즉, state1, state2 모두를 모니터링하고 있기 때문에, state1이나 state2 중 하나라도 변경되면 해당 위젯은 다시 빌드되는 것이다. (굉장히 비효율적이라고 생각할수 있다.)


이럴때 쓰는것이 Select이다. select는 모델전체가 기준이 아닌, 상태값 자체를 기준으로 구독하는 방식이라고 생각하면 된다. 아래의 코드를 살펴보자.

```dart
Widget build(BuildContext context) {
  // select를 사용하여 MyModel의 특정 상태값만을 구독
  final state1 = context.select((MyModel model) => model.state1);
  return Text(state1); // state1만을 표시
}
```

이제 state2가 변하던 말던 상관없이 state1 값만을 바라보고 로직을 구성할수 있게 되었다. 이처럼 consumer나 select를 활용하면 성능에 최적화된 유연한 구조의 앱을 만들수 있다.


### Future Provider
이제 좀더 현실적으로 생각해보자. 우리가 사용하는 대부분의 앱은 서버와 통신하고, 데이터를 받아와서 가공하며, 화면에 표시하는 것이 일반적인 방식일 것이다. 즉 네트워크 환경이 필수적이라는 이야기이다. 'FutureProvider' 는 ‘비동기처리’ 방식인 'Future' 를 이용한 'Provider' 이다. 아래는 서버로 부터 데이터를 가져오는 가정을 통해 FutureProvider를 사용하는 예제이다.

```dart
// 서버로부터 데이터를 가져오는 함수 (비동기 작업)
// 서버에서 사용자 정보를 가져온다고 가정한다.
Future<String> fetchUserData() async { 
  await Future.delayed(Duration(seconds: 2)); // 2초 지연 (서버 응답 대기 시뮬레이션)
  return "홍길동";
}
```

>**[팁&노트]**
서버로 부터 데이터를 가져온다는 가정을 할때 항상 delay 코드를 작성하는 이유는, API 호출이 네트워크 환경에 따라서 물리적인 시간차가 필연적으로 발생하기 때문이다. (* 얼마나 걸릴지는 정확히 예측할수 없다.) 


사실 위 코드는 기존의 Future 함수 구현과 별 다를것이 없다. 해당 함수를 호출하면 await 구문에서 잠시 대기한뒤(2초간), “홍길동”을 return 하는 단순한 구조이다. 이제 provider에 적용해보자.

이번에도 FutureProvider를 상단 배치하고 create 에 fetchUserData 라는 비동기함수를 호출 하도록 한다. 이때 중요한것은 initialData 를 통해 반드시 초기값을 할당해야 한다는 것이다.

```dart
void main() {
  runApp(
    FutureProvider<String>( // 위젯 상단에 배치
      create: (context) => fetchUserData(), // 만들어둔 비동기 함수를 호출
      initialData: "Loading", // 반드시 초기 데이터를 제공해야한다.
      child: MyApp(),
    ),
  );
}
```

특별히 신경쓸것 없이, 비동기함수를 만들고, FutureProvider 를 작성하는것 만으로 준비는 끝이다. 이제 아래의 코드를 통해 비동기 데이터를 어떻게 사용하면 될지 살펴보자.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: UserScreen(),
    );
  }
}

// 사용자 정보를 표시하는 화면
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //FutureProvider에서 제공하는 비동기 데이터(유저정보)를 받아오기
    final userName = Provider.of<String>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text("FutureProvider Example"),
      ),
      body: Center(
        child: Text(
          'User Name: $userName', //사용자 이름을 표시
          style: TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}
```

UserScreen은 이전과 똑같이 provider.of 를 통해 데이터를 가져오고 표시하는 코드를 작성하면 된다. 비동기처리라고 해서 뭔가 다른 코드를 작성하거나, 수정할 필요가 전혀 없다는 뜻이다. 실행해보면 맨처음 초기값으로 전달한 “Loading” 이 화면에 표시되고, 2초 뒤에 “홍길동” 이라는 데이터가 화면에 표시된다. 자동으로 초기값에서 → 서버에서 2초뒤에 데이터를 전달받아, 그 데이터가 자동으로 화면에 표시되도록 만들어졌다. 우리가 알고있는 setState()함수 라던가, ChangeNotifier의 notifyListeners 와 같은 함수 호출을 하지 않았음에도, 자동으로 값이 변경되고, 자동으로 화면에 표시된다.(이전 방식을 쓰던 입장에서는 마법같은 일이 아닐수없다.) 

>**[팁&노트]**
이와 같이 특정 상태나, 이벤트를 구독하고 변화된 값에 따른 구현로직을 자동으로 구현하는 방법을 '리액트스타일' 혹은 '반응형프로그래밍' 방식 이라고 부른다.[^1]


### StreamProvider
앞서 비동기 처리 Stream 을 이해했다면 자연스럽게 배울수 있는 챕터이다. 다시 말하지만 Stream 은 일종의 흐름과 같다. 강물이 흐르고 그 위에 지속적으로(간헐적, 연속적) 데이터가 발생하고, 누군가 그것을 사용(구독)하는 처리 방식이다. 아래의 예제는 1초마다 카운트가 증가하는 단순한 Stream 함수이다.

```dart
// 매 1초마다 카운터 값을 증가시키는 스트림 함수
Stream<int> counterStream() {
  return Stream<int>.periodic(
    Duration(seconds: 1), 
	  (count) => count + 1
  ).take(10); // 10번만 카운터 값을 증가시키고 종료
}

```

위 함수는 특별할것 없는 Stream의 구조로, count 값을 0부터 시작해 1초 간격으로 10까지 출력하고 종료되는 로직이다. 이 흐름을 Provider를 통해 화면에 표시하기 위해서 우리는 StreamProvider를 사용해야한다. (FutureProvider 처럼 initialData를 반드시 등록해야한다.) 그리고 create 속성에는 위에서 정의한 Stream 함수인 counterStream()을 추가한다.

```dart
void main() {
  runApp(
    StreamProvider<int>( //상단에 배치
      create: (context) => counterStream(), // 작성한 Stream 함수
      initialData: 0, // 초기 데이터 제공
      child: MyApp(), //하위 위젯 배치
    ),
  );
}
```

이제 하위 위젯을 살펴보자. MyApp위젯 아래에 CounterScreen은 특별한 수정 없이 기존처럼 상태값을 읽어오기만 하면 된다. 예제를 실행해보면 자동으로 숫자가 증가(1초 간격으로)하는 것을 확인할 수 있다. (정말 마법같은 일이 아닐수없다.)

```dart

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: CounterScreen(),
    );
  }
}

class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
  
    // StreamProvider에서 제공한 Stream 데이터에 접근
    final counter = Provider.of<int>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text("StreamProvider Example"),
      ),
      body: Center(
        child: Text(
          'Counter Value: $counter', //1초 마다 데이터가 흘러들어온다!
          style: TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}
```

FutureProvider와 StreamProvider는 실제 앱에서 서버 통신, 실시간 데이터 표시 등 다양한 상황에서 매우 자주 사용되는 중요한 비동기 처리 방식이다. Flutter에서 상태 관리를 효율적으로 구현하고자 한다면, 이 두 Provider는 반드시 익혀두어야 한다.



### AI와 함께 학습하세요
>Q. dart의 provider 패키지에 대해서 설명하고, 다양한 활용 예제를 제공해줘.

---

## **Copilot을 통해 실습해보세요!**
 **Copilot과 함께 기본 상태 공유 구조 만들기**

pubdev를 통해, provider를 검색하고, 아래와 같이 패키지를 추가하세요. 

[provider](https://pub.dev/packages/provider)

```dart
// pubspec.yaml에 다음을 추가하세요:
 dependencies:
   provider: ^6.1.0
   //...
```

**단일값 실습**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    // Provider를 사용해 단일 문자열을 공유해보세요.
    Provider<String>(
      // create를 사용해 공급할 문자열을 지정하세요.
      create: (_) => //...
      // child에 MyApp을 연결하세요.
	     child: //...
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Provider를 통해 공유된 값을 읽어보세요.
    final greeting = //Provider.of...

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Provider 실습')),
        body: Center(
          child: Text(
						"..." //greeting을 사용해보세요.
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```

**비동기처리 실습 : FutureProvider**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 비동기 Future 함수를 작성하세요.
// 서버에서 데이터를 가져온다는 상황을 가정해, 2초간 딜레이하는 코드를 작성합니다.
Future<String> fetchData(){
  //...
}

void main() {
  runApp(
    FutureProvider<String>(
      // 비동기로 데이터를 제공해보세요.
      create: (_) => //비동기함수를 호출하세요.
      initialData: '로딩 중...',
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // FutureProvider에서 제공한 값을 읽어보세요.
    final greeting = //Provider.of...

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('FutureProvider 실습')),
        body: Center(
          child: Text(
            '...' // 여기서 비동기값을 사용
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```

**비동기처리 실습 : StreamProvider**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

//1초마다 숫자가 발생되는, Stream 함수를 작성합니다.
Stream<int> getStreamNumber(){
  //...
}

void main() {
  runApp(
    StreamProvider<int>(
      create: (_) => //작성된 Stream 함수를 호출합니다.
      initialData: 0,
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // StreamProvider에서 제공한 값을 읽어보세요.
    final count = //Provider.of...

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('StreamProvider 실습')),
        body: Center(
          child: Text(
            '현재 숫자: ...', // Stream 으로 가져오는 데이터를 표시합니다.
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
    );
  }
}
```


---

[^1] Wikipedia - Reactive programming : https://en.wikipedia.org/wiki/Reactive_programming