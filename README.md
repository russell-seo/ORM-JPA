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
   
 * 컬렉션 페치 조인
   (일대다 관계, 컬렉션 페치 조인)
       
