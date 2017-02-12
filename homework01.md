## 1. JVM 메모리 관리 메커니즘
### JVM 메모리 영역 (Runtime Data Areas)
{image}
#### - 총 6개의 영역

#### - Thread 비공유 영역 (Thread 마다 하나씩 생성):
  - _**PC Register**_ : Thread 가 시작될 때 생성되며 현재 수행중인 JVM명령의 주소를 갖는다.
  - _**JVM Stack - method call Trace 추적**_ : Thread 가 시작될 때 생성. Stack Frame 이라는 구조체를 저장하는 스택.
    - {JVM stack image}
    - *Stack Frame* :
      - JVM 내에서 메서드가 수행될 때마다 하나의 스택 Frame 생성되고 메서드 종료시 제거.
      - 지역 변수 배열, 피연산자 스택, 현재 실행 중인 메서드가 속한 클래스의 Runtime 상수 풀에 대한 Reference를 갖는다.
        - 지역 변수 배열 : index 0 = 메서드가 속한 클래스 인스턴스의 "this" Reference. 1부터는 메서드 Parameter 정보, 그 이후는 메서드의 지역변수 저장.
        - 피연산자 스택 : 메서드의 실제 작업 공간. 다른 메서드 호출 결과를 push, pop
  - _**Native Method Stack**_ : 자바 외의 언어로 작성된 네이티브 코드를 위한 스택. 언어에 맞게 C 스택이나 C++ 스택이 생성.

#### - Thread 공유 영역 (JVM이 시작될 때 생성):
 - _**Method Area**_ : JVM이 읽어 들인 각각의 클래스와 인터페이스에 대한 런타임 상수 풀, 필드와 메서드 정보, Static 변수, 메서드의 바이트코드 등을 보관. = __class, interface 정보__
   - _**Runtime Constant Pool**_ : 클래스 파일 포맷에서 constant_pool 테이블에 해당하는 영역. 어떤 **메서드나 필드** 를 참조할 때 JVM은 런타임 상수 풀을 통해 해당 메서드나 필드의 실제 메모리상 주소를 찾아서 참조.
 - _**Heap**_ : Runtime 시 동적으로 객체와 배열을 저장하는 공간으로 **가비지 컬렉션** 대상.

_참조_

## 2. JVM Full GC, Minor GC
###### _- stop-the-world : GC를 실행하기 위해 JVM이 GC 실행 Thread를 제외한 나머지 Thread 작업을 멈추고 GC가 완료 된 후 다시 시작한다._

###### _- Reachability 를 가지고 가비지 여부 판단 (http://d2.naver.com/helloworld/329631)_

### Minor GC
 - young 영역에서 객체가 사라질 때 Minor GC가 발생했다고 한다.
   - youg 영역 :
     - 새롭게 생성한 객체의 대부분이 여기에 위치.
     - 총 3개의 영역 (Eden, Survivor x2)
 - Flow :
   - 새로 생성한 대부분의 객체는 Eden 영역에 위치
   - Eden 영역에서 GC가 한 번 발생 후 살아남은 객체는 Survivor 영역중 하나로 이동. 그런 객체가 Survivor영역에 계속 쌓인다.
   - 하나의 Survivor영역이 가득차면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동하고 가득 찬 Survivor영역을 비운다.
   - 이 과정을 반복하다가 계속 살아남은 객체를 Old 영역으로 이동시킨다.

### Full GC
 - Old 영역, Perm영역 에서 객체가 사라질 때 Major GC, Full GC가 발생했다고 한다.
 - 기본적으로 데이터가 가득 차면 GC를 실행
 - 방식 (자세한 설명은 링크를 참조 : http://d2.naver.com/helloworld/1329)
   - Serial GC (mark-sweep-compaction)
   - Parallel GC
   - Parallel Old GC(Parallel Compacting GC)
   - Concurrent Mark & Sweep GC(CMS)
   - G1(Garbage First) GC

### Old 영역의 객체가 Young 영역의 객체를 참조한다면?
 - old 영역의 512바이트의 chunk로 되어있는 **card table** 존재.
 - minor gc 발생 시 카드 테이블을 참조하여 GC 대상 식별.

#### GC 튜닝
 - 참조 (http://d2.naver.com/helloworld/37111)

## 3. DB Connection Pool

###### 참조 : http://d2.naver.com/helloworld/5102792

### DB Connection Pool?
 - 데이터베이스와 연결된 커넥션을 미리 만들어서 풀(pool) 속에 저장해 두고 있다가 필요할 때 커넥션을 풀에서 쓰고 다시 풀에 반환하는 기법.
 - 웹 애플리케이션의 요청은 대부분 DBMS로 연결되기 때문에 커넥션 풀 라이브러리의 설정은 전체 애플리케이션의 성능과 안정성에 영향을 미친다.

### 남아있는 Connection이 없다면?
 - 해당 클라이언트는 대기 상태로 전환이 되고, 커넥션이 반환되면 대기하고 있는 순서대로 커넥션이 제공
 - 사용자가 몰려서 커넥션을 많이 사용할 때는 maxActive 값이 충분히 크지 않다면 병목 지점이 될 수 있다.

### Commons DBCP

#### - Connection Pool의 저장 구조
Timestamp, Connection의 레퍼런스를 한쌍으로 하는 ObjectTimestampPair라는 자료구조를 CursorableLinkedList로 관리한다.(LIFO)

#### - Connection 개수 관련 속성
BasicDataSource 의 Setter로 설정할 수 있다. 커넥션 개수와 관련된 가장 중요한 성능 요소는 일반적으로 커넥션의 최대 개수이다.
 - initialSize : getConnection() 호출 시 Connection pool에 채워 넣을 Connection 수
 - maxActive : 동시에 사용할 수 있는 최대 Connection 수 (default : 8)
 - maxIdle : 반납 시 최대로 유지될 수 있는 Connection 수 (default : 8)
 - minIdle : 최소한으로 유지할 수 있는 Connection 수 (default : 0)
 - _maxActive 개수 설정 시 고려해야할 점:_
   - DBMS 가 수용할 수 있는 Connection 수
   - 애플리케이션 서버의 개수,
   - Apache, Tomcat에서 동시에 처리할 수 있는 사용자 등..

#### TPS (Transaction per seconds)
초당 처리할 수 있는 요청?Transaction의 수?
 - ex) maxActive = 5, 평균 Query 1개 처리 속도 = 50ms, 1개의 요청에 10개의 쿼리가 실행 된다고 가정.
   - 1개의 요청이 처리되는 시간 = 500ms
   - Connection = 5개이므로 500ms 동안 5개의 요청을 처리할 수 있음.
   - **1초에 10의 요청 처리 = 10TPS**

##### _Connection 수를 늘리면 TPS를 높일 수 있지 않을까?_
DBMS의 리소스는 다른 서비스와 공유해 사용하는 경우가 많기 때문에 DBMS가 못버팀.
따라서 maxWait 값이 중요함.

#### MaxWait(Connection 이 반환될 때까지 이만큼 기다리겠다) Value
 - Too Long : 사용자가 인내할 수 있는 시간을 넘어서는 maxWait 값은 아무런 의미가 없다. 이미 요청자는 페이지를 떠났을 것임
 - Too short : 과부하 시 커넥션 풀에 여분의 커넥션이 없을 때마다 오류가 반환

###### 따라서 애플리케이션에서 주로 실행되는 쿼리의 성격, 사용자가 대기 가능한 시간, DBMS와 애플리케이션 서버의 자원, 발생 가능한 예외 상황, Tomcat 설정 등 많은 요소를 고려하여 설정해야함!

## 4. Tomcat Thread Pool
## 5. Tomcat Configuration, option, 성능 등
## 6. PaaS, SaaS
## 7. GTM, GSLB
