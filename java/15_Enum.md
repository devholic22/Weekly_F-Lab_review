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
