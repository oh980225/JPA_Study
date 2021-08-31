# 6장 다양한 연관관계 맵핑

## **-5장에 대한 복습-**

    다양한 연관관계 맵핑에 대해서 들어가기 앞서 연관관계의 기초인 5장에 내용을 정리하고 넘어가야 한다.

- 다중성
- 단방향, 양방향
- 연관관계의 주인

### **1.다중성**
    다중성을 판단하기 어려울 때는 반대의 경우를 생각해보면 편하다.
    (일대다의 반대는 다대일과 같은 경우, 일대일의 반대는 일대일이다.)
    But, 다대다 관계는 실무에서 거의 사용하지 않는다.

### **2.단방향, 양방향**
- 테이블의 경우  
    - 외래 키 하나로 조인을 사용해 양방향으로 퀄가 가능하므로 사실상 방향성이라는 개념이 없음  

- 객체의 경우  
    - 참조용 필드를 가지고 있는 객체만 연관된 객체를 조회할 수 있다. 그렇기에 한 쪽만 참조하는 것을 단방향 관계라 하고 양쪽이 서로 참조하는 것을 양방향 관계라 한다.

### **3.연관관계의 주인**
    데이터 베이스는 외래 키 하나로 두 테이블이 연관관계를 맺기에 연관관계를 관리하는 포인트는 1개이다.
    반면에 엔티티를 양방향으로 맵핑하면 (A -> B, B -> A)2곳에서 서로를 참조하기에 관리 포인트는 2개가 된다.!!

- JPA의 경우  
    
    - 두 객체중 하나의 객체를 정해서 데이터 베이스를 관리하는 연관관계의 주인으로 만든다. 보통 외래키를 가진 테이블과 매핑한 엔티티가 외래키를 관리하는게 효육적이므로 연관관계의 주인으로 만든다.


|연관관계의 주인|연관관계의 주인이 아님|
|:---:|:---:|
|mappedby 속성을 사용하지 않음|mappedby속성을 사용함(주인 필드 이름을 값으로 입력해야함)|
|외래 키를 수정 및 읽기 가능|외래 키를 수정할 수 없고 읽기만 가능|

## **-다양한 연관관계-**

    다중성과 단뱡향,양방향을 고려한 모든 연관관계이다.(참고로 왼쪽을 연관관계의 주인으로 정했다.)
|---|연관관계의 주인|방향|
|:---:|:---:|:---:|
|다대일|다|단방향, 양방향|
|일대다|일|단방향, 양방향|
|일대일|일|주 테이블 단방향, 양방향|
|일대일|일|대상 테이블 단방향, 양방향|
|다대다|다|단방향, 양방향|


## **1.다대일**
- JPA에서 가장 많이 사용하고 꼭 알아야 되는 다중성이다.  
- 데이터베이스 테이블의 일(1), 다(N) 관계에서는 외래 키는 항상 다쪽에 있다.
### **다대일 단방향[N:1]**
--------

``` java
//회원 엔티티
@Entity
@NoArgsConstructor
@ToString(exclude = "team")
@Getter @Setter
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관

    //getter, setter..
}
```
``` java
//팀 엔티티
@Entity
@NoArgsConstructor
@Getter @Setter
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    //getter, setter ..
}
```
![image](https://user-images.githubusercontent.com/68093714/131366634-d7656aad-fca6-410e-98d3-110d4961130a.png)

회원은 Member.team으로 팀 엔티티를 참조할 수 있다. 반대로 팀에는 회원을 참조하는 필드가 없기 때문에 다대일 단방향 연관관계이다. 특히
``` java
@ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관
```
@JoinColumn(name = "TEAM_ID")를 사용해서 Member.team필드를 TEAM_ID 외래 이와 매핑했기 때문에 Member.team 필드로 회원 테이블의 TEAM_ID를 관리한다.
<br/><br/><br/>

### **다대일 양방향[N:1, 1:N]**
--------
``` java
//회원 엔티티
@Entity
@NoArgsConstructor
@ToString(exclude = "team")
@Getter @Setter
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관

    //회원의 편의 메소드
    public void setTeam(Team team){
        
        this.team = team;
        //무한루프에 빠지지 않도록 체크합니다.
        if(!team.getMembers().contains(this)){
            team.getMembers().add(this);
        }
    }
    //getter, setter..
}
```
``` java
//팀 엔티티
@Entity
@NoArgsConstructor
@Getter @Setter
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    //팀의 편의 메소드
    public addMember(Member member){
        this.members.add(member);
        if(member.getTeam() != this){ // 무한루프에 빠지지 않도록 체크함
            member.setTeam(this);
        }
    }
}
```
![image](https://user-images.githubusercontent.com/68093714/131366580-a24c1195-d794-4c13-8212-7b3972025da4.png)

- 양방향은 외래 키가 있는 쪽이 연관관계의 주인이다.

    - JPA는 외래 키를 관리할 때 연관관계의 주인만 사용한다. 주인이 아닌 Team.members는 조회를 위한 JPQL이나 객체 그래프를 탐색할 때 사용한다.

- 양방향 연관관계는 항상 서로를 참조해야 한다.

    - 서로를 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋은데 현재 setTeam(), addMember() 메소드가 이런 메소드들이다. 편의 메소드는 한곳에만 작성하거나 양쪽 다 작성할 수 있는데, 양쪽에 다 작성하면 무한 루프에 빠지므로 주의해야 한다. 예제 코드는 편의 메소들를 양쪽에 다 작성해서 둘 중 하나만 호출하면 된다.
## **2.일대다**
- 일 쪽이 연관관계의 주인이 되는 경우이다.
- 표준스펙에서 지원은 하지만 실무에서 이 모델을 권장하지 않는다.
### **일대다 단방향[1:N]**
-----
``` java
//팀 엔티티
@Entity
@Getter @Setter
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)
    private List<Member> members = new ArrayList<>();
    //getter, setter..
}
```
``` java
//회원 엔티티
@Entity
@Setter
@Getter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131366919-405fddfa-c50c-4dbd-b5a3-6dd710aeadfb.png)

    일대다 단방향 관계에서는 특이한 부분이 있는데, 테이블에서는 Member에 TEAM_ID라는 외래 키를 관리하지만
    객체에서는 Team에 members로 참조 필드가 있다는 것이 특징이다.
일대다 단방향 관계를 매핑할 때는 @JoinColumn을 명시해야한다. 그렇지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑한다. 조인 테이블은 7.4절에서 만나요~
- 일대다 단방향 매핑의 단점  
    - 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점이다! 왜냐하면 본인 테이블에 있다면 한번의 SQL문(INSERT)으로 끝낼 수 있지만, 다른 테이블에 있다면 추가적인 SQL문(UPDATE)을 실행해줘야한다.
- 일대다 단방향 매핑 보다는 다대일 양방향 매핑을 사용하자
    - 위에서 나온 단점처럼 다른 테이블에 있는 외래 키를 관리하는 것과 같이 성능, 관리면에서 부담스러운 문제이다. 상황에 따라 다르지만 다대일 양방향 매핑을 권장한다.

<br/>

                    **일대다 단방향 매핑의 단점이 생기는 이유**
        객체와 테이블 연관관계에서 비는 부분을 SQL문으로 채워야 한다는 것이 핵심!!
<br/><br/><br/>

### **일대다 양방향[1:N, N:1]**
-----
``` java
//팀 엔티티
@Entity
@Getter @Setter
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)
    private List<Member> members = new ArrayList<>();
    //getter, setter..
}
```
``` java
//회원 엔티티
@Entity
@Setter
@Getter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
    
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131370705-03344507-53fb-49f7-b5de-ddde16e1d6ea.png)

    일대다 양방향 맵핑은 존재하지 않는다.(대부분의 코딩처럼 가능은 하다 like 위에 있는 코드)
    왜냐하면 연관관계의 주인은 항상 '다' 에게 있으므로 @ManyToOne에는 mappedBy 속성이 없기 때문이다.

- 위 코드 처럼 회원 엔티티에 TEAM_ID를 매핑했다. 이 방법의 단점으로는 다음과 같다.
    1. 같은 외래 키를 관리하므로 문제가 발생한다.
    2. @JoinColumn에 설정으로 읽기만 가능하게 만든 형태이기 때문에 단방향 매핑이 갖는 단점을 그대로 유지한다.
    3. 딴생각말고 다대일 양방향 매핑을 사용하자

## **3.일대일**
- 테이블 관계에서 일대다, 다대일은 항상 다(N)쪽에 외래 키를 가진다. 일대일의 경우는? 주 테이블, 대상 테이블 둘 중 어느 곳이나 외래 키를 가질 수 있다.
- 주 테이블에 외래 키
    - 주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 참조한다. 따라서 주 테이블만 확인해도 대상 테이블과의 연관관계가 있는지 알 수 있다.
- 대상 테이블에 외래 키
    - 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.
<br/>

### **주 테이블에 외래키(단방향)**
-----
``` java
//멤버 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;   
    //getter, setter..
}
```
``` java
//락커 엔티티
@Entity
@Getter @Setter
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131424993-3aa261cc-1083-4691-ae5e-880460c0eeb1.png)

@OneToOne을 사용했고 외래 키에 유니크 제약 조건(UNI)이 추가되어있고 다대일 단방향과 거의 비슷하다.
<br/><br/><br/>

### **주 테이블에 외래키(양방향)**
-----

``` java
//멤버 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;  
    //getter, setter..
}
```
``` java
//락커 엔티티
@Entity
@Getter @Setter
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker") // 연관관계의 주인이 아님을 선언
    private Member member;
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131425442-fce1dc87-d8ff-4ca4-baab-ffa836083e89.png)

양방향인 경우에는 당연히 연관관계의 주인을 정해야 한다. Member 테이블이 외래 키를 가지고 있으므로 Member.locker가 연관관계의 주인이다. Locker의 member에는 mapperBy를 선언해서 연관관계의 주인이 아님을 설정했다.
<br/><br/>
<br/>

### **대상 테이블에 외래키(단방향)**
-----
![image](https://user-images.githubusercontent.com/68093714/131426768-34c12cd6-66f7-4f3b-8134-517dbfa56d4f.png)

일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계는 JPA에서 지원하지 않고 매핑할 수 있는 방법도 없다. 따라서 관계의 방향을 변경하거나 양방향 관계에서 Locker를 연관관계의 주인으로 설정해야 한다.
<br/><br/><br/>

### **대상 테이블에 외래키(양방향)**
-----

``` java
//멤버 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @OneToOne(mappedBy = "member")
    private Locker locker;
    //getter, setter..
}
```
``` java
//락커 엔티티
@Entity
@Getter @Setter
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131426739-02c27aba-45b3-4225-a6cd-c69c8628d2f5.png)

코드가 달라진 부분은 mappedBy가 멤버 엔티티에 있다는 점, 연관관계의 주인이 대상 테이블이기 떄문이다.

    프록시를 사용할 때 외래 키를 직접 관리하지 않는 일대일 관계는 지연 로딩을 설정해도 즉시 로딩되는 경우가 있다. 8장에서 다시 만나요~!
참고 URL(https://developer.jboss.org/wiki/SomeExplanationsOnLazyLoadingone-to-one)
<br/><br/><br/>

## **4.다대다**
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- 보통 일대다, 다대일 관계로 풀어내는 연결테이블을 이용한다.
- 객체는 컬렉션을 사용해서 다대다의 관계를 나타낼 수 있다.
----
    데이터 베이스에서 다대다를 나타낸 것과 객체의 컬렉션
![image](https://user-images.githubusercontent.com/68093714/131428254-ebbcc7f5-d5a7-4c4f-9985-7cbdc62b8e4d.png)
![image](https://user-images.githubusercontent.com/68093714/131428261-67ab922a-d9db-4a3e-a695-1f277c7bf44f.png)
![image](https://user-images.githubusercontent.com/68093714/131428268-9fe033a9-3589-4d0e-a66d-babadf6c91f0.png)
<br/><br/><br/>

### **다대다 : 단방향**
-----
``` java
//회원 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
                joinColumns = @JoinColumn(name = "MEMBER_ID"),
                inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    List<Product> products = new ArrayList<Product>();
}
```
``` java
//상품 엔티티
@Entity
@Getter
@Setter
public class Product {

    @Id
    @JoinColumn(name = "PRODUCT_ID")
    private String id;

    private String name;
}
```

    중요한 점은 @ManyToMany와 @JoinTable을 사용해서 연결 테이블을 바로 매핑한 것이다.
    따라서, 회원_상품(MEMBER_PRODUCT)엔티티 없이도 매핑을 할 수 있다.

- @JoinTable의 속성
    - name == 연결 테이블을 지정한다.(MEMBER_PRODUCT)
    - joinColumns == 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.(MEMBER_ID)
    - inverseJoinColumns == 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.(PRODUCT_ID)

- @ManyToMany로 매핑한 덕분에 연결테이블을 신경쓰지 않고 다대다 관계를 사용할 수 있다.
- 저장, 조회 등이 @ManyToMany를 통해서 쉽고 간단하게 SQL문이 실행된다.
<br/><br/><br/>


### **다대다 : 양방향**
-----
``` java
@Entity
public class Product{

    @Id
    private String id;

    @ManyToMany(mappedBy = "products") // 역방향 추가
    private List<Member> members;

}
```
    역방향도 똑같이 @ManyToMany를 사용하고 양쪽 중 원하는 곳에 mappedBY로 연관관계를 만들어주면 된다.
    (물론 mappedBy가 없는 곳이 주인)

- 다대다의 양방향 연관관계 설정  
    member.getProducts().add(product);  
    product.getMembers().add(member);
- 메소드를 만들어서 관리하면 편리하다.  
``` java
public void addProduct(Product product){
    products.add(product);
    product.getMembers().add(this);
}
```
- 역방향 객체 그래프 탐색
``` java
public void findInverse() {
    Product product = em.find(Product.class, "productA");
    List<Member> members = product.getMembers();
    for (Member member : members){
        System.out.println("member = " + member.getUsername());
    }
}
```
<br/><br/><br/>

### **다대다 : 매핑의 한계와 극복, 연결 엔티티 사용**
-----
- @ManyToMany를 사용하면 연결 테이블을 자동으로 처리해주므로 도메인 모델이 단순해지고 여러 가지로 편하나 실무에서의 사용은 한계가 존재한다.
- 예를 들어, 상품 구매와 같은 상황에서 회원 이름, 상품 ID만을 기록하는 것이 아니다. 주문 수량, 주문 날짜와 같은 추가적인 내용을 기록해야 하는데 컬럼을 추가하게 되면 매핑할 수 없는 컬럼이 생기기에 @ManyToMany를 사용할 수 없다.
- 엔티티 간의 관계도 테이블 관계처럼 다대다에서 일대다, 다대일 관계로 풀어야 한다.

![image](https://user-images.githubusercontent.com/68093714/131432848-90a4ad14-8884-4647-8cd7-5ed5cdb63710.png)
![image](https://user-images.githubusercontent.com/68093714/131432853-61bb5285-2ef6-4d2d-acfa-812c59d219ea.png)
``` java
//회원 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts;
}
```
``` java
//상품 엔티티
@Entity
@Getter
@Setter
public class Product {

    @Id
    @JoinColumn(name = "PRODUCT_ID")
    private String id;

    private String name;
}
```
``` java
//회원상품 엔티티
@Entity
@IdClass(MemberProductId.class)
@public class MemberProduct{
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member; // MemberProductId.member와 연결

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product; // MemberProductId.product와 연결

    private int orderAmount;
}
```
``` java
//회원상품 식별자 클래스
public class MemberProductId implements Serializable{
    private String member; // MemberProduct.member와 연결
    private String product; // MemberProduct.product와 연결

    //hashCode and equals

    @override
    public boolean equals(Object o){...}

    @override
    public int hashCode(){...}
}
```

- 회원과 회원상품을 양방향 관계로 만들어 줬고 회원상품 쪽이 외래 키를 가지고 있으므로 연관관계의 주인이다.
- 상품 엔티티에서 회원상품 엔티티로의 객체 탐색 기능이 필요하지 않다고 판단되어 연관관계를 만들지 않았다.
- 회원상품 엔티티에서는 @Id, @JoinColumn을 동시에 사용해 기본 키, 외래 키를 한번에 처리하였다.
- @IdClass를 사용해서 복합 기본키 매핑을 했다.
- 복합 기본키
    - 복합 키는 별도의 식별자 클래스로 만들어야 한다.
    - Serializable, equals, hashCode 메소드를 구현해야한다.
    - 기본생성자가 있어야하며 식별자 클래스는 public이어야 한다.
    - @IdClass 외에도 @EmbeddedId를 사용하는 방법도 있다.
<br/><br/><br/>

### **다대다 : 새로운 기본 키 사용**
-----
![image](https://user-images.githubusercontent.com/68093714/131432855-deec71fe-0632-4154-9575-0e62ef07d364.png)

    교재에서 추천하는 기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것이다. 
    간편하고 거의 영구히 쓸 수 있으며 비즈니스에 의존하지 않는 장점이 있다.

MEMBERPRODUCT 대신 ORDER라는 연결 테이블에 새로운 기본 키를 사용하는 것이다.

``` java
//주문 엔티티
@Entity
public class Order {
    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long Id;

    @ManyToOne
    @JoinColumn(name= "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
    
    private int orderAmount;
    ...
}
```
``` java
//회원 엔티티
@Entity
@Getter
@Setter
public class Member {

    @Id @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts;
}
```
``` java
//상품 엔티티
@Entity
@Getter
@Setter
public class Product {

    @Id
    @JoinColumn(name = "PRODUCT_ID")
    private String id;

    private String name;
}
```
- 복합 키를 사용하는 것보다 매핑이 단순하고 이해하기가 쉽다.
- 저장, 조회도 식별자 클래스를 사용하지 않아서 코드가 한결 단순해진다.

<br/><br/><br/>

### **다대다 연관관계 정리**
-----
- 다대대 관계를 일대다, 다대일 관계로 풀어내기 위해 연결 테이블을 만들 때 식별자를 어떻게 구성할지 선택해야한다

|---|식별 관계|비식별 관계|
|:---:|:---:|:---:|
|개념|박아온 식별자를 기본 키 + 외래 키로 사용한다.|받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가한다.|
|데이터 베이스|부모의 테이블의 기본 키를 받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 것 | 단순히 외래 키로만 사용하는 것|
|객체|복합 키를 위한 식별자 클래스를 생성|복합 키를 위한 식별자 클래스를 안만들어도 되기에 단순하고 ORM 매핑을 편리하게 할 수 있다. 

