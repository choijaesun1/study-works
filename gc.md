# GC

> - Garbage Collector: 불필요한, 더 이상 사용하지 않는 메모리를 정리하여 한정적인 메모리를 효율적으로 사용하기 위한 작업.
>  - 개발자가 메모리 할당/해제를 직접 하지 않아도 된다.

## "더 이상 사용하지 않는 객체"..?

- weak generational hypothesis
  - 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
    - 특정 객체를 참조하는 객체(?)가 아무것도 없을 때 접근 불가능한 상태가 된다.
  - 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
  
  _**- Reachability 를 가지고 가비지 여부 판단**_

![](https://user-images.githubusercontent.com/16408892/58443241-cb8b6d80-812b-11e9-9451-9a2ece2709f0.png)

## GC 과정

### Heap

- Young
  - Eden
  - Survivor 1
  - Survivor 2
- Old

>- minor GC : Young 영역에서 발생
>- major GC : Old 혹은 Perm 영역에서 발생

#### 1. Eden
- 처음 생성된 객체는 Eden 영역에 지정된다. minor GC 가 발생하면 더이상 참조되지 않는 객체는 사라진다.
#### 2. Survivor
- Eden 영역에서 살아남은 객체는 Survivor 영역으로 이동한다. minor GC가 발생할 때 마다 Survivor 1 -> 2 혹은 Survivor 2 -> 1로 옮겨 가면서 불필요한 객체는 사라진다.
- 두 개의 영역 중 한 영역은 반드시 비어있어야하며, 비어있는 영역에 Eden영역에 있던 객체가 할당된다.
#### 3. Old
- 더 이상 Young 영역에 공간이 없거나 minor GC에서 살아남은 횟수 (age bit)가 일정 threshold 를 넘어가게 될 경우 Old 영역으로 이동한다.
- Old 영역에 있다가 미사용 되는 객체는 Full GC로 정리된다.

## GC 종류
> minor GC 가 발생하면 모두 stop-the-world 지만 시간이 짧음.
> 알고리즘은 방식마다 다름.

### 1. Serial GC

- 순차적으로 처리
- 한 스레드에서 수행 (옛날에는 CPU core 갯수가 많지 않았을테니..)

- Old/Perm 영역에 있는 객체들은 **Mark-sweep-compact** 콜렉션 알고리즘을 따름
  - Old 영역의 객체들 중 살아 있는 객체를 식별 (Mark)
  - Old 영역의 객체들을 훑는 작업을 수행하여 쓰레기 식별 (Sweep)
  - 필요없는 객체를 지우고 살아 있는 객체를 한곳으로 모음 (Compact)

![](https://user-images.githubusercontent.com/16408892/58443273-ea89ff80-812b-11e9-8838-1a65a0409f83.png)

### 2. Parallel GC

- minor GC를 멀티 쓰레드로 처리 (Parallel 이후로는 minor 모두 멀티스레드 사용)

![](https://user-images.githubusercontent.com/16408892/58443251-d6de9900-812b-11e9-97a7-72e2a5583889.png)

- Serial 보다는 Stop-the-world 시간이 단축

### 3. Parallel Old GC - 어드민 이거 쓰고 있음.

- Parallel GC 에서 Old GC를 개선한 버전
- Mark-**Summary**-Compaction 알고리즘 사용
  - Summary : 이전에 GC를 수행된 영역(Compaction 된 영역)에서 살아있는 객체를 식별
- Old GC도 멀티쓰레드로 병렬 수행 (thread 수는 옵션을 통해 지정 가능)

### 4. CMS GC

- stop-the-world 를 최소화 하는데 초점
- 백그라운드에서 쓰레드를 만들어서 Old 영역에 있는 필요없는 객체를 지속적으로 제거
  - 때문에 full GC 는 자주 발생하지 않음.
  - But CPU 많이 먹고 Compaction 을 파편화가 심할 경우에만 진행되기는 하지만 stop-the-world 시간이 다른 알고리즘 보다 오래 걸린다.
- Old Generation GC 알고리즘
  - Initial Mark : 살아남은 객체 탐색. Root 객체에서 가까운 객체만 1차적으로 찾으며 stop-the-world 시간이 적음.
  - Concurrent Mark : Initial Mark 한 객체들이 참조하는 객체들을 따라가면서 GC 대상을 찾음.
  - Remark : Concurrent Mark 단계에서 Mark 하는 동안 변경 된 객체를 확인하고 Remark. stop-the-world 발생하지만 multi thread 로 진행됨.
  - Concurrent Sweep : 쓰레기 정리

  ![](https://user-images.githubusercontent.com/16408892/58443260-df36d400-812b-11e9-8027-2276f3cf30f6.png)

### 5. G1(Garbage First) GC

- 큰 힙 메모리에서 짧은 GC 시간을 보장하는데 초점.
- 전체 힙 메모리 영역을 **Region** 이라는 특정한 크기로 나눠서 각 Region에 역할(Eden, Survivor, Old)이 동적으로 부여
  - Heap 은 2048개의 Region 으로 나뉘고 Region 의 size를 조절할 수 있는 옵션이 있으나 튜닝은 추천하지 않음.
- 쓸모 없는 객체만 쏙쏙 정리하는게 아니라 Region이 꽉차면 정리하여 다른 Region으로 옮기고(이 때 compacting 도 발생), 비워진 Region 을 사용가능하도록 만듦.

- ![](https://user-images.githubusercontent.com/16408892/58443255-d9d98980-812b-11e9-88c2-3e2b77ac5f2c.png)

- Humongous : Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간이며, 이 Region 에서는 GC 동작이 최적으로 동작하지 않음.
  - 이런 객체가 있다면 프로그램 설계 이슈가 없는지 봐야한다고 함.
- Available/Unused : 아직 사용되지 않은 Region을 의미한다.
- Old GC 는 Old Area에 비해 New Area가 감소할 때, 발생한다.
- Old Generation GC 알고리즘
  - Initial Mark : Old Region 에 존재하는 객체들이 참조하는 Survivor Region 을 찾음.
  - Root Region Scan : Initial Mark 에서 찾은 Survivor Region에 대한 GC 대상 스캔.
  - Concurrent Mark : 전체 힙의 Region에 대해 스캔하며 GC 대상이 없는 Region 은 이 후 단계 생략.
  - Remark : 최종적으로 GC 대상에서 제외될 객체(살아남을 객체)를 식별
  - Cleanup : 쓸모 없는 객체를 제거하고 (stop-the-world) 완전히 비워진 Region 을 사용 가능상태로 바꿈
  - Copy : GC 대상 Region에 속해 있었지만 살아남은 객체를 새로운 Region 으로 복사 하고 Compaction.

![](https://user-images.githubusercontent.com/16408892/58443425-b8c56880-812c-11e9-854c-1f59a2b9b303.png)

![](https://user-images.githubusercontent.com/16408892/58443257-dc3be380-812b-11e9-870a-e4c4dc9608d4.png)

### 6. Z GC - jdk 11 부터

- 짧은 GC 시간
  - pause time 이 10 ms을 넘지 않는다고 함.
  - Heap 크기에 따라서 pause time 이 늘어나지 않음. (Heap 이 TB 단위여도!)
- Region 기반
- **Concurrent**
  - 스레드 멈춤 없이 Marking/Relocation/Compaction 등을 수행함.
- 주요 동작
  - [Pause] Mark Start
    - Scan Thread stacks to find route objects.
  - [Concurrent] Mark/Remap
    - expensive operation
    - 전체 스캔하면서 GC 대상 Object를 찾음
  - [Pause] Mark End.
    - synchronization point to coordinate the end of the marking phase.
    - At this point jvm knows which objects are live and which are garbages
  - [Concurrent] Prepare for Relocation.
    - reference processing : the GC takes care of soft week and phantom references and finalization.
    - relocation set selection : the GC figures out what parts of the heap needs to be compacted to free up memory
  - [Pause] Relocate start
    - Scan thread stacks to handle roots pointing into the location set.
  - [Concurrent] Relocate
    - heavy work of Compaction.
    - For this, the GC will free up large chunks of memory and make that memory available

![](https://user-images.githubusercontent.com/16408892/58443281-f70e5800-812b-11e9-8043-c3328ac369d4.png)

- 지연시간 비교

![](https://user-images.githubusercontent.com/16408892/58443284-fc6ba280-812b-11e9-8dab-79535d5c39fb.png)

- **load barriers and Colored object pointers**
  - 이를 통해 애플리케이션 스레드가 동작하는 중간에, ZGC가 객체 재배치 같은 작업을 수행할 수 있게 해준다는데.. 뭔말인지 잘 모르겠음 ㅠ 다음에 알아보도록하자..

## 참조
- https://12bme.tistory.com/57
- https://mirinae312.github.io/develop/2018/06/04/jvm_gc.html
- https://plumbr.io/handbook/garbage-collection-in-java
- https://okky.kr/article/379036
- https://initproc.tistory.com/entry/G1-Garbage-Collection
- https://d2.naver.com/helloworld/
- https://www.opsian.com/blog/javas-new-zgc-is-very-exciting/
