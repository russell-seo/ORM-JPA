# JPA 활용 2편

 ## Entity를 DTO로 변환
 ~~~java
 @GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<SimpleOrderDto> result = orders.stream()
               .map(o -> new SimpleOrderDto(o))
               .collect(toList());
 return result;
}

@Data
static class SimpleOrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 
 public SimpleOrderDto(Order order) {
 orderId = order.getId();
 name = order.getMember().getName();
 orderDate = order.getOrderDate();
 orderStatus = order.getStatus();
 address = order.getDelivery().getAddress();
 }
}
 ~~~

- Entity를 DTO로 변환하는 일반적인 방법
- 쿼리가 총 1 + N + N 번 실행된다
  - `order`조회 1번(order 조회 결과 수가 N이 된다)
  - `order -> member` 지연 로딩 조회 N 번
  - `order -> delivery` 지연 로딩 조회 N 번
  - order 결과가 2개면 최악의 경우 1 + 2 + 2 번 실행된다
    - 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 날리지 않는다.

## Entity를 DTO로 변환 - 페치 조인 최적화

Controller 코드는 위 와 동일

OrderRepository - 추가 코드
~~~java
public List<Order> findAllWithMemberDelivery() {
 return em.createQuery(
               "select o from Order o" +
               " join fetch o.member m" +
               " join fetch o.delivery d", Order.class)
 .getResultList();
}
~~~
- Entity 페치 조인 을 사용하여 쿼리 1번에 조회
- 페치 조인으로 `order -> member`, `order -> delivery` 는 이미 조회 된 상태이므로 지연로딩 X

## JPA에서 DTO로 바로 조회

OrderQueryRepository - DTO로 조회하는 전용 리포지토리를 생성하는걸 추천
~~~java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
  private final EntityManager em;
 
 public List<OrderSimpleQueryDto> findOrderDtos() {
    return em.createQuery(
          "select new 
                  jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, 
                  o.orderDate, o.status, d.address)" +
                  " from Order o" +
                  " join o.member m" +
                  " join o.delivery d", OrderSimpleQueryDto.class)
                  .getResultList();
 }
}
~~~

- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회
- `new` 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
- SELECT 절에서 원하는 데이터를 직접 선택하므로 DB -> 애플리케이션 네트워크 용량 최적화(생각보다 미비)
- Repository 재사용성 떨어짐, API 스펙에 맞춘 코드가 Repository에 들어가는 단점

`정리`

`쿼리 방식 선택 권장 순서`

1. `우선 Entity를 DTO로 변환하는 방법을 선택한다`
2. `필요하면 Fetch Join으로 성능을 최적화 한다. -> 대부분의 성능 이슈가 해결`
3. `그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.`
4. `최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template를 사용해서 SQL을 직접 사용`




# 컬렉션(Collection) 조회 최적화

 ## Entity를 DTO로 변환
 
 Controller
 ~~~java
 @GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
 List<Order> orders = orderRepository.findAll();
 List<OrderDto> result = orders.stream()
 .map(o -> new OrderDto(o))
 .collect(toList());
 return result;
}
 ~~~



 DTO
 
 - Order -> OrderDTO로 매핑해야하며, Order 에 있는 orderItems 도 OrderItemDTO로 받아야 한다.
 - 많은 주니어 개발자들이 조회하는 `Entity`는 DTO로 변환하며 Presentation 쪽으로 반환하지만, `Entity안에 있는 Entity는 DTO로 변환하지 않는데 이 또한 DTO로 변환하며 반환해 주어야 한다.`
~~~java
@Data
static class OrderDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemDto> orderItems;
 
         public OrderDto(Order order) {
             orderId = order.getId();
             name = order.getMember().getName();
             orderDate = order.getOrderDate();
             orderStatus = order.getStatus();
             address = order.getDelivery().getAddress();
             orderItems = order.getOrderItems().stream()
                                               .map(orderItem -> new OrderItemDto(orderItem))
                                               .collect(toList());
         }
}

@Data
static class OrderItemDto {
  private String itemName;//상품 명
  private int orderPrice; //주문 가격
  private int count; //주문 수량
  
  public OrderItemDto(OrderItem orderItem) {
      itemName = orderItem.getItem().getName();
      orderPrice = orderItem.getOrderPrice();
      count = orderItem.getCount();
  }
}
~~~

 위 코드의 문제점은 
 
  - 지연 로딩으로 너무 많은 SQL이 실행된다.
  - SQL 실행 수
      - `order` 1번
      - `member`,`address` N번(order 조회 수 만큼)
      - `orderItem` N번(order 조회 수 만큼)
      - `item` N번 (orderItem 조회 수 만큼)


## Entity -> DTO로 변환 (페치 조인)

Repository
~~~java
public List<Order> findAllWithItem() {
 return em.createQuery(
              "select distinct o from Order o" +
              " join fetch o.member m" +
              " join fetch o.delivery d" +
              " join fetch o.orderItems oi" +
              " join fetch oi.item i", Order.class)
       .getResultList();
}

~~~

 - 페치 조인으로 SQL이 1번만 실행됨
 - `1 : N 에서 fetch join 할 시 DB ROW 갯수가 N개 만큼 증가하게 됨` 
   
주의 :  예) Order : orderItems 는 1 : N 관계이다. 만약 Order 가 2건이며, Order 1건 당 2개의 orderItems 를 가진다면. 
   `즉 Order 기준으로 ordetItems 와 fetch join 할 시 2개의 row가 아닌 총 4개의 row가 발생한다.`
 
 - `distinct`를 사용한 이유는 1 : N join이 있으므로 DB ROW 수가 증가한다. 그 결과 Order Entity 수 도 증가한다.
    
    하지만 JPA의 `distinct`는 두 가지 기능이 있다.
    
    1. DB에 distinct를 추가해서 쿼리를 날려준다. 하지만 DB의 `distinct`는 모든 column의 값이 같은 것만 중복을 제거 해준다. 즉 같지 않다면 똑같이 N 개의 ROW가 발생
    2. DB에서 1번과 같은 경우가 발생하지만 JPA에서 `distinct`는 애플리케이션 내부적으로 id 값이 같은 column에 대해 제거를 하고 가져와 준다 


 - 하지만 1 : N 페치 조인의 단점은 `페이징이 불가능` 하다.

    참고 : 컬렉션 페치 조인을 사용하면 `페이징이 불가능` 하다. Hibernate는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에
    페이징 해버린다(매우 위험하다. Out of Memory 가 발생할 수 있다)
    
    참고 : 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 1개를 페치 조인 할 때도 데이터가 뻥튀기
    되기 때문에 2개 이상이면 데이터가 부정합하게 조회 될 수 있다.
    

## Entity를 DTO로 변환 - 페이징과 한계 돌파

- 컬렉션을 페치 조인하면 페이징이 불가능하다.
   - 컬렉션을 페치조인 하면 1:N 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.
   - 1:N에서 일(1)을 기준으로 페이징하는 것이 목적, 그러나 데이터는 다(N)을 기준으로 row가 생성된다.
   - Order를 기준으로 페이징하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 된다.
- 이 경우 하이버네이트는 경고 로그를 남기고  모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.

### 해결책

- 먼저 ToOne(OneToOne, ManyToOne)관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`,`@BatchSize`를 적용한다.
   - hibernate.default_batch_fetch_size : 글로벌 설정
   - @BatchSize : 개별 최적화
   - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회한다.

Repository
~~~java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
   return em.createQuery(
              "select o from Order o" +
              " join fetch o.member m" +
              " join fetch o.delivery d", Order.class)
       .setFirstResult(offset)
       .setMaxResults(limit)
       .getResultList();
}
~~~

Controller
~~~java
/**
 * V3.1 엔티티를 조회해서 DTO로 변환 페이징 고려
 * - ToOne 관계만 우선 모두 페치 조인으로 최적화
 * - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
 */
@GetMapping("/api/v3.1/orders")
public List<OrderDto> ordersV3_page(
 @RequestParam(value = "offset", defaultValue = "0") int offset,
 @RequestParam(value = "limit", defaultValue = "100") int limit){
 
   List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
   
   List<OrderDto> result = orders.stream().map(o -> new OrderDto(o)).collect(toList());
   
   return result;
 }
~~~

application.yml(글로벌 설정)

![image](https://user-images.githubusercontent.com/79154652/147529436-94debe19-e4db-4cf7-a76b-56aaef7a3efd.png)

- 개별로 설정하려면 `@BatchSize`를 적용하면 된다.(컬렉션은 컬렉션 필드에, Entity는 Entity 클래스에 적용)


- __장점__
  - 쿼리 호출수가 `1 + N` -> `1 + 1`로 최적화 된다.
  - 조인보다 DB 데이터 전송량이 최적화 된다.(Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다)
  - 페치 조인 방식과 비교해서 쿼리 호출수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
  - 컬렉션 페치조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.
- 결론

  - ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리수를 줄이고 해결하며, 나머지는 `hibernate.default_batch_fetch_size`로 최적화 하자.

>참고 : `default_batch_fetch_size`의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다. 이 전략을 SQL IN 절을 사용하는데, DB에 따라 IN 절 파라미터를 1000으로
        제한하기도 한다. 
     
 ## JPA에서 DTO 직접 조회
     
  `Repository`
  
  ~~~java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {
   private final EntityManager em;
 /**
 * 컬렉션은 별도로 조회
 * Query: 루트 1번, 컬렉션 N 번
 * 단건 조회에서 많이 사용하는 방식
 */
  
 public List<OrderQueryDto> findOrderQueryDtos() {
  
  //루트 조회(toOne 코드를 모두 한번에 조회)
  List<OrderQueryDto> result = findOrders();
 
 //루프를 돌면서 컬렉션 추가(추가 쿼리 실행)
 result.forEach(o -> {
 List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
         o.setOrderItems(orderItems);
 });
       return result;
 }
 
 /**
 * 1:N 관계(컬렉션)를 제외한 나머지를 한번에 조회
 */
 
 private List<OrderQueryDto> findOrders() {
     return em.createQuery(
          "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
          " from Order o" +
          " join o.member m" +
          " join o.delivery d", OrderQueryDto.class)
            .getResultList();
 }
 
 /**
 * 1:N 관계인 orderItems 조회
 */
 
 private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
        " from OrderItem oi" +
        " join oi.item i" +
        " where oi.order.id = : orderId", OrderItemQueryDto.class)
              .setParameter("orderId", orderId)
              .getResultList();
  }
}

  ~~~
  
  `OrderQueryDto`
  
  ~~~java
  @Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
 private Long orderId;
 private String name;
 private LocalDateTime orderDate; //주문시간
 private OrderStatus orderStatus;
 private Address address;
 private List<OrderItemQueryDto> orderItems;
 
 public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate,
OrderStatus orderStatus, Address address) {
 this.orderId = orderId;
 this.name = name;
 this.orderDate = orderDate;
 this.orderStatus = orderStatus;
 this.address = address;
 }
}
  
  ~~~
  
  `OrderItemQueryDto`
  
  ~~~java
  
@Data
public class OrderItemQueryDto {
 
 @JsonIgnore
 private Long orderId; //주문번호
 private String itemName;//상품 명
 private int orderPrice; //주문 가격
 private int count; //주문 수량
 
 public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
    this.orderId = orderId;
    this.itemName = itemName;
    this.orderPrice = orderPrice;
    this.count = count;
 }
}
  
  ~~~
  
  - Query : 루트 1번, 컬렉션 N번
  - ToOne(N:1, 1:1)관계들을 먼저 조회하고, ToMany(1:N)관계는 각각 별도로 처리한다.
     - 이런 방식을 선택한 이유
        - ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.
        - ToMany(1:N) 관계는 조인하면 row 수가 증가한다.
  - row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화하기 쉬우므로 한번에 조회하고, ToMany 관계는 최적화 하기 어려우므로 `findOrderItems()`같은 별도의 메소드로 조회
  

# JPA 기본 정리
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


## 8. 페치 조인(fetch join)
    
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
