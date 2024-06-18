# Synchronized vs Volatile vs Atomic

<img src="https://velog.velcdn.com/images/dabeen-jung/post/a98e9aac-64ea-4807-bfd9-060bf98ae252/image.png" alt="Java to RAM struct" width="500" height="auto">

Java에서의 메모리 구조는 CPU, CPU Cache Memory, RAM으로 구성되어 있으며, 멀티 스레드 환경에서는 가시성 문제와 동시 접근 문제를 직면할 수 있습니다.<br> 이를 해결하기 위해 synchronized, volatile, Atomic을 사용할 수 있으며, 각 방법의 특징과 사용법을 예시 코드와 함께 설명하겠습니다.

## 가시성 문제
Java에서는 CPU가 Main Memory(RAM)에서 데이터를 가져온 후, 동일한 데이터에 대해 빠른 접근을 위해 CPU Cache에 데이터를 저장합니다.<br> 각 쓰레드는 레지스터와 캐시를 사용하여 메모리에 접근하므로, 캐시 일관성 문제가 발생할 수 있습니다.<br> 이는 한 쓰레드가 변경한 데이터를 다른 쓰레드가 즉시 보지 못하는 경우를 의미합니다.

```java
public class test {
    private static boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread newThread = new Thread(() -> {
            while (running) { }
            System.out.println("Thread terminated.");
        });

        newThread.start();
        Thread.sleep(1000);

        running = false;
        System.out.println("Main thread set running to false.");
    }
}
```

다음과 같은 코드에서 Main Thread와 new Thread가 실행된후 running = false를 하여 while문이 false가 되어 new Thread가 종료 될거란 예상이 있지만 실상은 그렇지 않습니다.<br>
running field는 각 Thread마다 Cpu가 아는 값을 마지막으로 복사 후 각 쓰레드별로 따로 값을 저장 중 이므로 newThread에서는 running이 Main thread에서 값이 변경되더라도 그것을 인지 할 수 없습니다.

## 동시 접근 문제
멀티 스레드 환경에서는 여러 쓰레드가 동일한 변수에 접근하거나 수정할 때 동시 접근 문제가 발생할 수 있습니다. 이는 가장 늦게 반영된 쓰레드의 값이 최종 값으로 반영되는것을 의미합니다.

```java
    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[10];

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    System.out.println("current thread = " + threads[finalI].getName() + " count = " + counter++);
                }
            });
        }

        for (Thread t : threads) {
            t.start();
        }

        for (Thread t : threads) {
            t.join();
        }
    }
```

다음 코드에서 counter의 최종 출력값이 10000 이 예상 하실 수 있지만 실상은, 같은 값을 출력할 때도 있고, 10000까지 도달하지도 않습니다.<br> 그 이유는 공유자원에 동시에 접근하기 때문에, 연산 속도가 빠른 쓰레드에서의 변경값이, 연산속도가 느린 쓰레드에서의 변경값으로 덮어씌워지는 것 이다.

## 해결책

이러한 동시성 이슈들을 해결하기 위해 Java에서는 3가지 키워드를 제공합니다.

## Synchronized 
동일한 자원에 대한 접근을 오로지 한 Thread만 허용 토록 하는 것 입니다.
```java
//synchronized method
public synchronized void doSomething() {}

//synchronized block
synchronized(T) {}
``` 
method, block에 다음과 같이 synchronized 를 적용하면 오로지 한 thread만 접근이 가능하므로, 모든 이슈가 사라집니다.<br>
하지만 단점으로는 synchronized 같은 경우에 한 thread가 이미 임계영역을 장악하고 있을시 다른 thread는 lock이 걸리게되어 병목현상이 발생할 수 있어, 무분별한 synchronized 사용은 성능을 저하시킵니다.

## volatile

volatile는 생소할 수 있는 keyword입니다. volatile는 data를 오로지 Main Memory 에만 저장 하겠다는 키워드입니다.<br>
다음은 volatile가 어떻게 가시성 이슈를 해결 할 수 있는지에 대한 flow입니다.

- 메인 메모리 접근
    - 변수가 volatile로 선언되어있다면. 따라서 변수의 값을 읽거나 쓸 때 항상 메인 메모리에서 직접 수행됩니다. 메인 스레드가 변수값을 변경하면, 작업 스레드는 즉시 그 변경된 값을 볼 수 있습니다.

- 메모리 배리어
    - 변수의 쓰기 연산과 읽기 연산에는 메모리 배리어가 삽입되어, 변수의 최신 상태가 일관되게 모든 스레드에 보이게 됩니다. 이는 변수의 값을 변경하는 순서가 보장됨을 의미합니다.

하지만 volatile만으로는 동시접근이슈까지는 해결을 할 수 없습니다.

## atomic
atomic이란 synchronized의 성능 이슈를 해결하기 위한 방법입니다.

CAS연산을 사용하여 가시성, 동시 접근 이슈 둘 다 해결합니다.

### CAS FLOW

1. 초기 값 읽기: 처음에 메모리 위치(V)에서 현재 값을 읽습니다. 이 값은 캐시에 저장될 수도 있습니다.
2. 비교: 현재 값이 기대 값(A)과 같은지 비교합니다.
3. 교체 시도: 현재 값이 기대 값(A)과 같다면, 메모리 위치(V)에 새로운 값(B)을 원자적으로 저장합니다.
4. 교체 실패 시 재시도: 만약 현재 값이 기대 값(A)과 같지 않다면, 최신 값을 다시 읽어와 비교합니다. 이 과정은 일반적으로 반복됩니다.

### 의사 코드
````java
public boolean compareAndSwap(int expectedValue, int newValue) {
    while (true) {
        int currentValue = getValueFromMemory(); // 메모리에서 현재 값을 읽어옴
        if (currentValue == expectedValue) {
            if (compareAndSet(currentValue, newValue)) {
                return true;
            }
        } else {
            // 값이 다르다면 현재 값을 갱신하고 다시 시도
            expectedValue = currentValue;
        }
    }
}
```

CAS는 Lock을 사용하지 않으므로 Synchronized 키워드에 비해 성능이 좀 더 빠릅니다.  
특정 스레드가 block에 걸리지 않으므로 데드락 현상도 없습니다.

다만 CAS도 ABA라는 추가적인 문제점을 제공합니다.
ABA를 해결하기위해서는 Double Check나 Tag라는 해결책이 있지만 후술 하도록 하겠습니다.