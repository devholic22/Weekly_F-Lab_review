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
