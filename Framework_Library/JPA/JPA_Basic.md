# JPA(Java Persistence API) 기본

#### 2021.04.09

---

1. JPA 시작

- JPA를 잘 모르면서 바로 실무에 투입할 경우?
  - 객체나 테이블을 정확하게 매핑할 수 없다. 감조차도 잡기 힘들다.
    > 1. 객체와 테이블을 제대로 설계하고 매핑하는 방법을 익혀보자. <br>
    > 2. JPA의 내부 동작 방식을 정확하게 이해하고 사용하자. <br>

2. JPA와 모던 자바 데이터 저장 기술

- 대부분이 객체지향 언어와 관계형 DB를 많이 사용함.

  - 객체를 관계형 DB를 통해 관리함.
  - 즉, 관계형 DB가 알 수 있는 SQL 중심의 코드를 짜게됨.
  - SQL에 의존적인 개발을 피하기가 어려움.
  - 관계형 DB의 슈퍼타입 / 서브타입으로 유사하게 구현해볼 수는 있다.

    - 근데, 일일이 다 쪼갰다가 합쳤다가 미친 작업량을 만날 수 있음.

      > Java Collection으로 조회를 한다면???????

    - 다형성을 활용할 수 있음.

  - 객체 지향은 객체 그래프 탐색이 가능해야 하지만, SQL에 따라서 탐색 범위가 결정되어 버림.
    - 이는 곧 Entity에 대한 **신뢰성 문제**와 연결됨.
      > 진정한 의미의 계층 분할이 어렵다.
  - Collection에서 객체를 다룰 때와 SQL에서 객체를 다룰 때가 달라짐
    > 객체답게 모델링 할수록 매핑 작업만 늘어남
  - **객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까?**
    > JPA(Java Persistence API)!!

3. JPA

- Java 진영의 ORM 표준 기술
  - ORM(Object-relational mapping): 객체는 개체대로 설계하고 관계형 DB는 관계형 DB대로 설계해서, ORM Framework가 중간에서 매핑함. 대중적인 언어에서는 ORM을 제공함
  - Application과 JDBC 사이에서 동작함.
  - EJB(Entity Bean) > Hibernate > JPA
- JPA는 Interface의 모음
  - JPA 2.1 표준 명세를 구현한 3가지 구현체(Hibernate, EclipseLink, DataNucleus)
- JPA를 왜 사용해야 하는가?
  - SQL 중심적인 개발에서 객체 중심으로 개발
  - 생산성
    - 저장: jpa.persist(member)
    - 조회: jpa.find(memberId)
    - 수정: member.setName("변경 이름")
    - 삭제: jpa.remove(member)
  - 유지보수
  - 패러다임의 불일치 해결
    - 상속: 슈퍼타입/서브타입으로 설계해놓으면 JPA가 알아서 Query를 쪼개서 처리해줌
    - 조회, 연관관계, 객체 그래프 탐색도 마찬가지
    - **동일한 Transaction에서 조회한 Entity는 같음을 보장**
  - 성능
    - 1차 캐시와 동일성(identity) 보장
    - Transaction을 지원하는 쓰기 지연(transactional write-behind)
    - 지연 로딩(Lazy Loading)
  - 데이터 접근 추상화와 벤더 독립성
  - 표준

> ORM은 객체와 RDB 두 기둥 위에 있는 기술. 즉, 둘 다 잘 알아야 한다.

4. JPA 설정하기

- JPA 설정 파일
  - /META-INF/persistence.xml 위치
  - persistence-unit name으로 이름 지정
  - javax.persistence로 시작 : JPA 표준 속성
  - hibernate로 시작 : Hibernate 전용 속성
- **JPA는 특정 데이터베이스에 종속적이지 않다!!!**
  - 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다른데(MYSQL : LIMIT / Oracle : ROWNUM), 이러한 것들은 데이터베이스 방언이라 하고, 이를 알아서 JPA가 처리해줌.
