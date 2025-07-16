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

5. JPA 구동 방식

- JPA Persistence class가 설정정보를 읽어서 EntityManagerFactory 생성
- 필요할 때마다 EneityManager 생성 & 호출
- JPA 사용할 모델(Model)에는 반드시 '@Entity' Annotation을 달아주어야 함.
  - 그리고 Id Annotation을 통해서 PK임을 명시해줘야함.
- Entity Manager는 쓰레드 간에 공유하면 안됨!!!!!
  - 사용하고 버려야 함
- JPA의 모든 데이터 변경은 Transaction 안에서 실행

```java
// App Loading 시점에서 Factory는 한 번만 생성하면 됨.
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnitName);
// EntityManager는 Trasaction 단위로 생성
EntityManager em = emf.createEntityManager();
```

6. JPQL

- 객체지향 SQL
- 가장 단순한 조회 방법으로 검색 조건이 붙는다면??
  - 검색을 할 때도 테이블이 아닌 Entity 객체를 대상으로 검색하는 것
  - 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능함
  - 필요한 데이터만 DB에서 불러오려면 결국 검색조건이 포함된 SQL이 필요
- SQL을 추상화한 JPQL이란 객체지향 쿼리 언어
  - SQL : 데이터베이스 테이블을 대상으로 쿼리
  - JPQL : Entity를 대상으로 쿼리

7. 영속성 관리

- 영속성 컨텍스트(Persistence Context)
  - 실제로 JPA가 내부적으로 어떻게 동작하는거야??
  - **'Entity를 영구 저장하는 환경'**이라는 뜻
    > Entity를 영속성 컨텍스트라는데 저장한다는 것
  - EntityManager.persist(entity);
  - 논리적인 개념으로, 눈에 보이지 않으며 Entity Manager를 통해 접근한다.
- Entity의 생명주기
  - 비영속(new / transient) : 영속성 Context와 전혀 관계가 없는 새로운 상태
  - 영속(managed) : 영속성 컨텍스트에 관리되는 상태
    - 영속 상태가 된다고 해서 **Query가 날라가는게 아님**
  - 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
  - 삭제(removed) : 삭제된 상태

```java
// 비영속
Member member = new Member();
member.setId(100L);
member.setName("hi");

// 영속
em.persist(member);
```

- 영속성 컨텍스트의 이점

  - 1차 캐시
  - 동일성(identity) 보장
    - 1차 캐시로 반복가능한 읽기(Repeatable read) 등급의 트랜잭션 격리 수준을 DB가 아닌 Application 차원에서 제공
  - 트랜잭션을 지원하는 쓰기 지연
    - 쓰기 지연 SQL 저장소에 저장해둔 SQL이 commit 시점에서 전송됨
  - 변경 감지(Dirty Checking)
  - 지연 로딩

- Flush

  - 영속성 컨텍스트의 변경내용을 데이터베이스에 반영
  - Flush 발생 시, 변경을 감지하고 수정된 Entity를 '쓰기 지연 SQL 저장소'에 등록
  - 쓰기지연 SQL 저장소의 Query를 DB에 전송
  - 직접 호출할 수도 있고, commit이나 JPQL Query를 실행 할 때 Flush를 호출 함
  - Flush를 한다 해서 1차 캐시를 지우는 것은 아님

- 준영속 상태
  - 영속 상태의 Entity가 영속성 컨텍스트에서 분리(detached)된 상태
  - 영속성 컨텍스트가 제공하는 기능을 사용할 수 없음
  - em.detach(entity) : 특정 Entity만 준영속 상태로 전환
  - em.clear() : 영속성 Context를 통으로 초기화
  - em.close() : 영속성 Context 종료

8. Entity Mapping

- 영속성 컨텍스트와 마찬가지로 가장 중요한 부분
  - 객체와 테이블 매핑 : @Entity, @Table
  - 필드와 컬럼 매핑 : @Column
  - 기본 키 매핑 : @Id
  - 연관관계 매핑 : @ManyToOne, @JoinColumn
- **@Entity**
  - JPA가 관리하는 Entity
  - JPA를 사용해서 테이블과 매핑할 클래스는 필수적으로 달아줘야 함
  - **기본 생성자는 필수!!**
- **@Table**
  - name, catalog, schema, uniqueConstraints
- **데이터베이스 스키마 자동 생성**
  - DDL을 Application 실행 시점에 자동 생성
  - 이는 개발장비에서만 사용할 것!!!!!
  - DB 방언을 사용해서 DB에 맞는 적절한 DDL 생성
    - create / create-drop / update / validate / none
    - update는 추가된 테이블 / 컬럼에만 적용됨
    - validate는 엔티티와 테이블이 정상 매핑된지 확인하는 것
  - **운영 장비에는 절대 create, create-drop, update 사용하면 안됨!!**
    - 개발 초기 : create, update
    - 테스트 서버 : update, validate
    - 스테이징, 운영 : validate, none
    - 테스트 서버라도 사용하지 않는 것 권장
  - DDL 생성 기능은 DDL을 자동 생성할 때만 사용됨
- **필드와 컬럼 매핑**
  - @Column, @Temporal, @Enumerated, @Lob, @Transient
- 기본키 매핑
  - @Id
    - 직접 할당
  - @GeneratedValue
    - 자동 할당
    - IDENTITY : DB에 위임 - MYSQL
      - **영속성 관리는 PK가 있어야 하는데, IDENTITY 전략은 Insert 해봐야 값을 알 수 있음**
      - 그래서 IDENTITY 전략은 **persist 시점에 Query를 날림**
    - SEQUENCE : DB 시퀀스 오브젝트 사용
      - @SequenceGenerator 필요
      - **IDENTITY 전략과 마찬가지로 PK가 있어야 하기 때문에, persist 시점에 Sequence에서 값을 가져와놓음**
    - TABLE : 키 생성용 테이블 사용
      - @TableGenerator 필요
      - 키 생성 전용 테이블을 하나 만들어서 DB Sequence를 흉내내는 전략으로, 모든 DB에 적용가능하지만, 성능이 단점
      - 권장 : Long 형 + 대체키 + 키 생성 전략
    - 아니, 아무튼 Network 탈 바에 한 번 INSERT 하는게 낫지 않음?
      - **allocationSize** : 메모리에 미리 정해놓은 개수만큼 땡겨놓고, 다 쓰면 다시 떙겨오기
