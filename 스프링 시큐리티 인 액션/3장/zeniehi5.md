# 3장. 사용자 관리

---

📌 UserDetailsService를 제대로 이해하자

- UserDetails - 사용자를 기술
- GrantedAuthority - 사용자가 실행할 수 있는 작업을 정의
- UserDetailsManager - UserDetailsService 계약 확장
    - 사용자 만들기, 사용자의 암호 수정이나 삭제 등의 작업 지원

### 3.1 스프링 시큐리티의 인증 구현

![IMG_20A7308037E3-1.jpeg](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205f73af18afc343cf917ad3a821013168/IMG_20A7308037E3-1.jpeg)

- 사용자 관리를 위해서 UserDetailsService 및 UserDetailsManager 인터페이스 이용
    - UserDetailsService - 사용자 이름으로 사용자를 검색하는 역할
    - UserDetailsManager - 사용자 추가, 수정, 삭제 작업을 추가
- 두 계약 간 인터페이스 분리를 통해 애플리케이션의 유연성 향상
- UserDetails
    - 개발자는 프레임워크가 이해할 수 있게 사용자를 기술
        - 사용자는 사용자가 수행할 수 있는 작업의 집합을 가짐 → “권한”
        - GrantedAuthority 인터페이스

![IMG_6F05187D96F4-1.jpeg](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205f73af18afc343cf917ad3a821013168/IMG_6F05187D96F4-1.jpeg)

### 3.2 사용자 기술하기

📌 스프링 시큐리티가 사용자를 이해할 수 있도록 애플리케이션의 사용자를 기술하는 방법

- 사용자 정의는 UserDetails 계약을 준수

**3.2.1 UserDetails 계약의 정의 이해하기**

- getUsername() 및 getPassword()
    - 각각 사용자 이름과 암호를 반환
    - 반환된 사용자 이름과 암호는 앱에서 인증 과정에 사용
- 나머지
    - 사용자가 애플리케이션의 리소스에 접근할 수 있도록 권한을 부여하기 위한 것
        - 계정 만료
        - 계정 잠금
        - 자격 증명 만료
        - 계정 비활성화
    - getAuthorities()
        - 사용자에게 부여된 권한의 그룹을 반환

**3.2.2 GrantedAuthority 계약 살펴보기**

- 권한(Authority)
    - 사용자가 애플리케이션에서 수행할 수 있는 작업
    - 권한을 만들려면 해당 이용 권리의 이름만 찾으면 가능
        - 추후 권한 부여 규칙을 작성할 때 참조
- GrantedAuthority 구현 방법

```java
GrantedAuthority g1 = () -> "READ";
GrantedAuthority g2 = new SimpleGrantedAuthority("READ");
/** 
	SimpleGrantedAuthority 클래스 -> GrantedAuthority 형식의 변경이 불가능한 인스턴스
*/
```

**3.2.3 최소한의 UserDetails 구현 작성**

🏁 작성 완료!

**3.2.4 빌더를 이용해 UserDetails 형식의 인스턴스 만들기**

```java
UserDetails u = User.withUsername("bill")
									.password("12345")
									.authorities("read", "write")
									.accountExpired(false)
									.disabled(true)
									.build();
```

- User 클래스
    - UserDetails 형식의 인스턴스를 간단하게 만드는 방법
        - 이 클래스로 UserDetails의 변경이 불가능한 인스턴스를 만들 수 있음

**3.2.5 사용자와 연관된 여러 책임 결합**

- 사용자를 데이터베이스에 저장 → 영속성 엔티티를 나타내는 클래스 필요
- 다른 시스템에서 웹 서비스를 통해 사용자를 가져오기 → 사용자 인스턴스를 나타내는 전송 객체 필요
    - 스프링 시큐리티 계약까지 동일한 클래스로 구현 시 → 클래스 복잡도 증가
        - 복잡한 이유? 두 책임을 혼합했기 때문!
        - User 클래스를 장식하는 SecurityUser라는 별도의 클래스를 정의 → 책임 분리

🏁 작성 완료!

### 3.3 스프링 시큐리티가 사용자를 관리하는 방법 지정

**3.3.1 UserDetailsService 계약의 이해**

![IMG_B8517C7D483C-1.jpeg](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205f73af18afc343cf917ad3a821013168/IMG_B8517C7D483C-1.jpeg)

- loadUserByUsername(String username) 메서드만 포함
    - 주어진 사용자 이름을 가진 사용자의 세부 정보를 얻는다
    - 이름이 존재하지 않으면 UsernameNotFoundException 투척

**3.3.2 UserDetailsService 계약 구현**

- 애플리케이션은 사용자의 자격 증명과 다른 측면의 세부 정보를 관리
    - 데이터베이스에 저장하거나 웹 서비스 또는 기타 방법으로 접근하는 다른 시스템 등에서 관리
- 시스템이 어떻게 작동하는지와 관계없이 스프링 시큐리티에 필요한 것은 사용자 이름으로 사용자를 검색하는 구현을 제공하는 것!

**3.3.3 UserDetailsManager 계약 구현**

- UserDetailsManager
    - UserDetailsService 계약을 확장하고 메서드를 추가
        - 새로운 사용자 추가, 삭제와 같은 사용자 관리 기능
- 사용자 관리에 JdbcUserDetailsMangager 이용
    - SQL 데이터베이스에 저장된 사용자를 관리하며 JDBC를 통해 데이터베이스에 직접 연결
    - resources 폴더에 schema.sql, data.sql 추가
        - 스프링 부트가 대신 스크립트를 실행
    - 쿼리를 재정의하는 방법도 있음
- 사용자 관리에 LdapUserDetailsManager 이용
    - 사용자 관리를 위해 LDAP 시스템을 통합해야 할 때 유용
