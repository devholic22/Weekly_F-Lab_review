## JVM의 역할
* 플랫폼 독립적이게 해 줌 (바이트 코드)
* 가비지 컬렉터 관리
  ### 가비지 컬렉터
  [참고 링크 1](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)  
  [참고 링크 2](https://donghyeon.dev/java/2020/03/31/%EC%9E%90%EB%B0%94%EC%9D%98-JVM-%EA%B5%AC%EC%A1%B0%EC%99%80-Garbage-Collection/)
    * 가비지 컬렉터는 참조가 끊겨진 객체 (Unreachable)를 제거하는 방식으로 작동
    * 기본적으로 `Mark-and-Sweep` 방식으로 작동 (식별하고, 제거하며, 공간을 압축한다.)
    * `Stop The World`: 가비지 컬렉션이 자동으로 청소한다고 해도 메모리가 언제 해제되는지 알 수 없어 제어하기 힘들며, 가비지 컬렉션이 동작하는 동안에는 다른 동작을 멈추기 때문에 오버헤드가 발생되는 현상이 나타난다. 이를 Stop The World라 한다.
        * 메모리 청소가 실행되는 동안 관련 스레드를 제외한 모든 스레드는 멈추게 되며, 이는 곧 서비스 이용에 차질이 생길 수 있다. 즉 이 시간을 최소화시켜야 한다. 일례로, 익스플로러는 너무 GC가 자주 발생하여 성능 문제가 높았다.
        * Stop The World 현상은 Minor GC, Major GC에서 모두 발생한다. GC가 발생하면 GC를 제외한 모든 스레드가 멈추기 때문이다.
    * 말하자면 실시간성이 매우 강조되는 프로그램이라면 가비지 컬렉터에게 메모리를 맡기는 것은 맞지 않을 수 있다.
    * 이러한 GC 최적화 작업을 **GC 성능 튜닝**이라 한다.
    * 가비지 컬렉터의 대상이 되는 힙은 크게 `Young 영역`과 `Old 영역`, 그리고 Perm으로 나뉜다.
      #### Young 영역
        * 젊은 객체들이 존재
        * 새로 생성된 객체는 Eden 영역에 저장
        * Eden 영역이 다 차면, 살아있는 객체들만 Survivor 0 영역으로 복사 후 다시 Eden 영역 채움
        * Survivor 0 영역이 다 차면, Survivor 1 영역으로 복사. 이때 Eden 영역에 있던 다른 객체들도 복사됨
        * 즉 Survivor 0, Survivor 1 둘 중 하나는 비어 있어야 함
        * 하나의 힙 영역을 세부적으로 쪼갬으로서 객체의 생존 기간을 면밀하게 제어하여 가비지 컬렉터(GC)를 보다 정확하게 불필요한 객체를 제거하는 프로세스를 실행
        * 여기에서 발생하는 GC가 Minor GC, Young 영역이 Old 영역에 비해 더 적으므로 GC 속도 또한 빠름
      #### Old 영역
        * 늙은 객체들이 존재
        * 객체의 age가 임계값에 도달될 시 Old 영역으로 이동, Old 영역이 다 찰 시 GC 발생
        * 성능 저하의 원인
        * 여기에서 발생하는 GC가 Major GC
      #### Perm (Permanent)
        * 클래스, 메서드에 대한 정보 등이 쌓인다.
        * Java 7 까지 힙 영역에 존재, Java 8 이후에는 Native Method Stack에 편입
    * 적용 가능한 GC 알고리즘: Serial (절대 사용 X) / ConcMarkSweep (CMS) / Parallel / G1 / ZGC / Epsilon / Shenandoah
  #### 메모리 누수 (Memory Leak)
  [참고 링크](https://junghyungil.tistory.com/133)
    * 컴퓨터 프로그램이 필요하지 않은 메모리를 계속 점유하고 있는 상황
    * Old 영역에 계속 누적된 객체로 인해 Major GC 빈번히 발생, 성능 저하 및 OutOfMemory Error 발생
    * static 객체 남발, 스트림 객체 생성 후 자원 반환 X, 무의미한 객체 생성 등에서 발생 가능
