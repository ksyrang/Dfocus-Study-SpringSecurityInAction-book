# 3장 사용자 관리

## 이 단원의 내용

- UserDetails 인터페이스로 사용자 기술하기
- 인증 흐름에 UserDetailsService 이용하기
- UserDetailsService의 맞춤형 구현 만들기
- UserDetails"anager의 맞춤형 구현 만들기
- 인증 흐름에 JdbcUserDetails"anager 이용하기

### 소주제

스프링 시큐리티의 인증 구현

키워드

관련 질문

### 노트

- 스프링 시큐리티 인증 흐름
    
    ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled.png)
    
- UserDetailsManager는 대부분의 애플리케이션에 필요한 User 추가, 수정, 삭제 작업을 추가 한다.
- 두 계약 간의 분리는 인터페이스 분리 원칙의 훌륭한 예다. 인터페이스를 분리하면 앱에 필요 없는 동작을 구현하도록 프레임워크에서 강제하지 않기 때문에 유연성이 향상된다.
- 사용자를 인증하는 기능만 필요한 경우 UserDetailsService 계약만 구현하면 필요한 기능을 제공할 수 있다.
- 사용자는 하나 이상의 권한을 가진다.
- 사용자 관리 부분의 구성 요소 간의 관계
    
    ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled%201.png)
    

<aside>
📌 **요약: 스프링 시큐리티 인증 구현은 인터페이스를 통해 유연성 있게 구현 할 수 있다.**

</aside>

### 소주제

사용자 기술 하기

키워드

관련 질문

### 노트

- 프레임워크가사용자를 인식할 수 있게 사용자를 나타내는 방법을 배우는 것은 인증 흐름을 구축하기 위한 필수 단계.
- 애플리케이션은 사용자가 누구 인지에 따라 특정 기능을 호출할 수 있는지 여부를 결정.
- 스프링 시큐리티에서 사용자 정의는 UserDetails 계약을 준수.
- UserDetails 계약은 스프링 시큐리티가 이해하는 방식으로 사용자를 나타낸다
- GrantedAuthority 인터페이스
    - 사용자에게 허가된 작업을 권한(authority)이라고 한다
    - 권한은 사용자가 애플리케이션에서 수행할 수 있는 작업을 나타낸다.
    - 권한이 없으면 모든 사용자가 동등하다.
    - 애플리케이션은 사용자가 필요로 하는 권한인 애플리케이션 기능 요구 사항에 따란 다른 유형의 사용자를 구분해야 한다.
    - 
- UserDetails 클래스 만들기
    - UserDetails 클래스 상속받아서 클래스 만들기
    - UserBuilder를 이용하여 User 인스턴스
    
    ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled%202.png)
    
- 사용자와 연관된 여러 책임 결합
    - 하나의 클래스에 여러 책임과 관련된 메소드가 있으면 바람직 하지 않다.
    - 책임별로 클래스를 나누어 사용하는 것이 바람직하다.
        - JPA Entity용 클래스를 객체로 이용해 userDetail을 implement 받아서 메소드를 구현
            
            ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled%203.png)
            
    - 위의 SecurityUser 클래스는 시스템에서 사용자 세부 정보를 스프링 시큐리티가 이해하는 UserDetails 계약에 매핑하는 일만 한다. SecurityUser에 User Entity가 반드시 필요하다는 것을 지정하기 위해 필드를 final로 지정했다. 사옹자는 생성자를 통해 지정해야 한다. SecurityUser Class로 User Entity Class를 장식하고 스프링 시큐리티 계약에 필요한 코드를 추가해서 JPA Entity에 코드를 섞어 결과적으로 여러 다른 작업을 구현하지 않도록 했다.
    - 애플리케이션의 유지 관리성을 높이려면 책임을 흔합하지 말고 최대한 분리해서 코드를 작성해야 한다.

<aside>
📌 **요약: 유저 기술할때 책임을 혼합 시키지 말**

</aside>

### 소주제

스프링 시큐리티가 사용자를 관리하는 방법 지정

키워드

관련 질문

### 노트

- UserDitalsManager 인터페이스를 이용해 자격 증명 비교, 사용자 추가 와 기존 사용자 변경하는 방법을 실
- UserDetailsService 인터페이스
    - UserDetailsService 기본적으로 하나의 메소드만 가지고 있다.
        
        ```java
        public interface UserDetailsService {
            UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
        }
        ```
        
    
    ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled%204.png)
    
- UserDetailsService 구현
    
    ```java
    @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    
            return users.stream().filter(u->u.getUsername().equals(username))
                    .findFirst()
                    .orElseThrow(()-> new UsernameNotFoundException("User not found"));
    
        }
    ```
    
- UserDetailsManager 구현
    - UserDetailsService에서 사용자 추가 또는 삭제 등의 확장된 기능을 포함한다.
    - InMemoryUserDetailsManager 또한 UserDetailsManager의 구현 클래스이다.
    - 또 다른 구현 클래스로 JdbcUserDetailsManager가 있다.
- JdbcUserDetailsManager
    
    ![Untitled](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%8C%E1%85%A1%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20b08e3e3d325c437db3f34ccfcbb28442/Untitled%205.png)
    
- LdapUserDetailsManager
    
    LDAP용 UserDetailsManager 도 제공한다. 이는 LDAP 시스템을 통합해야 할 때 유용하다.
    

<aside>
📌 **요약: 사용자 관리를 위해** UserDetailsService로 커스텀하게 구현하고 제공되는 Manager를 잘 활용하자

</aside>

## 3장 요약

- UserDetails 인터페이스는 스프링 시큐리티에서 사용자를 기술하는데 이용되는 계약(인터페이스)이다.
- UserDetailsService 인터페이스는 애플리케이션이 사용자 세부 정보를 얻는 방법을 설명하기 위해 스프링 시큐리티의 인증 아키텍처에서 구현해야 하는 계약이다.
- UserDetailsManager 인터페이스는 UserDetailsService를 확장하고 사용자 생성, 변경 삭제에 관련된 동작을 추가한다.
- 스프링 시큐리티는 UserDetailsManager 계약의 여러 구현을 제공한다. 이러한 구현에는 InMemoryUserDetailsManager, JdbcUserDetailsManager, LdapUserDetailsManager가 있다.
- JdbcUserDetailsManager는 JDBC를 직접 이용하므로 애플리케이션이 다른 프레임워크에 고정되지 않는다는 이점이 있다.
