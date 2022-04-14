

# GC, GC알고리즘

#목차
- ## 1. GC 란 무엇인가!?
- ## 2. GC 동작 원리
- ## 3. GC 알고리즘


---
## 1. GC 란 무엇인가!?

- GC란 프로그래밍에서 쓰이는 개념으로 가비지 컬렉터(Garbage Collector) 단어 그대로 '쓰레기 수집' 이며 수집한 쓰레기를 모아서 폐기하는걸 뜻한다.
- 그럼 '쓰레기'는 어떤것을 뜻하는것인가? 라고 하면 '정리되지 않은 메모리', '유효하지 않은 메모리 주소' 라고 칭할 수 있다.

쉽게 예를 들면 아래와 같이 예를 들 수 있다.
```
    String[] array = new String[2]; // (1)

    array[0] = '0';
    array[1] = '1';

    array = new String[] {'G', 'C' }; // (2)
```
- (1) 에서 객체를 선언하여 주소를 받은 후 데이터를 넣고 해당 변수에 (2)에서 새롭게 new 를 선언 할 시 (1)에서 선언된 주소는 어떻게 됐을까?
- 정리하면 (1)처럼 찾을 수 없는 주소를 의미하며 이것을 '정리되지 않는 메모리라고 칭한다.'

### GC를 정의하면 메모리 관리를 위해 힙 영역에서 더이상 사용하지 않는 객체를 메모리에서 제거하는 것이 된다.

---
## 2. GC 관리 영역 및 동작 원리

> 그러면 GC는 어떻게 동작하며 어떻게 **'정리되지 않은 메모리'** 를 찾으며 어떻게 메모리 반환하는 이벤트를 하고 있는 것일까? 라는 의문을 갖게 된다.

####동작 원리를 알기 전에 먼저 JVM Heap 영역 안에 Young, Old 의 영역을 이야기를 해야한다.
- (초기에는 Perm 영역이 존재하였지만 Java8부터 제거되었다.)

### a. GC 관리 영역

- Young 영역(Young Generation)
    - 새롭게 생성된 객체가 할당(Allocation)되는 영역
    - 대부분의 객체가 금방 Unreachable 상태가 되기 때문에, 많은 객체가 Young 영역에 생성되었다가 사라진다.
    - Young 영역에 대한 가비지 컬렉션(Garbage Collection)을 Minor GC라고 부른다.


- Old 영역(Old Generation)
    - Young영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역
    - Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 가비지는 적게 발생한다.
    - Old 영역에 대한 가비지 컬렉션(Garbage Collection)을 Major GC 또는 Full GC라고 부른다.


- Old 영역이 Young 영역보다 크게 할당되는 이유는 Young 영역의 수명이 짧은 객체들은 큰 공간을 필요로 하지 않으며 큰 객체들은 Young 영역이 아니라 바로 Old 영역에 할당되기 때문이다.
```
(Young 영역) -> Old(영역) 
```
- 예외적인 상황으로 Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우도 존재할 것이다. 이러한 경우를 대비하여 Old 영역에는 512 bytes의 덩어리(Chunk)로 되어 있는 카드 테이블(Card Table)이 존재한다.
```
(Young 영역) <- Old(영역) - 카드테이블 참조
```
- 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때 마다 그에 대한 정보가 표시된다.


- 카드 테이블이 도입된 이유는 간단한다. Young 영역에서 가비지 컬렉션(Minor GC)가 실행될 때 모든 Old 영역에 존재하는 객체를 검사하여 참조되지 않는 Young 영역의 객체를 식별하는 것이 비효율적이기 때문이다. 그렇기 때문에 Young 영역에서 가비지 컬렉션이 진행될 때 카드 테이블만 조회하여 GC의 대상인지 식별할 수 있도록 하고 있다.


### a. GC 동작 방식

Young 영역과 Old 영역은 서로 다른 메모리 구조로 되어 있기 때문에, 세부적인 동작 방식은 다르다. 하지만 기본적으로 가비지 컬렉션이 실행된다고 하면 다음의 2가지 공통적인 단계를 따르게 된다.

- Stop The World
- Mark and Sweep

1. Stop The World
    - Stop The World는 가비지 컬렉션을 실행하기 위해 JVM이 애플리케이션의 실행을 멈추는 작업이다.
    - GC가 실행될 때는 GC를 실행하는 쓰레드를 제외한 모든 쓰레드들의 작업이 중단되고, GC가 완료되면 작업이 재개된다.
    - 당연히 모든 쓰레드들의 작업이 중단되면 애플리케이션이 멈추기 때문에, GC의 성능 개선을 위해 튜닝을 한다고 하면 보통 stop-the-world의 시간을 줄이는 작업을 하는 것이다.


2. Mark and Sweep
    - Mark: 사용되는 메모리와 사용되지 않는 메모리를 식별하는 작업이다.
    - Sweep: Mark 단계에서 사용되지 않음으로 식별된 메모리를 해제하는 작업이다.
    - compaction: 살아남은 각 객체들이 연속된 형태로 존재하도록 힙의 가장 앞부분부터 채워서 객체가 존재하는 부분과 존재하지 않는 부분으로 나눔

   > Stop The World를 통해 모든 작업을 중단시키면, GC는 스택의 모든 변수 또는 Reachable 객체를 스캔하면서 각각이 어떤 객체를 참고하고 있는지를 탐색하게 된다. 그리고 사용되고 있는 메모리를 식별하는데, 이러한 과정을 Mark라고 한다. 이후에 Mark가 되지 않은 객체들을 메모리에서 제거하는데, 이러한 과정을 Sweep라고 한다.

####기본적으로 두가지의 방식으로 이뤄지며 각 버전마다 알고리즘에 따라서 세세한 동작 방식이 달라진다.

## 3. GC 알고리즘

### a. Serial GC
```
단어 뜻 그대로 순차적인 GC라는 의미로, Mark-Sweep-Compaction 알고리즘이 한 번에 하나씩만 동작한다.
순차적으로 동작할 수밖에 없는 이유는 GC를 처리하는 스레드가 하나이기 때문이다. 

메모리나 CPU 코어 리소스가 부족할 때 사용할 수 있을 것이다. 가장 오래된 GC이며 Stop-The-World 시간이 너무 길기에 요즘에는 사용되지 않는다.
```

### b. Parallel GC
```
Serial GC을 사용하던 시절보다 CPU 코어와 스레드의 개수가 증가하고 성능이 향상된 때에는 Serial GC를 멀티스레드로 실행하는 게 성능에 더 좋을 것이다.

Parallel GC는 Minor GC를 처리하는 스레드를 여러 개로 늘려 좀 더 빠른 동작이 가능하게 한 방식이다.
처리시간보다는 처리량에 초점을 맞춘 GC이기 때문에 Throughput GC라고도 부른다.
```

### c. Parallel Old GC (with Mark-Summary-Compaction)
```
Parallel GC와 비슷하나 Mark-Sweep-Compaction 알고리즘 대신 Mark-Summary-Compaction 알고리즘을 사용한다.

Major GC 수행 시 멀티 스레드로 동작한다.

Old Generation을 Region이라는 논리적인 단위로 균일하게 나누고, 각 스레드들은 Region 별로 사용되는 객체를 표시 (Mark)
Sweep 작업 중에 사용되는 객체를 식별하는 작업이 추가됨. 

Region 단위로 작업을 수행하며 각 Region마다 사용되는 객체의 밀도(Density)를 평가 후 Dense prefix를 설정 이후 Compaction 대상이 되는 Region의 첫 번째로 사용되는 객체 주소를 찾아 저장하여 GC 수행 범위를 정함 (Summary)
모든 스레드가 각 Region을 할당받아 Compaction을 수행
```

### d. CMS(Concurrent Mark Sweep) GC
```
 GC에서 발전한 방식이며, GC 과정에서 발생하는 Stop the World 처리시간을 최소화하는데 초점을 맞춘 방식이다. Low Latency GC라고도 부른다.

Stop the World 처리시간에 초점을 맞추다 보니 GC 대상을 최대한 자세히 파악할 수밖에 없다. 그 과정이 복잡하기 때문에 다른 GC 대비 CPU 사용량이 높다.

 

Initial Mark → Concurrent Mark → Remark → Concurrent Sweep 과정을 거친다. (아래 참조)

Initial Mark
    GC 과정에서 살아남은 객체를 탐색하는 시작 객체(GC Root)에서 참조 Tree상 가까운 객체만 1차적으로 찾아가며 객체가 GC 대상(참조가 끊긴 객체)인지를 판단한다.
    이때 Stop the World 현상이 발생하지만 탐색 깊이가 얕아 처리시간은 매우 짧다.
  
Concurrent Mark
    Stop the World 현상 없이 진행되며, Initial Mark 단계에서 GC 대상으로 판별된 객체들이 참조하는 다른 객체들을 따라가며 GC 대상인지 추가적으로 확인한다.

Remark
    Concurrent Mark 단계의 결과를 검증한다. 
    Concurrent Mark 단계에서 GC 대상으로 추가 확인되거나 참조가 제거되었는지 등의 확인을 한다. 
    이 검증과정은 Stop the World를 유발하기 때문에 그 처리시간을 최대한 줄이기 위해 멀티 스레드로 검증 작업을 수행한다.
    
Concurrent Sweep
    Stop the World 없이 Remark 단계에서 검증 완료된 GC 객체들을 메모리에서 제거한다.
    
참고로 CMS GC는 연속적인 메모리 할당이 어려울 정도로 메모리 단편화가 심한 경우에만 Compaction 과정을 수행한다.
때문에 Stop-The-World 시간이 다른 GC보다 더 길게 걸릴 수도 있다.
```

### e. G1 (Garbage First) GC
```
하드웨어가 발전되어 JVM을 가동하는 메모리의 크기도 점점 커져가는데, 이전까지의 GC들은 큰 용량의 메모리에 적합하지 않다. 
Root set부터 순차적으로 탐색하기 때문에 용량이 클수록 탐색 시간이 길어지기 때문이다.

큰 Heap 메모리에서 짧은 GC 시간을 보장하는데 그 목적을 둔다.
G1 GC는 앞서 살펴본 Eden, Survivor, Old 영역이 존재하지만, 고정된 크기로 고정된 위치에 존재하지 않는다.

전체 Heap 영역을 Region이라는 특정한 크기로 나눠서 각 Region의 상태에 따라 역할(Eden, Survivor, Old)이 동적으로 부여된다.
보통 2048개 의 Region으로 나뉠 수 있으며 옵션을 통해 1MB~32MB 사이로 지정할 수 있다.

Java 9부터 기본 방식으로 채택됐다.
```

### f. Young GC (Miner GC)
```
Stop the World 현상이 발생하며 그 시간을 최대한 줄이기 위해 멀티스레드로 한다.

Young GC는 각 Region 중 GC대상 객체가 가장 많은 Region(Eden 또는 Survivor 역할)에서 수행되며,
이 Region에서 살아남은 객체를 다른 Region(Survivor 역할)으로 옮긴 후 비워진 Region을 사용 가능한 Region으로 돌리는 형태로 동작한다.
```

### g. Full GC (Major GC)
```
Initial Mark → Root Region Scan → Concurrent Mark → Remark → Cleanup → Copy 단계를 거치게 된다.

- 위에 내용을 Snapshot-At-The-Beginning (SATB) 알고리즘이라 한다. 

Initial Mark
    Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾는다. 이 과정에서 Stop the World 현상이 발생하게 된다.
    
Root Region Scan
    Initial Mark에서 찾은 Survivor Region에 대한 GC 대상 객체 스캔 작업을 진행한다.
    
Concurrent Mark
    전체 힙의 Region에 대해 스캔 작업을 진행하며, GC 대상 객체가 발견되지 않은 Region 은 이후 단계를 처리하는데 제외되도록 한다.

Remark
    애플리케이션을 멈추고(Stop the World) 최종적으로 GC 대상에서 제외될 객체(즉, 사용될 객체)를 식별해낸다. 여기까지 진행하면 사용되는 객체의 Marking이 완료된다.

Cleanup
    애플리케이션을 멈추고(Stop the World) 사용되는 객체가 가장 적은 Region에 대해서 사용되지 않는 객체 제거를 수행한다. 이후 Stop the World를 끝내고 앞선 GC 과정에서 완전히 비워진 Region을 별도의 리스트로 관리하고 추가해 재사용될 수 있게 한다.

Copy
    GC 대상 Region이었지만 Cleanup 과정에서 완전히 비워지지 않은 Region의 사용되는 객체들을 새로운(Available/Unused) Region에 복사하여 Compaction 작업을 수행한다.
```

### h. Z GC
```
Java 11부터 추가된 GC이며 Scalable 하고 Low Latency를 가진 GC이다.

다음의 목표를 충족하기 위해 설계됐다.

- Stop the World 처리시간이 최대 10ms를 초과하지 않음
- 힙 크기가 증가해도 Stop the World 처리시간이 증가하지 않음
- 8MB~16TB에 이르는 스펙트럼의 힙 처리가 가능
- G1 GC 보다 애플리케이션 처리량이 15% 이상 떨어지지 않을 것

Z GC는 ZPages라는 개념을 사용한다. G1 GC의 Region과 비슷한 영역의 개념이지만 Region은 고정된 크기인 것에 반해 ZPages는 크기가 2MB의 배수로 동적으로 생성 및 삭제될 수 있는 것이다.

Z GC는 목표한 속도와 안정성을 위해 Colored pointers와 Load barriers라는 주요한 알고리즘 2가지를 사용한다.

Colored pointers

    객체를 가리키는 변수의 포인터에서 64bit 메모리를 활용해 Mark를 진행하고 객체 상태 값을 저장해 사용하는 방식이다.
    때문에 64bit 운영체제에서만 작동하며 JDK11, 12까지는 4TB의 메모리만 지원하였고 JDK13에서 16TB까지 메모리 확대가 이뤄졌다.

Load barriers

    스레드에서 참조 객체를 Load 할 때 실행되는 코드를 말한다.
    Z GC는 재배치에 대해서 Stop the World 없이 동시적으로 재배치를 실행하기 때문에 참조를 수정해야 하는 일이 일어나게 되는데,
    이때 Load barriers가 아래의 순서대로 Remap Mark bit와 Relocation Set을 확인하며 참조와 Mark를 업데이트하고 올바른 참조값으로 수정해준다.
    
    
동작 방식

ZGC는 총 3번의 Pause만이 일어난다.

    Pause Mark Start : ZGC의 Root에서 가리키는 객체에 Marking 한다.
    Concurrent Mark/Remap : 객체의 참조를 탐색하며 모든 객체에 Marking 한다.
    Pause Mark End : 새롭게 들어온 객체들의 대해서 Marking 한다.
    Concurrent Pereare for Reloc : 재배치하려는 영역을 찾아 Relocation Set에 배치한다
    Pause Relocate Start : 모든 루트 참조의 재배치를 진행하고 업데이트한다.
    Concurrent Relocate : 이후 Load barriers를 사용하여 모든 객체를 재배치 및 참조를 수정한다.
```

##참고
- Allocation: 배정
- Throughput : 데이터 처리량
- 모든 구글...
