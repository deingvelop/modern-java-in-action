
# Box-and-channel Model (박스와 채널 모델)

- 동시성 모델을 설계하고 개념화한 그림

![[Pasted image 20230713181816.png]]
- 방법 1
  ```java
  int t = p(x);
  System.out.println( r(q1(t), q2(t)) );
  ```

- 방법 2 : Future를 이용해 f, g를 병렬로 평가함
  ```java
  int t = p(x);
  Future<Integer> a1 = executorService.submit(() -> q1(t));
  Future<Integer> a2 = executorService.submit(() -> q2(t));
  System.out.println( r(a1,get(), a2.get()));
  ```
  - p는 다른 어떤 작업보다 먼저 처리해야 하며 r은 모든 작업이 끝난 다음 가장 마지막으로 처리해야 한다.
  ```java
  System.out.println( r(q1(t)z q2(t)) + s(x) );
  ```
  - 위 코드에서 병렬성을 극대화하려면 모든 다섯 함수(p, q1, q2, r, s)를 Future로 감싸야 함


## 발생하는 문제
- 시스템이 커지고 각각의 많은 박스와 채널 다이어그램 등장 => 만약 각 박스가 내부적으로 자신만의 박스와 채널 사용  => 많은 task가 `get()` 메서드를 호출해 Future가 끝나기를 기다리는 상태일 수 있음
- 결과적으로 하드웨어의 병렬성을 제대로 활용하지 못할 수 있음
- 심지어 데드락에 걸릴 수 있음
- 대규모 시스템 구조 -> 얼마나 많은 수의 `get()`을 감당할 수 있는지 이해하기 어려움

<br>

# Java 8에서의 해결
:  **CompletableFuture와 combinator를 이용하여 문제 해결**
- combinator 이용
  - 변경 전 : compose()와 andThen() 이용
    ```java
    Function<Integer, Integer> myfun = add1.andThen(dble);
    ```
  - 변경 후 : combinator 이용
    ```java
    p.thenBoth(q1,q2).thenCombine(r);
    ```


## Tips

- 박스와 채널 모델을 이용하여 생각과 코드를 구조화할 수 있음
- 즉, 대규모 시스템 구현의 추상화 수준을 높일 수 있음
- 박스(혹은 프로그램의 콤비네이터)로 원하는 연산을 표현(계산은 나중에 이루어짐)하면, 계산을 손으로 코딩한 결과보다 효율적임
- 콤비네이터 : 수학적 함수 뿐만 아니라 Future와 Reactive Stream Data에도 적용 가능함
- 박스와 채널 모델 : 병렬성을 직접 프로그래밍하는 관점을 콤비네이터를 이용해 내부적으로 작업을 처리하는 관점으로 바꿔줌
   ( ≒ Java 8 stream : 자료구조를 반복해야 하는 코드를 내부적으로 작업을 처리하는 스트림 콤비네이터로 바꿔줌)