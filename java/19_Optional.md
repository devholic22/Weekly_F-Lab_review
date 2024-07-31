## Optional
[참고 링크 1](https://mangkyu.tistory.com/70)  
[참고 링크 2](https://mangkyu.tistory.com/203)
```java
public final class Optional<T> {
    private final T value;
    
    ...
}
```
* Optional은 Java8에서부터 제공하는 용법이다.
* NPE (NullPointerException)을 더 쉽게 방지할 수 있도록 도와준다.
### Optional.empty()
* Optional.empty()는 값이 없는 상태로 생성한 것이다.
* 내부적으로 static 변수로 EMPTY 객체를 미리 생성하고 있기에, 빈 객체를 여러 번 생성해야 하는 경우 객체를 공유함으로써 메모리를 절약하고 있다.
```java
Optional<String> optional = Optional.empty();
// optional.isEmpty() // true
```
### Optional.of()
* Optional.of()는 데이터가 절대 null이 아닐 경우 사용할 수 있다.
* null을 저장하려고 할 시 NPE가 발생한다.
```java
Optional<String> optional = Optional.of("hello"); 
```
### Optional.ofNullable()
* Optional.ofNullable()은 데이터가 null일 수도 있고 null이 아닐 수도 있을 경우에 쓴다.
* orElse, orElseGet을 이용해서 값이 없을 때에도 안전하게 값을 가져올 수 있다.
```java
Optional<String> optional = Optional.ofNullable(getName());
String name = optional.orElse("anon");
```
#### orElse
* orElse는 파라미터로 값을 받는다.
* 값 타입으로 넘기기에, 함수를 전달했다면 함수가 실행된다.
* value가 있더라도 함수가 실행되기 때문에 장애가 발생할 수 있다.
```java
public T orElse(T other) {
    return value != null ? value : other;
}
```
#### orElseGet
* orElseGet은 함수형 인터페이스를 받는다.
* ofNullable로 가졌던 value가 null이 아닐 경우에는 other.get()이 호출되지 않는다.
```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```
### 람다와의 활용
Optional과 람다를 함께 이용하면 가독성 좋은 코드를 작성할 수 있다.
```java
List<String> names = Optional.ofNullable(getNames())
        .orElseGet(() -> new ArrayList<>());
```
### Optional 단점 / 올바른 사용법
* Wrapper 클래스이기에 이 과정에서 오버헤드가 있다.
  * Optional은 객체를 감싸는 컨테이너라 Optional 객체 자체를 저장하기 위한 메모리가 추가로 필요하다.
  * Optional 안에 있는 객체를 얻기 위해서는 Optional을 한번 통과해야 하므로 접근 비용이 증가한다.
* 절대 null이 아니라면 사용하지 않는 게 좋다.
* Optional은 파라미터로 넘어가는 게 아니라 반환 타입으로써 제한적으로 사용되도록 권장되었다.
  * 파라미터로 넘어간다면 값이 존재하는지 여부를 또 호출해야 한다.
* Optional에 값이 있는지 검사를 진행하지 않은 채 값을 꺼내려 한다면 NPE 대신 NoSuchElementException 발생
* 직렬화를 지원하지 않아, 직렬화 객체에 필드로 가질 수 없다.
  * Jackson, Spring의 캐시 추상화 기술들은 wrap/unwrap 기능을 제공한다.
* 컬렉션의 경우 Optional보다 빈 컬렉션을 이용하라.
* 가급적이면 isPresent & get으로 값을 꺼내기보다는 orElseGet(기본값)을 활용하라.
