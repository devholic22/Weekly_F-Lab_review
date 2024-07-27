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
15. [Enum](#enum)
16. [직렬화](#직렬화)

## 자바의 특징

## 자바 컴파일 과정

## Jar 파일이 실행될 수 있는 이유 (with Spring)

## JVM의 역할은?

## &와 &&의 차이점

## 배열과 ArrayList의 차이

## 자바 클래스-인스턴스 변환 과정

## new String과 쌍따옴표 방식의 차이

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

## Enum
[참고 링크](https://velog.io/@mooh2jj/Java-Enum%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)
* enum을 이용하면 `상수의 집합`으로 표현할 수 있다.
* enum 생성자는 package-private (default)이나 private으로만 가능하다.
* 값이 추가되거나 변경되는 경우 한 곳에서만 변경하면 되기 때문에 `유지보수`가 용이하다.
* enum은 태생적으로 `싱글턴`이 보장되도록 만들어졌다.
* 컴파일 시점에 `타입 안정성`을 제공한다. 특정 범위의 값만 이용할 수 있기 때문이다.
* 개인적인 생각으로는 `묻지 말고 시켜라` 전략을 시킴으로써 책임을 적절하게 할당할 수 있다는 장점이 있다고 생각한다. 예를 들어 지정된 자동차 회사에 따라 정해진 할인 금액을 조회하고자 한다면 메인 클래스에서 각 이름을 case로 작성하는 게 아니라, Enum 클래스에게 메서드로 반환하게끔 하는 것이다.
    ```java
    class enum CarCompany {
        BMW("BMW", 500),
        AUDI("AUDI", 1000);
  
        private final String name;
        private final int money;
  
        private CarCompany(final String name, final int money) {
            this.name = name;
            this.money = money;
        }
  
        public static int calculateMoney(final String name) {
            // name에 맞는 CarCompany를 조회 후 반환하도록 설정
        }
    }
    ```

## 직렬화
[참고 링크](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)  
직렬화란 자바 언어에서 사용되는 Object 또는 Data를 다른 컴퓨터의 자바 시스템에서도 사용할 수 있도록 바이트 스트림 (stream of bytes) 형태로 연속적인 (serial) 데이터로 변환하는 포맷 변환 기술이다.  
반대인 역직렬화는 바이트로 변화된 데이터를 원래대로 자바 시스템의 Object 또는 Data로 변환하는 기술이다.
### 직렬화가 필요한 이유
직렬화 (Serializable)는 내가 만든 클래스가 파일에 읽거나 쓸 수 있도록 하거나, 다른 서버로 보내거나 받을 수 있도록 할 때 필요하다.
### serialVersionUID의 용도
* Serializable 인터페이스를 구현한 후에는 serialVersionUID 값을 지정하길 권장하고 있다.
* 별도로 지정하지 않으면 자바 소스가 컴파일될 때 자동으로 생성된다.
* static final long 지정 및 변수명을 serialVersionUID로 지정해야 한다.
* 해당 객체의 버전을 명시하는 데 사용된다.
* 클래스 이름이 같더라도 이 ID가 다르면 다른 클래스라고 인식하며, 같은 ID더라도 변수의 개수, 타입 등이 다르면 다른 클래스로 인식한다.
