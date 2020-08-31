## 자바 Concurrent

> Concurrent 소프트웨어
- 동시에여러작업을할수있는소프트웨어
- 예)웹브라우저로유튜브를보면서키보드로문서에타이핑을할수있다.
- 예)녹화를하면서인텔리J로코딩을하고워드에적어둔문서를보거나수정할수있다.
> >  지원하는 컨커런트 프로그래밍 
- 멀티프로세싱 (ProcessBuilder)
- 멀티쓰레드
> 자바 멀티쓰레드 프로그래밍
- Thread / Runnable
```java
//스레드 상속을 통한 스레드 구현
public static void main(String[] args) {
    HelloThread helloThread = new HelloThread();
    helloThread.start();
    System.out.println("hello : " + Thread.currentThread().getName());
}
static class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println("world : " + Thread.currentThread().getName());
    } 
}

///Runnable 구현 또는 람다
Thread thread = new Thread(() -> System.out.println("world : " + Thread.currentThread().getName())); thread.start();
System.out.println("hello : " + Thread.currentThread().getName());
```
> 쓰레드 주요 기능
- 현재 쓰레드 멈춰두기 (sleep): 다른 쓰레드가 처리할 수 있도록 기회를 주지만 그렇다고
락을 놔주진 않는다. (잘못하면 데드락 걸릴 수 있겠죠.)
- 다른 쓰레드 깨우기 (interupt): 다른 쓰레드를 깨워서 interruptedExeption을 발생 시킨다.
그 에러가 발생했을 때 할 일은 코딩하기 나름. 종료 시킬 수도 있고 계속 하던 일 할 수도
있고.
- 다른 쓰레드 기다리기 (join): 다른 쓰레드가 끝날 때까지 기다린다.

## Executors
 > 고수준 (High-Level) Concurrency 프로그래밍
- 쓰레드를 만들고 관리하는 작업을 애플리케이션에서 분리.
- 그런 기능을 Executors에게 위임.
 > Executors가 하는 일
- 쓰레드 만들기: 애플리케이션이 사용할 쓰레드 풀을 만들어 관리한다.
- 쓰레드 관리: 쓰레드 생명 주기를 관리한다.
- 작업처리및실행:쓰레드로실행할작업을제공할수있는API를제공한다.
 > 주요 인터페이스
- Executor: execute(Runnable)
- ExecutorService: Executor 상속 받은 인터페이스로, Callable도 실행할 수 있으며,Executor를 종료 시키거나, 여러 Callable을 동시에 실행하는 등의 기능을 제공한다.
- ScheduledExecutorService: ExecutorService를 상속 받은 인터페이스로 특정 시간 이후에 또는 주기적으로 작업을 실행할 수 있다.

```java
///ExecutorService로 작업 실행하기
ExecutorService executorService = Executors.newSingleThreadExecutor(); executorService.submit(() -> {
System.out.println("Hello :" + Thread.currentThread().getName()); });

///ExecutorService로 멈추기
executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
executorService.shutdownNow(); // 당장 종료
```
- Fork/Join 프레임워크
> ExecutorService의 구현체로 손쉽게 멀티 프로세서를 활용할 수 있게끔 도와준다.

## Callable과 Future
> Callable
- Runnable과 유사하지만 작업의 결과를 받을 수 있다.
Future
- 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html
```java
///결과를 가져오기 get()
ExecutorService executorService = Executors.newSingleThreadExecutor(); Future<String> helloFuture = executorService.submit(() -> {
Thread.sleep(2000L);
return "Callable"; });
System.out.println("Hello");
String result = helloFuture.get();
System.out.println(result); executorService.shutdown();
```
- 블록킹 콜이다.
- 타임아웃(최대한으로 기다릴 시간)을 설정할 수 있다.
> 작업 상태 확인하기 isDone()
- 완료 했으면 true 아니면 false를 리턴한다.
> 작업 취소하기 cancel()
- 취소 했으면 true 못했으면 false를 리턴한다.
- parameter로 true를 전달하면 현재 진행중인 쓰레드를 interrupt하고 그러지 않으면 현재
> 진행중인 작업이 끝날때까지 기다린다. 여러 작업 동시에 실행하기 invokeAll()
- 동시에실행한작업중에제일오래걸리는작업만큼시간이걸린다.
> 여러 작업 중에 하나라도 먼저 응답이 오면 끝내기 invokeAny()
- 동시에실행한작업중에제일짧게걸리는작업만큼시간이걸린다.
- 블록킹 콜이다.

## CompletableFuture
> 자바에서 비동기(Asynchronous) 프로그래밍을 가능케하는 인터페이스.
- Future를 사용해서도 어느정도 가능했지만 하기 힘들 일들이 많았다.
> Future로는 하기 어렵던 작업들
- Future를 외부에서 완료 시킬 수 없다. 취소하거나, get()에 타임아웃을 설정할 수는 있다.
- 블로킹 코드(get())를 사용하지 않고서는 작업이 끝났을 때 콜백을 실행할 수 없다.
- 여러 Future를 조합할 수 없다, 예) Event 정보 가져온 다음 Event에 참석하는 회원 목록
> 가져오기
- 예외 처리용 API를 제공하지 않는다.
> CompletableFuture
- Implements Future
- Implements ​CompletionStage
> 비동기로 작업 실행하기
- 리턴값이 없는 경우: runAsync()
- 리턴값이 있는 경우: supplyAsync()
- 원하는 Executor(쓰레드풀)를 사용해서 실행할 수도 있다. (기본은ForkJoinPoolcommonPoo())
> 콜백 제공하기
- thenApply(Function): 리턴값을 받아서 다른 값으로 바꾸는 콜백
- thenAccept(Consumer): 리턴값을 또 다른 작업을 처리하는 콜백 (리턴없이)
- thenRun(Runnable): 리턴값 받지 다른 작업을 처리하는 콜백
- 콜백자체를또다른쓰레드에서실행할수있다.
> 조합하기
- thenCompose(): 두 작업이 서로 이어서 실행하도록 조합
- thenCombine(): 두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행
- allOf(): 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행
- anyOf():여러작업중에가장빨리끝난하나의결과에콜백실행
> 예외처리
- exeptionally(Function) - handle(BiFunction):