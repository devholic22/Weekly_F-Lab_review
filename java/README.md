# Java

## 목차
1. [자바의 특징](#자바의-특징)
2. [자바 컴파일 과정](#자바-컴파일-과정)
3. [Jar 파일이 실행될 수 있는 이유](#jar-파일이-실행될-수-있는-이유-with-spring)
4. [JVM의 역할은?](#jvm의-역할은)
5. [&와 &&의 차이점](#와-의-차이점)
6. [배열과 ArrayList의 차이](#배열과-arraylist의-차이)
7. [자바 클래스-인스턴스 변환 과정](#자바-클래스-인스턴스-변환-과정)
8. [기본 생성자가 필요한 이유는?](#기본-생성자가-필요한-이유는)
9. [new String과 쌍따옴표 방식의 차이](#new-string과-쌍따옴표-방식의-차이)
10. [상속과 Composition의 차이](#상속과-composition의-차이)
11. [싱글톤 패턴의 스레드 안정성을 확보하는 방법](#싱글톤-패턴의-스레드-안정성을-확보하는-방법)
12. [static 블록의 사용 예](#static-블록의-사용-예)
13. [어노테이션](#어노테이션)
14. [try-catch-finally vs try-with-resources](#try-catch-finally-vs-try-with-resources)

## 자바의 특징
* 객체 지향을 위해 설계된 언어
* 운영체제에 독립적
* 멀티 스레드 이용 가능
  * 동시성과 병렬성의 차이
* 메모리 자동 관리

## 자바 컴파일 과정
1. `javac 파일.java`
2. 바이트 코드 파일 생성됨 (`파일.class`)
3. `java 파일`, 프로그램 실행됨

### 자바 컴파일 과정이 있는 이유, 목적은 무엇일까?
* JVM이 애플리케이션 `코드가 모든 플랫폼에 독립적으로 운용`되게끔 하기 위해 (독립된 코드인 바이트 코드를 만들기 위해)
* (자바를 떠나서) 모든 컴파일의 목적은 결국 `우리가 작성한 코드를 컴퓨터가 이해할 수 있는 기계어로 변환`하기 위함
* 문법적 오류, 타입 체크, 의존성 확인 등 (컴파일 예외)을 위해
* 바이트 코드만 있기 때문에 역공학을 어렵게 만들고, 이는 코드 보호와 보안에 도움을 줄 수 있음
* 소스 코드를 함께 배포할 필요가 없음. 사용자가 쉽게 프로그램을 이용할 수 있도록 해 줌

## Jar 파일이 실행될 수 있는 이유 (with Spring)
[참고 링크](https://elsboo.tistory.com/27)
1. gradle의 bootJar 명령어를 실행하면 jar 파일이 생성됨
2. jar 파일은 BOOT-INF, META-INF, org/springframework/boot/loader로 구성
    ### BOOT-INF
    * 우리가 개발한 소스 코드 영역
    * `classes`: JVM이 읽을 수 있도록 컴파일 된 (바이트 코드) 클래스 파일이 존재
    * `lib`: 코드 실행에 필요한 디펜던시 라이브러리들이 존재
    * `classpath.idx`: 스프링 부트가 lib에 있는 외부 라이브러들을 찾을 수 있도록 클래스 경로 명시
    * `layers.idx`: 도커에서 jar 이미지를 만들 때 쓰이는 정보
    ### META-INF
    * 스프링 부트와 우리가 개발한 소스 코드를 연결해 주는 영역
    * `MANIFEST.MF`: jar 파일이 실행될 때 `JarLauncher`가 호출되는데, 여기에서 우리가 개발한 Start-Class (`MainApplication`)를 호출하기 위해 필요한 경로 설정 파일
    * 즉 jar 실행 명령어를 입력하면, `MANIFEST.MF`의 Main-Class인 JarLauncher를 호출하고, JarLauncher가 MANIFEST.MF의 Start-Class인 MainApplication을 호출한다.
    * 여기에서 `프레임워크와 라이브러리의 차이`가 나온다. 흐름 (호출) 통제권이 프레임워크에 있다. 라이브러리는 호출을 개발자가 한다.
    ### org/springframework/boot/loader
    * 스프링 부트 영역
    * spring-boot-loader (gradle이 빌드할 때 넣어줌) 를 통해 jar 안의 jar (nested jar)도 로드되도록 함
3. JarLauncher의 main 메서드 실행, 최상위 부모인 Launcher의 launch 메서드 실행
    ```java
   // JarLauncher
    public static void main(String[] args) throws Exception {
        (new JarLauncher()).launch(args);
    }
    ```
4. main 메서드 클래스명과 클래스 로더를 생성함. 이 기능 구현은 자식 클래스인 ExecutableArchiveLauncher에서 구현됨
    ```java
   // Launcher
    protected void launch(String[] args) throws Exception {
        if (!this.isExploded()) {
            Handlers.register();
        }

        try {
            // ExecutableArchiveLauncher
            ClassLoader classLoader = this.createClassLoader((Collection)this.getClassPathUrls());
            String jarMode = System.getProperty("jarmode");
            String mainClassName = this.hasLength(jarMode) ? JAR_MODE_RUNNER_CLASS_NAME : this.getMainClass();
            this.launch(classLoader, mainClassName, args);
        } catch (UncheckedIOException var5) {
            UncheckedIOException ex = var5;
            throw ex.getCause();
        }
    }
    ```
5. 스레드에 클래스 로더를 지정하며, 런타임 시점에 `리플렉션`을 사용하여 메인 클래스에서 main 메서드를 찾아 호출
    ```java
   // Launcher
    protected void launch(ClassLoader classLoader, String mainClassName, String[] args) throws Exception {
        Thread.currentThread().setContextClassLoader(classLoader);
        Class<?> mainClass = Class.forName(mainClassName, false, classLoader);
        Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
        mainMethod.setAccessible(true);
        mainMethod.invoke((Object)null, args);
    }
   ```
정리하자면 jar 파일 실행 - JarLauncher main 메서드 - Launcher launch 메서드 - 클래스 로더 생성 후 스레드에 클래스 로더를 지정하고 런타임 시점에 리플렉션을 사용하여 메인 클래스에서 main 메서드를 찾게 된다. 이 과정이 이루어지기 위해서는 우리가 작성한 파일이 지정되어 있는 META-INF 파일이 필요하다.

## JVM의 역할은?
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

## &와 &&의 차이점
[참고 링크](https://itbeginner2020.tistory.com/14)
* & 연산자와 && 연산자 모두 여러 개의 조건을 하나로 연결하는 AND 연산자 (모두 true여야 true)
* & 연산자: 앞의 조건식이 false 여도 뒤의 조건식을 판별
* && 연산자: 앞의 조건식이 false일 경우 뒤의 조건식을 판별하지 않음
이때 무조건적으로 && 연산자를 쓰는 것은 바람직하지 않다.
```java
String a = "hi";
String b = null;
if (a.equals("hh") & b.equals("b")) {
    System.out.println("true");
} else {
    System.out.println("false");    
}
```
이때 첫 번째 조건식인 `a.equals("hh")`는 false이므로 AND 연산자 특징 상 false가 나올 것 같지만, 실제로는 두 번째 조건까지 검사하기 때문에 NPE가 발생한다.  
반면 && 연산자는 false가 나온다.
```java
String a = "hi";
String b = null;
if (a.equals("hh") && b.equals("b")) {
    System.out.println("true");
} else {
    System.out.println("false");    
}
```
또한, & 연산자는 비트 단위 연산에서도 쓸 수 있다. (&&은 불가능)
```java
int a = 5; // 0101
int b = 3; // 0011
int result = a & b; // 0001 (둘이 같아야만 1)
```
정리하자면 & 연산자는 비트 단위 연산이 필요하거나, 조건식에 사용되는 모든 객체가 null이 아니어야만 할 때 사용할 수 있다. 다만 모든 조건을 검사하므로 비효율적일 수도 있다.  
반면 && 연산자는 논리 연산에서만 가능하며, 조건식에 사용되는 일부 객체가 null이더라도 앞 조건에서 이미 false가 가능할 때 사용하면 좋다. 안전성과 효율성에 대해서 적절한 선택을 하면 될 것 같다.
## 배열과 ArrayList의 차이
[참고 링크 1](https://inpa.tistory.com/entry/JAVA-%E2%98%95-ArrayList-%EA%B5%AC%EC%A1%B0-%EC%82%AC%EC%9A%A9%EB%B2%95)  
[참고 링크 2](https://chunsubyeong.tistory.com/82)  
[참고 링크 3](https://baby-care-dev.tistory.com/52)
### 배열
* 고정 길이 (새로운 데이터를 추가할 경우, 사이즈를 늘린 새로운 배열을 만든 뒤 복사해야 함)
* primitive type, reference type 모두 가능하다. (null 값 저장 가능)
* index에 위치한 데이터를 삭제하더라도 해당 index는 빈 공간으로 남는다.
  * 배열의 크기가 고정되어 있기 때문에 너무 큰 크기로 설정했을 경우 메모리 낭비가 될 수 있다.
* Arrays 메서드를 이용할 수 있음
### ArrayList
* 가변 길이
  * 기본 길이는 10
  * 데이터를 추가할 경우 (add) 기존 데이터 복사 과정 (`System.arraycopy`)이 발생함
  * 데이터를 추가할 때 크기가 1.5배 증가된다.
  * 초기 길이 (initial capacity)를 주지 않은 ArrayList는 첫 번째 add 메서드가 실행되어야만 default capacity 10이 적용 (lazy loading)
  * add 메서드를 실행할 때 마다 capacity가 size와 같은지 비교하고, 같다면 1.5배 늘리기 때문에 (기존 요소 복사 발생) 처음부터 데이터의 크기를 고려해서 initial capacity를 지정해주는 게 좋다.
* reference type만 가능
  * null 값 저장 가능
  * 내부적으로 `Object[]`를 가짐
* Collections 메서드를 이용할 수 있음
* 배열과 비교했을 때 원소에 접근하는 get 메서드가 [시간이 약 5배 차이](https://velog.io/@yoojkim/java-Collection-vs-Array)날 수 있음
### 정리하자면
* 배열을 쓰는 경우
  * primitive type을 배열로 관리해야 할 경우
  * 배열의 크기가 변할 일이 없는 경우
    * 크기가 낭비될 수 있음에 주의
  * Arrays 메서드를 사용해야 하는 경우
    * 배열 요소를 출력하는 경우
  * 타입 안정성을 보장하지 않아도 되는 경우 (제네릭을 이용하지 못함)
* ArrayList를 쓰는 경우
  * 배열의 크기가 가변적으로 변할 경우
    * 크기가 다 찰 시 복사 과정이 발생함에 주의
  * Collections 메서드를 사용해야 하는 경우
    * 최솟값, 최댓값을 구하는 경우
    * 원소들을 섞어야 하는 경우
    * 원소들을 반대로 전환해야 하는 경우 (배열에서도 가능하지만, 배열은 박싱되어 있어야 함)
  * 타입 안정성을 보장받아야 하는 경우 (제네릭 이용 가능)
## 자바 클래스-인스턴스 변환 과정
[참고 링크](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%96%B8%EC%A0%9C-%EB%A9%94%EB%AA%A8%EB%A6%AC%EC%97%90-%EB%A1%9C%EB%94%A9-%EC%B4%88%EA%B8%B0%ED%99%94-%EB%90%98%EB%8A%94%EA%B0%80-%E2%9D%93)  
### 클래스 로더
클래스 로더는 컴파일 된 자바의 클래스 파일 (.class)을 `동적으로 로드`하고, JVM의 메모리 영역인 Runtime Data Areas에 배치하는 작업을 수행한다.
1. Loading (로딩): 클래스 파일을 가져와서 JVM의 메모리에 로드
    * 한 번에 메모리에 올리지 않고 애플리케이션에서 필요한 경우 동적으로 메모리에 적재
    * static 멤버/메서드들이 있더라도 클래스가 사용되지 않는다면 해당 클래스는 로드되지 않음 (static 멤버/메서드를 호출하면 인스턴스화하지 않아도 클래스가 로드됨)
    * 클래스를 인스턴스화하면 클래스가 로드되며, 내부 클래스는 인스턴스를 생성하지 않으니 로드되지 않는다.
    * 상수 (static final)를 호출할 경우 클래스가 로드되지 않음 - 상수는 JVM의 Method Area에 있는 Constant pool에 따로 저장되어 관리되기 때문
2. Linking (링킹): 클래스 파일을 사용하기 위해 검증
3. Initialization (초기화): 클래스 변수들을 적절한 값으로 초기화
   * 클래스 초기화 작업은 한 번만 수행된다.
   * 멀티 쓰레드 환경에서 여러 개의 쓰레드가 동시에 클래스를 인스턴스화 하여도 클래스 초기화는 한 번만 수행된다.
### 내부 클래스 주의사항
* 단순한 내부 클래스는 외부 클래스를 먼저 생성한 뒤에야 인스턴스화 할 수 있기 때문에 외부 클래스와 내부 클래스 모두 로드된다.
* 그러나 내부 클래스는 내부적으로 외부 클래스를 참조하고 있기 때문에, 외부 클래스가 필요없고 내부 클래스만 필요한 경우에도 외부 클래스가 계속 남아있게 되어 메모리 누수가 발생한다.
* 그렇기 때문에 내부 클래스는 static 내부 클래스로 만들어야 한다. static 내부 클래스는 생성 시 외부 클래스를 호출하지 않는다.
## 기본 생성자가 필요한 이유는?
[참고 링크](https://colour-my-memories-blue.tistory.com/16)
* 리플렉션이 생성자의 파라미터 정보를 가져올 수 없다는 것은 오해 - Java 8 이후부터는 가능하기 때문
* 생성자가 많은 경우 프레임워크, 라이브러리 등이 어떤 생성자를 이용해야 할 지 판단하기 어렵다.
* 기본 생성자를 이용하도록 강제하면 판단하기 간단하다.
* 생성자에 로직이 있을 경우 원하는 값을 넣을 수 없다.
* [Hibernate 공식 문서](https://docs.jboss.org/hibernate/core/3.5/reference/en/html/persistent-classes.html)에서는 다음과 같은 이유 때문에 필요하다고 하고 있다.
    > All persistent classes must have a default constructor (which can be non-public) so that Hibernate can instantiate them using Constructor.newInstance(). It is recommended that you have a default constructor with at least package visibility for runtime proxy generation in Hibernate.  
모든 영속 클래스들은 기본 생성자 (non-public이 가능한 - private은 상속을 불가능하게 만들기 때문에 불가능)를 가지고 있어야 합니다. 그래야 Hibernate가 그들을 Constructor.newInstance() 메서드를 이용해 초기화할 수 있기 때문입니다. Hibernate에서 런타임 프록시 생성을 위해 최소한 패키지 가시성을 갖춘 기본 생성자가 있는 것이 좋습니다.
## new String과 쌍따옴표 방식의 차이
[참고 링크 1](https://velog.io/@ditt/Java-String-literal-vs-new-String)  
[참고 링크 2](https://velog.io/@mooh2jj/%EC%9E%90%EB%B0%94%EC%9D%98-String-%EC%83%9D%EC%84%B1-%EB%B0%A9%EC%8B%9D-%EB%A6%AC%ED%84%B0%EB%9F%B4%EB%B0%A9%EC%8B%9D-vs-new-%EC%97%B0%EC%82%B0%EC%9E%90-%EB%B0%A9%EC%8B%9D)  
[참고 링크 3](https://simple-ing.tistory.com/3)
### new String 방식
* 생성자를 이용하여 객체를 만들기 때문에 항상 새로운 String 인스턴스가 생성됨
* String.intern(): String pool에서 해당 문자열이 존재하는지 체크하고, 존재하면 해당 문자열을 반환, 존재하지 않으면 리터럴을 String pool에 넣은 뒤 반환함
### 쌍따옴표 방식 (리터럴)
* 기존에 있던 문자열일 경우 재사용
* 객체 생성이 아닌, Constant String pool을 참조
#### Constant String pool
* 자바 힙 메모리 내에 문자열 리터럴을 저장한 공간 (HashMap 구현)
* 리터럴로 문자열을 생성하면
  * String pool에 같은 값이 있는지 조회
  * 같은 값이 있다면 그 참조값이 반환됨 - 즉, 메모리를 절약할 수 있음
  * 같은 값이 없다면 String pool에 문자열이 등록된 후 해당 참조값이 반환됨
* OOM (Out Of Memory) 현상
  * Java 6 이하에서는 Perm 영역에 보관됨 - 고정된 사이즈, 런타임 시 사이즈가 확장되지 않음
  * Java 7 이상에서는 Perm 영역 대신 힙 영역에 들어가게 되었고, String pool의 모든 문자열도 GC의 대상이 됨
  * Java 8에서는 Perm 영역이 사라지고 MetaSpace가 이를 대체
### 1억 개의 서로 다른 문자열을 관리할 때 어떤 방식이 그나마 더 좋을까?
#### new String 방식
```java
    long beforeTime = System.currentTimeMillis();
    for (int i = 1; i <= 100_000_000; i++) {
        new String(i + "");
    }
    long afterTime = System.currentTimeMillis();
    long secDiffTime = (afterTime - beforeTime)/1000;
    System.out.println("소요 시간: " + secDiffTime + "초");
    Thread.sleep(10000);
```
* 소요 시간: 3초
* 힙 점유율: 사이즈 322,961,408B, 점유 28,668,656B = 8.87%
#### 리터럴 방식
```java
    long beforeTime = System.currentTimeMillis();
    for (int i = 1; i <= 100_000_000; i++) {
        String a = (i + "");
    }
    long afterTime = System.currentTimeMillis();
    long secDiffTime = (afterTime - beforeTime)/1000;
    System.out.println("소요 시간: " + secDiffTime + "초");
    Thread.sleep(10000);
```
* 소요 시간: 2초
* 힙 점유율: 사이즈 268,435,456B, 점유 11,446,144B = 4.26%

개인적인 생각으로는 String은 직접 힙 영역에 생성되고 리터럴은 힙 영역의 Constant pool에 저장되는 것이다보니 생성 비용으로 봤을 때 리터럴이 그나마 상대적으로 저렴한 방식이라는 생각이 든다.  
더욱이 생성자 방식은 생성자를 호출하는 것이다보니 객체 생성 관점에서도 비용이 발생할 것이라 생각한다.
## 상속과 Composition의 차이
[참고 링크](https://colevelup.tistory.com/2)  
* 상속은 상위 객체로부터 하위 객체가 물려받는 방식이고, Composition (조합)은 클래스를 구성하는 부분의 합으로 정의한다.
* 상속은 상위 클래스를 변경하면 코드 손상의 위험이 있기 때문에 상속을 통해 생성된 클래스와 객체는 강하게 결합 (tightly coupled)되어 있다.
  * 만약 상위 클래스 설계자가 확장 (extends)을 충분히 고려하지 않거나, 문서화를 하지 않거나 서로 다른 패키지 안에 있는 클래스들이라면 건드리지 않은 하위 클래스에서 오작동이 발생할수도 있다.
* 조합은 느슨하게 결합되어 있어, 코드를 손상시키지 않고 구성 요소들을 바꿀 수 있다.
### 상속이 적합한 경우
* 서브 클래스가 메인 클래스의 특별한 (specialized) 버전인 경우에 사용할 수 있다. (Vehicle - Car)
* is a, is a kind of 관계
* 나쁜 예로, SampleExample 클래스가 HashSet을 상속받을 필요는 없다. 사용하지 않을 많은 메서드들을 상속받게 되기 때문이다.
### 조합이 적합한 경우
* 객체가 또다른 객체를 점유 (has)하거나 구성 (is part of)하는 경우 사용할 수 있다. (Person - heart)
* has, is part of 관계
* 위의 SampleExample 클래스에서는 HashSet을 상속받기보다 필드로써 의존하는 게 더 유리하다.
### 개인적인 생각
* 의존성을 완전히 제거하려고 한다면, 조합보다는 메서드로써만 통신하는 게 더 낫지 않을까 싶다. 일례로 SampleExample에서 HashSet을 호출하기 위해 필요한 메서드에 단순 파라미터들만 넣어두는 것이다.  
* 또는, 인터페이스로 추상화 할 경우 조합을 활용하는 방법으로 한다면 기존 방식보다 더 나을 것 같다.
## 싱글톤 패턴의 스레드 안정성을 확보하는 방법
## static 블록의 사용 예
[참고 링크 1](https://velog.io/@limjaewoo/Java-static)  
[참고 링크 2](https://kellis.tistory.com/127)
### 장점
* 고정 메모리 - 메모리를 효율적으로 사용할 수 있다.
* 빠른 속도 - 객체를 생성하지 않고 사용하므로 빠르다.
### 단점
* 무분별한 static의 사용은 메모리 누수 (leak)의 원인
* 객체지향적이지 않다. 예컨대 클래스가 가지고 있는 변수들을 public static으로 열어둔다면 캡슐화가 이루어지지 않는다. (물론 한편으로는 아래 Math.PI처럼 다른 클래스들에서 쉽게 사용할 수 있도록 해야 한다면 논외일 것 같다.)
* 개인적인 생각으로는 테스트를 작성하기에도 난감할 것 같다. Service에서 사용하는 Repository의 메서드를 static으로 만들면 클래스 자체를 의존해야 한다.
  * `Car.from(...)`처럼  정적 팩터리 메서드를 이용하여 객체를 생성하는 경우에는 괜찮다고 생각이 든다. 이러한 경우는 보통 Car 도메인에 대한 테스트를 작성하는 편이기 때문이다.
  * 그러나 Repository 같은 클래스는 여러 구현체로 나뉠 수 있을 여지가 있다. 그런 경우 (추상화와 다형성이 필요한 경우)에는 적합하지 않아 보인다.
* thread-safe하지 않다. 전역으로 관리되기 때문에 기본적으로 thread-safe하지 않으며, synchronized를 사용하여 보장하기 위해서는 추가 작업이 필요하다.
* static 메서드는 재정의를 할 수 없다.
### 사용이 적절한 경우
* 정적 팩터리 메서드 - 다양한 의도/이름을 붙이기 위한 생성자를 만드는 경우
* Math.PI처럼 완전히 불변이면서 자주 사용되는 값이 필요한 경우
### 사용이 적절하지 않을 경우
* 인터페이스의 구현체로 재정의할 메서드인 경우
* 테스트가 진행되어야 할 경우 (단위 테스트 제외)
### static block
[참고 링크](https://hibiskim.tistory.com/9)
* static block은 한 번만 수행된다.
* static block에서는 클래스 변수만 사용할 수 있다. 메서드 또한 내부에 있는 다른 메서드를 호출하면 그 메서드도 static이어야 한다.
* 한 번만 수행되기에, 클래스 초기화 때 꼭 수행되어야 할 작업이 있을 때 유용하게 사용할 수 있다.
## 어노테이션
[참고 링크](https://ittrue.tistory.com/156) + 자바의 정석 1권
* 어노테이션 (annotation)은 쉽게 말해 프로그램 (컴파일러)에게 부가적인 정보를 제공하는 것이다. 주석은 사람에게 추가정보를 제공하지만, 어노테이션은 컴파일러에게 알리는 데 주 목적이 있다.
### 기본 어노테이션
* `@Override`: 재정의
* `@Deprecated`: 앞으로 사용하지 않을 것임을 알림
* `@SuppressWarning`: 컴파일러가 경고 메시지를 내보내지 않음
### 역할 및 특징
* 컴파일러에게 문법 에러를 체크하도록 정보를 제공한다.
* 프로그램을 빌드할 때 코드를 자동으로 생성할 수 있도록 정보를 제공한다.
* 런타임에 특정 기능을 실행하도록 정보를 제공한다. (스프링의 경우 주로 활용)
* 상속 (extends)할 수 없다.
* `@interface`로 생성한다.
### 속성 정보
#### @Target
* 적용 대상을 지정한다.
* `CONSTRUCTOR`: 생성자 선언 시
* `FIELD`: enum 상수를 포함한 필드(field) 값 선언 시
* `LOCAL_VARIABLE`: 지역 변수 선언 시
* `METHOD`: 메소드 선언 시
* `PACKAGE`: 패키지 선언 시
* `PARAMETER`: 매개 변수 선언 시
* `TYPE`: 클래스, 인터페이스, enum 등 선언 시
#### @Retention
* 얼마나 오래 어노테이션 정보가 유지되는지를 선언한다.
* `SOURCE`: 컴파일 시 사라진다. (ex: `@Override`)
* `CLASS`: 클래스 파일에 있는 어노테이션 정보가 컴파일러에 의해서 참조 가능하다. 하지만, 가상 머신 (Virtual Machine)에서는 사라진다. (ex: `lombok`)
* `RUNTIME`: 실행 시 어노테이션 정보가 가상 머신에 의해서 참조 가능하다. (ex: `@Service`, `@Transactional`)
#### @Documented
* 해당 어노테이션에 대한 정보가 Javadocs API 문서에 포함된다는 것을 선언한다.
#### @Inherited
* 모든 자식 클래스에서 부모 클래스의 어노테이션을 사용 가능하다는 것을 선언한다.
## try-catch-finally vs try-with-resources
[참고 링크](https://mangkyu.tistory.com/217)
### try-catch-finally

```java
public static void main(String[] args) {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("file.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        // catch
    } finally {
        // close
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}
```
* Java7 이전 일반적으로 사용했던 방식
* 사용 후 반납해주어야 하는 자원들은 Closable 인터페이스를 구현하고 있으며 사용 후에 close 메서드들을 호출했어야 함
* try 및 finally 작업에서 예외가 발생할 경우 스택 트레이스는 finally에서의 예외만 보여줌
### try-with-resources
```java
try (FileReader fr = new FileReader("file.txt")) {
    int i;
    while ((i = fr.read()) != -1) {
        System.out.println((char) i);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```
* Java7 이후 등장
* 기존 try-catch-finally의 문제: 자원 반납에 의한 번거로운 코드, 실수/에러로 자원을 반납하지 못하는 경우 발생
* AutoClosable 인터페이스를 구현하고 있는 자원에 대해 try-with-resources를 적용하도록 함
* finally를 작성하지 않아도 된다.
* 기존 Closable이 AutoClosable을 상속하도록 설정 - `하위 호환성 100% 달성`을 위함
* try 및 finally 작업에서 예외가 발생할 경우 모든 발생한 예외를 보여줌
* 여러 finally 작업을 할 때 한 자원에서 예외가 발생할 경우 다른 자원의 close를 제대로 하지 못했던 문제를 해결해 줌 - 컴파일러가 finally에서의 모든 경우를 try-catch-finally로 변환
