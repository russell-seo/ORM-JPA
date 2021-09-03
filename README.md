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

  
