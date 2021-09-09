# ORM-JPA
## 1.Entity 매핑
속성 : name
  * JPA에서 사용할 엔티티 이름 지정
  * 기본값 : 클래스 이름을 그대로 사용
  * 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.

## 2.Table 매핑
  @Table은 엔티티와 매핑할 테이블 지정
  
  * name - 매핑할 테이블 이름
  
  * catalog - 데이터베이스 catalog 매핑
  
  * schema - 데이터베이스 schema 매핑
  
  * uniqueConstraints - DDL 생성 시에 유니크 제약조건 생성
  
## 3. 데이터베이스 스키마 자동생성 - 주의

* 운영 장비에는 절대 create, create-drop, update 사용 X

* 테스트 서버는 update 또는 validate

* 스테이징과 운영 서버는 validate 또는 none

## 4.DDL 생성 기능

* 제약조건 추가 : 회원 이름은 필수, 10자 초과 X

  @Column(nullable = false, length = 10)
  
* 유니크 제약조건 추가
  
  @Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE} )})
  
  * DDL 생성 기능은 DDL을 자동 생성할 때 만 사용
  JPA의 실행 로직에는 영향을 주지 않음
  
## 5. 매핑 어노테이션 정리

  @Column - 컬럼 매핑
  
  @Temporal - 날짜 타입 매핑
  
  @Enumerated - enum 타입 매핑
  
  @Lob - BLOB, CLOB 매핑
  
  @Transient - 특정 필드를 컬럼에 매핑하지 않음


## 6. 연관관계 매핑

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
     



## 7. 상속관계 매핑

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


## 10. 페치 조인(fetch join)
    
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
   
 ## 컬렉션 페치 조인
   (일대다 관계, 컬렉션 페치 조인)
       
     * [JPQL]
        select t from Team t join fetch t.members where t.name = "팀A"
        
        * 아래와 같이 일대다 관계에서는 데이터가 뻥튀기 되어서 중복 결과를 반환해준다.
        
           ![image](https://user-images.githubusercontent.com/79154652/132701636-ce2c9321-6fb5-4dc5-80ef-e83260c66ab9.png)
           
 ## 컬렉션 페치 조인 데이터 뻥튀기 해결 방안
     
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
   
    `
    String qlString = "update Member m set m.age = 20"
    
    int result = em.createQuery(qlString, Member.class)
                 .executeUpdate();
    `
    
## 벌크 연산 주의

    * 중요
      * 벌크 연산 수행 후 영속성 컨텍스트 초기화
        
        em.clear();
