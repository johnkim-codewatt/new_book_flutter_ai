# Riverpod
Riverpod은 비교적 최신의 상태관리 패키지 이다. 간단한 사용법과 달리 강력한 상태관리 방법을 제공하기 때문에 인기가 높아지고 있는 추세이다. Provider 이외에 최신의 상태관리 패키지를 이용하고 싶다면 Riverpod은  최고의 솔루션이 될수 있다. Riverpod은 처음부터 Provider의 한계를 극복하고자 설계된 새로운 상태 관리 라이브러리이며, Provider의 철학을 계승하면서도, 더 나은 유연성과 안정성을 제공하기 위한 구조로 재설계되었다.

결국 여러가지 상태관리 패키지를 사용하는 목적은, 어떤 패키지를 선택하든 본질적으로는 동일할것이다.(상태관리를 잘하기 위함이다.) 잠시 그 내용을 들여다 보자. 첫번째, Flutter 앱에서 가장 핵심적인 과제 중 하나인 상태 관리를 보다 원활하게 처리하는 것이다. 우리는 Provider를 사용하면서 간편하게 상태관리 하는 방법을 배웠다. 둘째, UI 코드와 비즈니스 로직(상태 변화에 따른 처리 로직)을 명확히 분리하는 구조를 만드는 것이다. 이는 단순히 코드 정리를 위한 개념이 아니라, 실제 프로젝트 규모가 커지고 유지보수가 중요 해질수록 기능별 책임을 나누고 관리하는 구조를 만들기 위함이다. 굳이 '클린코드', '클린아키텍처' 와 같은 골치아픈 이론을 따르지 않더라도, UI와 상태로직을 분리하는 것 자체만으로도 큰 의미가 있다. 이제 Riverpod을 통해서 상태관리와 비지니스 로직 처리를 어떻게 해야할지 자세히 살펴보자.

>**[팁&노트]**
클린아키텍처는, 로버트 마틴(Uncle Bob)이 제안한 소프트웨어 설계 원칙으로, UI와 비즈니스 로직, 데이터 계층을 명확히 분리하여 독립적으로 테스트하고 확장 가능하게 만드는 구조를 말한다. 이는 코드의 가독성과 유지보수성이 크게 향상시켜준다.[^2] 



**패키지추가**
https://pub.dev/packages/riverpod
https://pub.dev/packages/flutter_riverpod

아래의 내용은 flutter_riverpod을 사용한다.

```dart
dependencies:
  flutter_riverpod: ^2.6.1
```

우리에게 친숙한 카운터 예제를 통해 내용을 살펴보자. Riverpod은 기본적으로 Provider의 사용과 비슷한점이 많다.

**전체소스**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 상태를 전역으로 관리하기 위해 글로벌 변수로 선언하고
// StateProvider 간단한 단일 상태정보를 처리하기에 좋다.
final counterProvider = StateProvider<int>((ref) => 0);

// 앱 진입점에서 ProviderScope 설정
void main() {
  runApp(
	  ProviderScope( //Scope를 설정한다.
		  child: MyApp()
	  )
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: CounterPage(),
    );
  }
}

// 상태를 사용하는 위젯
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  // 제공되는 WidgetRef ref 를 통해 상태정보를 읽어올수 있다.
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    //전역으로 선언된 counterProvider 를 구독한다.
    final count = ref.watch(counterProvider); // 상태 읽기

    return Scaffold(
      appBar: AppBar(title: const Text('Riverpod 카운터')),
      body: Center(
        child: Text(
          '$count', //상태정보를 사용한다.
          style: const TextStyle(fontSize: 40),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () { //버튼을 눌렀을때 상태 값증가 코드
          ref.read(counterProvider.notifier).state++; // 직접 상태에 접근해서 값 변경
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}

```

기존의 Provider의 사용에 익숙하다면, 특별히 어려운 코드는 없을것이다. 우선 riverpod에서 provider를 통해 상태값을 제공하려면 ProviderScope를 작성해야한다. ProviderScope는 쉽게 말해 상태값을 공유할 영역을 지정하는 코드이다.

**ProviderScope**
대부분의 경우 앱 진입점에서 ProviderScope를 설정하면된다. 앱 전역에서 어디서든 상태정보에 접근하기 위함이다. Provider의 위젯트리에 기반한 상태관리 구조에서 벗어나, 지정된 영역안에서 온전히 상태관리와 사용에만 집중할수 있다.

```dart
void main() {
  runApp(
    ProviderScope( // 앱 전체를 감싸는 형태 = 전역사용설정
      child: MyApp(),
    ),
  );
}
```

**StateProvider**

위에서 잠시 살펴본, StateProvider는 간단한 값(int, String, bool 등)을 상태정보로써 관리할 때 사용하기 적합한 Provider이다.
Flutter 위젯 트리의 상하 구조와 무관하게 전역으로 상태를 정의할 수 있기때문에 매우 편리하다.



```dart
final counterProvider = StateProvider<int>((ref) => 0); //계층구조와 무관하게 글로벌 변수로 상태를 등록
```

>등록된 상태는 어디로 가는걸까요?, Riverpod은 내부적으로 해당 상태를 Provider Pool(상태 저장소와 같은 개념)에 등록하여 관리합니다. 필요할때마다 꺼내쓰면 되는것이지요. 


**read & watch**
상태공유를 위한 영역설정, 등록을 마쳤으니 이제 사용할 차례이다. Riverpod은 값을 읽어오기 위해 두가지 방법을 제공한다. read 와 watch이다. 눈치채겠지만, 이둘의 의미는 provider의 그것과 동일하다.


<table>
  <thead>
    <tr>
      <th><strong>항목</strong></th>
      <th><code>ref.watch()</code></th>
      <th><code>ref.read()</code></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>역할</strong></td>
      <td>상태를 <strong>“구독”</strong> (감시)</td>
      <td>상태를 <strong>“읽기만”</strong> 함</td>
    </tr>
    <tr>
      <td><strong>반응성</strong></td>
      <td>상태가 바뀌면 위젯이 <strong>자동 리빌드됨</strong></td>
      <td>상태가 바뀌어도 위젯은 <strong>리빌드 안됨</strong></td>
    </tr>
    <tr>
      <td><strong>사용 위치</strong></td>
      <td><code>build()</code> 등 <strong>UI 렌더링에서 사용</strong></td>
      <td><strong>onPressed</strong>, <strong>initState</strong> 등 <strong>비UI 이벤트에서 사용</strong></td>
    </tr>
    <tr>
      <td><strong>예시</strong></td>
      <td><code>final count = ref.watch(counterProvider)</code></td>
      <td><code>ref.read(counterProvider.notifier).state++</code></td>
    </tr>
  </tbody>
</table>

ref.watch()는 구독(감시)의 의미이다. 해당 상태를 구독하고, 구독은 상태가 변경되면 해당 값을 사용중인 위젯이 갱신 처리된다. (그렇기 때문에 build 안에 코드를 작성한다.)

```dart
 @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider); // 상태 구독
		//...
```

반면 ref.read()는 단순히 값을 읽을때 사용하면된다.

```dart
ref.read(counterProvider.notifier).state //ref.read를 통해 state에 접근한다.
```

---

**반드시 구분해서 써야 하는 watch와 read**

ref.watch()와 ref.read()의 차이를 제대로 이해하지 못하고 상황에 맞지 않게 사용하면, 불필요한 위젯 리빌드가 발생할 수 있다. 처음엔 아무런 문제 없이 작동하는 것처럼 보여도, 앱 규모가 커질수록 성능에 영향을 줄 수 있기 때문에 주의해야한다.

예를 들어 아래의 코드를 살펴보자.

```dart
ElevatedButton(
  onPressed: () {
    // 상태 변화를 감시할 필요가 없음에도 불구하고 watch를 사용함
    final counter = ref.watch(counterProvider);
    print('현재 값: $counter');
  },
  child: Text('증가'),
)
```

겉보기에는 아무 문제가 없어 보이며, 동작도 문제없다. 하지만 이 버튼은 상태 변화에 따라서 화면을 다시 그릴 필요가 전혀 없어 보인다. 그럼에도 watch를 사용했기 때문에, 상태가 변경될 때마다 자동적으로 이 버튼을 포함한 위젯 전체가 불필요하게 갱신 된다. 이런 문제가 비동기처리를 위한 함수 내부에서도 발생할 수 있다.

```dart
void someAsyncFunction(WidgetRef ref) async {
  final value = ref.watch(myFutureProvider); // 비동기 로직 내부에 watch 사용
  // ...
}
```

비동기 함수 로직 내부에는 굳이 상태를 구독할 필요가 없기 때문에, read로 충분히 잘 동작한다. 해당 코드는 의도치 않은 갱신을 유발하고, 이는 앱의 성능 저하로 이어진다. 반드시 read, watch를 상황에 맞게 써야하는 이유이다.

---

## **ConsumerWidget(StatelessWidget)**

```dart
// 상태를 사용하는 위젯
class CounterPage extends ConsumerWidget { //ConsumerWidget를 상속한다.
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    //...
  }
//...

}

```

위 코드를 자세히 들여다보면, 낯선 구문이 눈에 띄었을 것이다. 바로 ConsumerWidget이다. 반드시 상속해야 하는 필수 요소는 아니지만, 대부분의 경우에서 편리성과 명확한 상태관리를 위해 자주 활용된다.

build 메서드를 자세히 보면, WidgetRef ref라는 매개변수가 존재한다. 이 ref가 바로  ConsumerWidget을 상속할 때 build를 통해 자동으로 제공되는 매개변수이다. 이를 통해 Provider의 상태를 구독하거나 읽는 작업을 손쉽게 수행할 수 있는 것이다. 

```dart
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

>**[팁&노트]** 
Riverpod은 위젯트리구조에 의존하지 않기 때문에, ref만 있다면 어디서든 안전하게 상태를 읽거나 조작할 수 있다. 이것은 기존의 BuildContext에 의존하는 Provider보다 더 유연한 상태관리 방법이다. [^1]

---

## **ConsumerStatefulWidget**
그렇다면, 우리가 배운 StatelessWidget과, StatefulWidget은 어디로 간것일까? 사실 위에서 배운 ConsumerWidget은 StatelessWidget를 기반하는 위젯이다. 이는 자체적으로 상태 갱신 처리를 할수 없다는 뜻이다. 하지만 실무에서는 복잡한 상태관리와 화면갱신처리가 필요한 경우가 있다. 

아래의 코드를 살펴보자. 전역으로 관리하는 상태정보인 counterProvider와, 하위 위젯 자체에서만 사용할 isLoading 이라는 상태정보를 복합적으로 관리하는 해야하는 상황이다.

```dart
/// 전역 상태: 정수형 카운터를 저장하는 Provider
final counterProvider = StateProvider<int>((ref) => 0);
```

```dart
class MyPage extends ConsumerStatefulWidget {// ConsumerStatefulWidget를 상속
  const MyPage({Key? key}) : super(key: key);

  @override
  ConsumerState<MyPage> createState() => _MyPageState();
}

class _MyPageState extends ConsumerState<MyPage> {
  bool isLoading = false; // 내부에서만 사용하는 자신만의 상태정보

  @override
  Widget build(BuildContext context) {
    final count = ref.watch(counterProvider); // 전역상태 구독

    return Column(
      children: [
        if (isLoading) CircularProgressIndicator(), //내부에서만 사용되는 상태 isLoading 사용
        Text('카운터: $count'),
        ElevatedButton(
          onPressed: () async {
            setState(() => isLoading = true); 
            await Future.delayed(Duration(seconds: 1));
            ref.read(counterProvider.notifier).state++;
            setState(() => isLoading = false); //setState를 통해 스스로 화면을 갱신
          },
          child: Text('증가'),
        ),
      ],
    );
  }
}
```

위 코드처럼 복합적인 상태관리가 필요하다면, ConsumerStatefulWidget 를 상속하면된다. isLoading은 해당 위젯 안에서만 사용하는 로컬 상태이며, 버튼을 눌렀을 때만 상태가 변경되므로, 전역으로 관리할 필요가 없다. 따라서 StatefulWidget을 사용하듯이, setState()로 처리하면 된다.
전역상태는 ref.watch()를 통해 counterProvider를 구독처리하고, 상태 변경 시 build()가 자동으로 다시 호출되도록 작성하면 된다.


### AI와 함께 학습하세요
>Q. dart에서 riverpod 패키지를 사용중인데, 왜 ConsumerWidget(StatelessWidget)은 build의 매개변수로 ref를 사용하고, ConsumerStatefulWidget(StatefulWidget) 에서는 build가 아닌 전역 ref를 사용하지? 일관성이 없어보이는데?


이렇게 복합적인 구조를 사용하면 상황에 맞는 유연한 대처가 가능해진다. 전역 상태(Provider)와 로컬 상태(setState)를 필요에 따라 조합함으로써, 각 상태의 사용 범위와 로직을 분리할수 있고, 과도한 전역화나 불필요한 복잡성을 피할 수 있다.

---



## **Consumer**
ConsumerWidget을 사용하지 않고도 Riverpod을 활용할 수 있는 방법이 있다. 바로 Consumer 위젯을 사용하면된다. 사용방법은 Provider와 유사하고, 특정 부분만 상태 변화에 반응하도록 감싸는 용도로 사용된다. 즉, 위젯 트리 중 리빌드가 필요한 부분만 따로 감싸서 최적화할 수 있는 유연한 도구인 것이다.

```dart
class HomePage extends StatelessWidget { //StatelessWidget 를 상속한다.
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('고정 텍스트'), // 이 부분은 리빌드되지 않음
        Consumer(
          builder: (context, ref, child) { //Consumer의 build를 통해 ref에 접근할수있다.
            final name = ref.watch(userProvider);
            return Text('사용자: $name');
          },
        )
      ],
    );
  }
}
```

위 예제처럼 Consumer를 사용하면 ref를 부분적으로 사용할 수 있기 때문에, 불필요한 리빌드를 방지하고 필요한 부분만 반응형으로 처리하는 매우 합리적인 코드 구조가 된다. 이러한 방식은 성능 최적화는 물론이고, 코드 가독성과 유지보수 측면에서도 유리하다. 따라서 위젯 전체를 ConsumerWidget으로 감싸기 부담스러울 때, 또는 상태 변경이 일어나는 일부 요소만 갱신시키고 싶을 때 Consumer는 좋은 선택이 될 수 있을것이다.

---

## **StateNotifierProvider**
StateNotifierProvider는 단순히 값을 전달하는 것이 아닌, 상태를 ‘관리’ 하는 로직에 초점을 맞출 때 사용하는 Provider 이다. 기존의 StateProvider는 int, String, bool 등 간단한 값을 저장하고 전달하는데 적합하지만, 상태 변경에 대한 로직이 필요한 경우 한계가 있다. 이럴 때 StateNotifierProvider를 사용하면 더 유연하고 명확한 상태 관리를 할수 있다.

```dart
class CounterNotifier extends StateNotifier<int> { //StateNotifier를 상속한다.
  CounterNotifier() : super(0); // super를 통해 초기값(0)을 전달한다.

  void increment() => state++; //값 증가를 담당하는 기능
}
```

상태를 관리하는 로직은 StateNotifier<T> 클래스를 상속받아 구현하며, <T>(제네릭)에는 관리할 상태의 타입을 지정한다. (int, List, Map, 혹은 사용자 정의 클래스 등) 위 클래스는 상태를 변경하는 로직을 내부에 포함한 일종의 상태 컨트롤러 역할을 수행한다. state는 우리가 정의한 변수는 아니지만, 내부적으로 가지고 있는 현재 상태값을 의미하며, 만약 이를 변경하면 자동으로 UI가 갱신된다.

이제 StateNotifierProvider 를 등록할 차례이다.

```dart
final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(), //상태정보 컨트롤러를 지정한다.
);
```
 
 아래의 전체 코드를 살펴보자.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 상태 컨트롤러
class CounterNotifier extends StateNotifier<int>{ //StateNotifier를 상속한다.
  CounterNotifier() : super(0); // 초기값 0

  void increment() => state++; //증가시키거나
  void decrement() => state--; //감소시키거나
}

// Provider 정의
final counterProvider =
    StateNotifierProvider<CounterNotifier, int>((ref) => CounterNotifier());

void main() {
  runApp(ProviderScope(child: MyApp()));//ProviderScope 정의
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: CounterPage());
  }
}

class CounterPage extends ConsumerWidget {//ConsumerWidget 상속
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider); // 상태 구독

    return Scaffold(
      appBar: AppBar(title: Text('StateNotifier 카운터')),
      body: Center(child: Text('$count', style: TextStyle(fontSize: 40))),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            onPressed: () => ref.read(counterProvider.notifier).increment(), // 상태 변경 로직
            child: Icon(Icons.add),
          ),
          SizedBox(height: 10),
          FloatingActionButton(
            onPressed: () =>
                ref.read(counterProvider.notifier).decrement(), // 상태 변경 로직
            child: Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

CounterNotifier는 상태를 변경하는 로직을 내부에 포함한 일종의 상태 컨트롤러이다. 이 컨트롤러는 state 값을 직접 관리하며, 외부 위젯에서는 ref를 통해 해당 상태를 읽거나 변경할 수 있다. 이렇게 함으로써 UI작성에 관련된 코드와, 상태, 로직에 관련된 코드가 좀 더 명확하게 구분된것을 확인할수 있다. (우리가 하는 대부분의 액션은, 유지보수와 가독성을 위함이다.)



### AI와 함께 학습하세요
>Q. 반응형 프로그래밍이란 무엇을 의미하는가?


---

## Copilot을 통해 실습해보세요

Riverpod에서 **상태를 직접 제어하는 구조**를 배우고,
Copilot의 제안을 활용해 increment / decrement 기능을 완성해보세요.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 상태를 다루는 Notifier 클래스 정의
// 1. StateNotifier<int>를 상속하세요.
class CounterNotifier extends //... {
  CounterNotifier() : super(0);

  // 2. 값을 1 증가시키는 메서드를 작성해보세요.
  void increment() {
    //...
  }

  // 3. 값을 1 감소시키는 메서드를 작성해보세요.
  void decrement() {
    // ...
  }
}
```

```dart
// Provider 등록하기
final counterProvider =
    // CounterNotifier와 int를 연결하는 StateNotifierProvider를 작성해보세요.
    StateNotifierProvider<//...>((ref){//...});
```

```dart
// 앱의 시작점: ProviderScope로 감싸기
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

```dart
// 앱 UI 작성
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: CounterPage());
  }
}
```

```dart
// 상태를 사용하는 화면: ConsumerWidget 활용
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 현재 count 상태를 구독해보세요.
    final count = //...

    return Scaffold(
      appBar: AppBar(title: Text('📦 Riverpod 실습')),
      body: Center(
        child: Text(
          '현재 값: $count', //count를 표시합니다.
          style: TextStyle(fontSize: 32),
        ),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          // 증가 버튼
          FloatingActionButton(
            onPressed: () {
              // 버튼을 누르면 값이 증가하는 처리를 수행합니다.
              // ...
            },
            child: Icon(Icons.add),
          ),
          SizedBox(height: 12),
          // 감소 버튼
          FloatingActionButton(
            onPressed: () {
              // 버튼을 누르면 값이 감소하는 처리를 수행합니다.
              // ...
            },
            child: Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

---

[^1] Riverpod : https://riverpod.dev/docs/from_provider/provider_vs_riverpod

[^2] Robert C. Martin(“Uncle Bob”) 저서 Clean Architecture: A Craftsman’s Guide to Software Structure and Design