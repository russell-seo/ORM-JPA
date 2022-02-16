# JPA 기본 정리

 - [JPA-Entity 매핑](#entity-매핑)
 - [Fetch Join](#페치-조인)
 - [기본키 전략](#기본-키-매핑)
  
## Entity 매핑
속성 : name
  * JPA에서 사용할 엔티티 이름 지정
  * 기본값 : 클래스 이름을 그대로 사용
  * 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.

## 1.Table 매핑
  @Table은 엔티티와 매핑할 테이블 지정
  
  * name - 매핑할 테이블 이름
  
  * catalog - 데이터베이스 catalog 매핑
  
  * schema - 데이터베이스 schema 매핑
  
  * uniqueConstraints - DDL 생성 시에 유니크 제약조건 생성
  
## 2. 데이터베이스 스키마 자동생성 - 주의

* 운영 장비에는 절대 create, create-drop, update 사용 X

* 테스트 서버는 update 또는 validate

* 스테이징과 운영 서버는 validate 또는 none

## 3.DDL 생성 기능

* 제약조건 추가 : 회원 이름은 필수, 10자 초과 X

  @Column(nullable = false, length = 10)
  
* 유니크 제약조건 추가
  
  @Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE} )})
  
  * DDL 생성 기능은 DDL을 자동 생성할 때 만 사용
  JPA의 실행 로직에는 영향을 주지 않음
  
## 4. 매핑 어노테이션 정리

  @Column - 컬럼 매핑
  
  @Temporal - 날짜 타입 매핑
  
  @Enumerated - enum 타입 매핑
  
  @Lob - BLOB, CLOB 매핑
  
  @Transient - 특정 필드를 컬럼에 매핑하지 않음


## 5. 연관관계 매핑

   * 다대일 [N:1]
     
     *다대일 양방향 매핑
     
     @중요
      * 외래 키가 있는 쪽이 연관관계 주인
      * 양쪽을 서로 참조하도록 개발
      * 실무에 가장 많이 쓰임
     
     
![image](https://user-images.githubusercontent.com/79154652/132367754-0e2d08d6-43dd-4b76-b103-12f4c256fccf.png)
  
  
   * 일대다 [1:N]
    
     *일대다 양방향
      
      @정리
        * 이런 매핑은 공식적으로 존재 X
        * @JoinColumn(insertable = false, updatable = false)
        * 다대일 양방향을 사용하자
        
        
![image](https://user-images.githubusercontent.com/79154652/132368163-b61aca10-4e2f-4882-ac9d-0e381689a506.png)


  * N:M(다대다) 관계는 1:N, N:1로
    
    * 테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1
    * 실전에서는 중간 테이블이 단순하지 않다.
    * @ManyToMany는 제약 : 필드 추가X, 엔티티 테이블 불일치
    * 실전에서는 @ManyToMany 사용X


 * @JoinColumn
   
   *외래 키를 매핑 할 때 사용
     
     *name - 매핑할 외래 키 이름(기본값은 테이블의 기본 키 컬럼명)
     *referencedColumnName - 외래 키가 참조하는 대상 테이블의 컬럼명(참조하는 테이블의 기본 키 컬럼명)
     *foreignKey(DDL) - 외래 키 제약조건을 직접 지정할 수 있다. 이 속성은 테이블을 생성할 때만 사용한다.
     
     
* @ManyToOne
   
   *다대일 관계 매핑
   
     *optional - false로 설정하면 연관된 엔티티가 항상 있어야 한다.(기본값 TRUE)
     *fetch - 글로벌 페치 전략을 설정한다(@ManyToOne = FetchType.EAGER, @OneToMany = FetchType.LAZY)
     *cascade - 영속성 전이 기능을 사용한다.
     
* @OneToMany

   *다대일 관계 매핑
   
     *mappedBy - 연관관계의 주인 필드를 선택한다
     *fetch - 글로벌 페치 전략을 설정한다.(@ManyToOne = FetchType.EAGER, @OneToMany = FetchType.LAZY)
     *cascade - 영속성 전이 기능을 사용한다.
     



## 6. 상속관계 매핑

    * 관계형 데이터베이스는 상속 관계X
    * 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
    * 상속관계 매핑 : 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑

  ![image](https://user-images.githubusercontent.com/79154652/132370638-96265d82-adab-4a04-9337-1e30c020d2d0.png)

  * 주요 어노테이션
    
    * @inheritance(strategy = InheritanceType.XXX)
      
      * JOINED : 조인 전략
      * SINGLE_TABLE : 단일 테이블 전략
      * TABLE_PER_CLASS : 구현 클래스마다 테이블 전략
    
    * @DiscriminatorColumn(name = "DTYPE")
    
    * @DiscriminatorValue("XXX")

    
    
    * @MappedSuperclass
      
      * 공통 매핑 정보가 필요할 때 사용(id, name)
      
      ![image](https://user-images.githubusercontent.com/79154652/132372117-200e9927-82ed-4c52-a9f2-44a36dc2e271.png)
      
      
      * 상속관계 매핑 X
      * 엔티티X, 테이블과 매핑 X
      * 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
      * 조회, 검색 불가
      * 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
      * @Entity 클래스는 엔티티나 @MappedSuperClass로 지정한 클래스만 상속 가능


## 페치 조인
    
  * fetch join
  
    * SQL 조인 종류 X
    
    * JPQL에서 성능 최적화를 위해 제공하는 기능
    
    * 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능
    
    * join fetch 명령어 사용
    
    * 페치 조인 : = [LEFT[OUTER]| INNER] JOIN FETCH 조인경로

      
       * 엔티티 페치 조인
         * 회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에)
         * SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT
         * [JPQL] select m from Member m join fetch m.team
         * [SQL] select M.*, T.* from Member m inner join team t ON M.team_ID = T.ID


![image](https://user-images.githubusercontent.com/79154652/132541871-a775e1d6-f725-4daf-a881-4971f5bd4ea7.png)


   `
   String jpql = "select m from Member m join fetch m.team";
   
   List<Member> members = em.createQuery(jpql, Member.class).getResultList();
   `
   
 * 컬렉션 페치 조인
   (일대다 관계, 컬렉션 페치 조인)
       
     * [JPQL]
        select t from Team t join fetch t.members where t.name = "팀A"
        
        * 아래와 같이 일대다 관계에서는 데이터가 뻥튀기 되어서 중복 결과를 반환해준다.
        
           ![image](https://user-images.githubusercontent.com/79154652/132701636-ce2c9321-6fb5-4dc5-80ef-e83260c66ab9.png)
           
 * 컬렉션 페치 조인 데이터 뻥튀기 해결 방안
     
       * SQL의 DISTINCT 는 중복된 결과를 제거
       
       * JPQL의 DISTINCT 2가지 기능 제공
       
         1. SQL에 DISTINCT 를 추가
         2. 애플리케이션에서 엔티티 중복 제거

      ****중요 : SQL에 DISTINCT 를 추가해도 데이터가 다르므로 SQL에서 중복 제거 실패

       
![image](https://user-images.githubusercontent.com/79154652/132702281-57d66942-987a-4b3d-bad8-64e7da83136a.png)
          
          * DISTINCT 가 추가로 애플리케이션에서 중복 제거를 시도함.
          
          * 같은 식별자를 가진 Team 엔티티 제거
          
![image](https://user-images.githubusercontent.com/79154652/132702416-f28bcb70-9b23-4fc7-9f93-2ed0113a51d3.png)


## 페치 조인과 일반 조인의 차이
 
   * 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)
   
   * 페치 조이은 객체 그래프를 SQL 한번에 조회하는 개념 


## 페치 조인의 특징과 한계

  * 페치 조인 대상에는 별칭을 못 준다.
  
  * 둘 이상의 컬렉션은 페치 조인 불가능.

  * 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
  
    * 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 기능
    
    * 하이버네이트는 경고 로그를 남기고 메모리에서 페이징 
  
  * 연관된 엔티티들을 SQL 한 번 으로 조회 - 성능 최적화
  
  * 엔티티에 직접 적용하는 글로벌 로딩 전략 @OneToMany(fetch = FetchType.LAZY)보다 우선함
  
  * 실무에서 글로벌 로딩 전략은 모두 지연 로딩
  

## 페치 조인 - 정리

  * 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야하면,
  
    페치조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적
    
## 벌크 연산

   * JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL 실행
      
      * 변경된 데이터가 100건 이라면 100번의 UPDATE SQL실행
   
   * executeUpdate()의 결과는 영향받은 엔티티 수 반환
   
   * UPDATE, DELETE 지원
   
   ~~~java
    String qlString = "update Member m set m.age = 20"
    
    int result = em.createQuery(qlString, Member.class)
                 .executeUpdate();
   ~~~
    
## 벌크 연산 주의

    * 중요
      * 벌크 연산 수행 후 영속성 컨텍스트 초기화
        
        em.clear();


## 기본 키 매핑

 1. `@id` : 현재 필드가 기본키임을 알리기 위한 어노테이션
 2. `@GeneratedValue` : 기본키 값 생성에 관련된 어노테이션

 ~~~java
 @id
 @GeneratedValue(strategy = GenerationType.IDENTITY)
 private Long id;
 ~~~
 
  ### @GeneratedValue
   
   - 기본 키 값을 자동생성하는 어노테이션으로 여러가지 전략이 있다.
     - `IDENTITY` : DB에 위임, 보통 Mysql 사용 시
     - `SEQUENCE` : DB의 시퀀스 오브젝트 사용, 보통 Oracle 사용 시
     - `TABLE` : 키 생성용 테이블을 사용
     - `AUTO` : 방언에 따라 자정 지정(default 전략)
    
 #### IDENTITY
 
   - 주로 MySQL사용시 선택하는 전략
   - DB에게 알아서 증가하도록 위임하는 방법
     (MySQL의 `auto increment` 수행)
   - 주의할 점(매우 중요)
       - JPA는 영속성 컨텍스트를 통해 `transaction.commit` 시점에 SQL이 실행 -> DB가 알아서 기본키 값을 증가시키려면 어디까지 번호가 생성된지 알아야하는데 JPA가 이렇게 동작하면 알수가 없다.
       - 그래서 JPA에서 `IDENTITY` 전략을 사용하면 영속화와 동시에 SQL이 실행되도록 설정되어 있다.
       - `em.persist()` 하면 바로 SQL이 수행되고 DB 식별자 번호를 반환해서 다음 번호로 영속성 컨텍스트에 저장된다.

 #### SEQUENCE
 
   - DB의 시퀀스 오브젝트를 사용한다.
   - DB 시퀀스 란?
     - 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트
   - `Oracle` 등에서 사용됨

   ~~~java
@Entity
/* SequenceGenerator로 Sequence를 생성 */
@SequenceGenerator(
  name = "MEMBER_SEQ_GENERATOR", 
  sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름 
  initialValue = 1,
  allocationSize = 1)
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
                  generator = "MEMBER_SEQ_GENERATOR")
                  // 만든 SequenceGenerator 연결
  private Long id; 
}
   ~~~
   1). `@SequenceGenerator` 선언
       : 제너레이터 이름 / 시퀀스 이름 / 속성 값을 설정
       
   2). `@GeneratedValue(...)` 지정
       :전략(strategy)를 SEQUENCE / generator를 위에서 만든 시퀀스 제너레이터 연결
  
  - 동작 원리
    1. `em.persist()` 로 영속화되어 등록 될 때 `sequence number`가 몇까지 나왔는지 모르기 때문에 `sequence call`을 수행
    2. `sequence call` 로 알아온 `number`를 통해 pk로 등록 후 영속성 컨텍스트에 등록
  
  - 주의할 점
    - 매번 `em.persist()` 할 때 마다 `sequence call`을 하면 비효율적이다. 그래서 제공되는 옵션이 `allocationSize` 이다.
    
    ![image](https://user-images.githubusercontent.com/79154652/154176341-de32d0b5-d34c-44f1-ab5c-dc9b731d5e0a.png)

     - 기본값이 50이 가지는 의미는 50번호 만큼 미리 메모리에 할당 해두는 것
     - 미리 50까지는 메모리에 저장해서 증가시키기 때문에 DB의 `sequence number`는 51이 등록되어 있을 것 임
     - 50번의 번호가 가득 찼을 때 다음 `sequence number`를 알아내는 `sequence call`이 수행되어 나름 효율적인 구조를 가지게 된다.


  #### Table
    
   - 키를 생성하고 관리하는 테이블을 하나 만들어서 DB 시퀀스를 흉내
   - 모든 DB에 대해 적용 가능
   - 성능이 좋지 못해 실무에서 잘 사용되지는 않는다.
   - 시퀀스처럼 동작하기 때문에 역시 `영속성 컨텍스트` 문제를 겪게 되고 시퀀스와 동일하게 `allocationSize` 옵션으로 동작한다.
   
   ~~~java
@Entity
@SequenceGenerator(
  name = "MEMBER_SEQ_GENERATOR", 
  table = "MY_SEQUENCES", // 데이터베이스 이름 
  pkColumnValue = "MEMBER_SEQ",
  allocationSize = 1)
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.TABLE,
                  generator = "MEMBER_SEQ_GENERATOR")
  private Long id; 

   ~~~
   
   #### AUTO
   
   - Default로 설정되는 값
   - 방언에 따라 위의 3가지 전략을 자동으로 지정
     (Mysql 인지/ Oracle 인지)
  
  
  #### 권장하는 식별자 전략
   
   - 기본키 제약조건을 만족
     - not null
     - 유일성과 최소성
     - 불변성
   
   - 권장하는 식별자 전략
     - `(Long형) + (대체키) + (적절한 키생성 전략)` 이용
       - `Long`형을 사용해야 하는 이유
         - `int`는 0을 사용해서 null 과 0의 구분이 애매해서 오류가 날 수 있음
         - `Integer`는 10억이 넘어가면 순환해서 오류가 날 수 있음
         - 그래서 Long 이 위 두 문제를 어느정도 해결
     - 대체키 사용
       - 랜덤값 / UUID 등 비지니스와 관계없는 값
     - 적절한 키 생성 전략
       - `IDENTITY` / `SEQUENCE` 추천
