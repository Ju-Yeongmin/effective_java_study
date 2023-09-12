# Item 80
## 스레드보다는 실행자, 테스크, 스트림을 애용하라

### java.util.concurrent
이 패키지는 실행자 프레임워크 라고하는 인터페이스 기반의 유연한 테스크 실행 기능을 담고 있다
- 과거에는 단순한 작업 큐를 만들기 위해서 수많은 코드를 작성해야 했다
- 이제는 아래와 같이 간단하게 작업 큐를 생성할 수 있다
  ```java
  // 큐를 생성
  ExecutorService exec = Executors.newSingleThreadExecutor();
  // 실행자에 실행할 태스크(task; 작업) 을 넘기는 방법
  exec.execute(runnable);
  // 실행자 종료
  exec.shutdown();
  ```
#### 이러한 실행자 서비스에는 다음과 같은 주요 기능들이 있다
- 특정 테스크가 완료되기를 기다린다
- 테스크 모음 중 하나(invokeAny) 또는 모든 테스크(invokeAll)가 완료되기를 기다린다
- 실행자 서비스가 종료하기를 기다린다(awaitTermination)
- 완료된 테스크의 결과를 차례로 받는다(ExcutorCompletionService 이용)
- 테스크를 특정 시간 혹은 주기적으로 실행하게 한다(ScheduledThread PoolExcutor 이용)

#### 그 외에도 아래의 다양한 기능들을 지원한다
- 큐를 둘 이상의 스레드가 처리하도록 스레드풀 생성
- ThreadPoolExecutor 클래스를 속성을 설정하여 커스텀할 수 있다
- 가벼운 프로그램이라면 Excutors.newCachedThreadPool을 사용하면 특별한 설정없이 사용할 수 있다
- 무거운 프로그램이면 Executors.newFixedThreadPool나 ThreadPoolExecutor를 사용할 수 있다

### 실행자 서비스를 주의해서 사용해야 하는 이유
- 동일한 자원을 공유하는 여러 스레드가 번갈아 동작할 때 어떤 task가 먼저 실행될지 알 수 없다
- Thread와 Runnable을 무작정 생성하여 멀티 스레드 프로그래밍을 하면 동시성 문제를 겪게된다
- 따라서 큐를 손수 만드는 일은 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로는 삼가야 한다

### ForkJoinTask
java 7 부터는 실행자 프레임워크는 포크-조인 테스크를 지원한다
- ForkJoinTask의 인스턴스는 작은 하위 테스크로 나뉠 수 있다
- ForkJoinPool을 구성하는 스레드들이 테스크들을 처리한다
- 일을 먼저 끝낸 스레드는 다른 스레드의 테스크를 가져와 대신 처리할 수 있다
- 모든 스레드가 바쁘게 움직여 CPU를 최대한 활용하기 때문에 높은 처리량을 보여준다

```java
핵심!
큐를 직접 만들거나 스레드를 직접다루는 것은 지양하고 실행자 프레임워크를 사용하자
```
