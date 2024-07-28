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
