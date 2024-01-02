# 02부 3장. 사용자관리

생성자: 이루리
생성 일시: December 31, 2023 7:02 PM

- UserDetails 인터페이스로 사용자 기술
- 인증 흐름에 UserDetailsService 이용
- UserDetailsService의 맞춤형 구현 만들기
- UserDetailsManager의 맞춤형 구현 만들기
- 인증 흐름에 JdbcUserDetailsManager 이용하기

> UserDetailsService 와 함께 다룰 내용
> 
> - 스프링 시큐리티에서 사용자를 기술하는 UserDetails
> - 사용자가 실행할 수 있는 작업을 정의하는 GrantedAuthority
> - UserDetailsService 계약을 확장하는 UserDetailsManager. 상속된 동작 외에 사용자 만들기, 사용자의 암호 수정이나 삭제 등의 작업도 지원

- UserDetailsService 와 PasswordEncoder
    - 사용자 세부 정보와 자격 증명을 직접 처리하는 구성 요소

### 스프링 시큐리티 인증 흐름

1. AuthenticationFilter (인증필터) 요청을 가로챈다.
2. AuthenticationMangager (인증관리자) 인증 책임 위임한다.
3. AuthenticationManager (인증관리자)는 인증 논리를 구현하기 위해 AuthenticationProvider(인증 공급자) 이용한다.
4. AuthenticationProvider (인증 공급자)는 사용자 이름과 암호를 확인하기 위해 UserDeatailsService(사용자 세부 정보 서비스) 및 PasswordEncoder(암호 인코더)를 이용해 검증한다. 
5. 인증 결과가 필터에 반환되고
6. 인증된 엔티티에 관한 세부 정보가 보안 컨텍스트에 저장된다.

### UserDetailsService 및 UserDetailsManager

- 사용자 관리를 위해 두 인터페이스 이용
- UserDetailsService
    - 사용자 이름으로 사용자를 검색하는 역할만 함.
    - 프레임워크가 인증을 완료하는데 반드시 필요한 유일한 작업
- UserDetailsManager
    - 대부분의 애플리케이션에 필요한 사용자 추가, 수정, 삭제 작업 추가
- 두 계약 간의 분리는 인터페이스 분리 원칙의 훌륭한 예
    - 인터페이스를 분리하면 앱에 필요 없는 동작을 구현하도록 프레임워크에서 강제하지 않기 때문에 유연성 향상
    - ex) 사용자 인증하는 기능만 필요한 경우 : UserDetailsService 계약만 구현
    - ex) 사용자를 관리하려면 UserDetailsService 및 UserDetailsManager 구성 요소에 사용자를 나타내는 방법 필요

### 사용자 관리를 수행하는 구성 요소 간의 종속성

1. UserDetailsService는 사용자 이름으로 찾은 사용자 세부 정보를 반환한다.
2. UserDetails 계약은 사용자를 기술한다.
3. 사용자는 GrantedAuthority 인터페이스로 나타내는 권한을 하나 이상 가진다.
4. UserDetailsManager 계약은 UserDetailsService를 확장해서 암호 생성, 삭제 , 변경 등의 작업을 추가한다.

## 3.2 사용자 기술하기

- 애플리케이션의 사용자를 기술하는 방법
- UserDetails 인터페이스
    - getUserName(), getPassword() 메서드는 각각 사용자 이름과 암호를 반환
    - → 앱에서 인증 과정에 사용, 인증과 관련된 유일한 세부 정보
    - 나머지 다섯 메서드는 사용자가 애플리케이션의 리소스에 접근할 수 있도록 권한을 부여하기 위한것
    - 사용자 계정을 필요에 따라 활성화 또는 비활성화 하는 네 메서드 (GrantedAutority인스턴스 컬렉션으로 반환하는 메소드 제외)
    - UserDetails 계약을 보면 사용자는 다음과 같은 작업을 할 수 있음
        - 계정 만료
        - 계정 잠금
        - 자격 증명 만료
        - 계정 비활성화
            - → 네 메서드를 재정의해야함

```java
public interface UserDetails extends Serializable{
		Collection<? extends GrantedAuthority> getAuthorities(); // 앱 사용자가 수행할 수 있는 작업을 GrantedAutority  인스턴스의 컬렉션으로 반환
	
    String getPassword(); // 사용자 자격 증명 반환

    String getUsername(); // 사용자 자격 증명 반환

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();

}
```

### 3.2.2 GrantedAuthority 계약 살펴보기

- 권한을 정의하는 방법
- 스프링 시큐리티에서는 GrantedAutority 인터페이스로 권한을 나타낸다.
    - → 이 인터페이스는 사용자 세부 정보의 정의에 이용되며 사용자에게 허가된 이용 권리를 나타낸다.
    - 사용자는 권한이 하나도 없거나 여러 권한을 가질 수 있지만 일반적으로 하나 이상의 권한을 가진다.
    
    ```java
    public interface GrantedAuthority extends Serializable {
    	String getAuthority();
    }
    ```
    
    - 권한을 만들려면 나중에 권한 부여 규칙을 작성할 때 참조할 수 있게 해당 이용 권리의 이름만 찾으면 된다. 권한 이름을 String 으로 반환하도록 getAuthority()를 구현.
    - GrantedAuthority를 구현하는 두 예시
    
    ```java
    GrantedAuthority g1 = () -> "READ"; // 람다식
    // ** 람다식 구현전 @FunctionalInterface 어노테이션을 지정해 인터페이스가 함수형임을 지정하는 것임이 좋음.
    GrantedAuthority g2 = new SimpleGrantedAuthority("READ");
    
    ```
    

### 3.2.3 최소한의 UserDetails 구현 작성

- DummyUser 클래스
    - 권한 목록을 위한 정의 → getAuthorities() 메서드 구현
    - 이 구현은 모든 인스턴스는 같은 사용자를 나타내는데, 실제 애플리케이션에서는 다른 사용자를 나타내는 인스턴스를 생성할 수 있도록 클래스를 작성해야함.

```java
public class DummyUser implements UserDetails{
	@Override
	public String getUsername(){
		return "bill";
	}

	@Override
	public String getPassword(){
		return "12345";
	}
	
	@Override
	public Collection<? extends GrantedAuthority> getAuthorities(){
		return List.of(()->"READ");
	}
	
	// 모두 true 반환 사용자가 항상 활성화되고 사용 가능하다는 의미
	@Override
	public boolean isAccountNonExpired(){
		return true;
	}
	
	@Override
	public boolean isAccountNonLocked(){
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired(){
		return true;
	}

	@Override
	public boolean isEnabled(){
		return true;
	}
}
```

- 더 현실적인 UserDetails 인터페이스 구현

```java
public class SimpleUser implements UserDetails{
	private final String username;
	private final String password;

	public SimpleUser(String username, String password){
		this.username = username;
		this.password = password;
	}

	@Override
	pubic String getUsername(){
		return this.username;
	}

	@Override
	public String getPassword(){
		return this.password;
	}
}
```

### 3.2.4 빌더를 이용해 UserDetail 형식의 인스턴스 만들기

- 스프링 시큐리티에 있는 빌더 클래스로 간단한 사용자 인스턴스 만들 수 있음.

```java
User.UserBuilder builder1 = User.withUsername("bill"); //주어진 사용자 이름으로 사용자 생성
UserDetails u1 = builder1
									.password("12345")
									.authorities("read","write")
									.passwordEncoder(p -> encode(p))
									.accountExpired(false)
									.disabled(true)
									.build(); // 빌더 파이프라인 끝에서 메서드 호출
User.UserBuilder builder2 = User.withUserDetails(u);  // 기존 UserDetails 인스턴스에서 사용자를 만들 수도 있음.

UserDetails u2 = builder.builder();
```

### 3.2.5 사용자와 연관된 여러 책임 결합

- 사용자가 여러 책임을 갖는 것이 일반적
- User 클래스를 장식하는 SecurityUser라는 별도의 클래스를 정의해 책임을 분리
    - SecurityUser 클래스는 UserDetails 계약을 구현하고 이를 이용해서 사용자를 스프링 시큐리티 아키텍처에 연결→ User클래스는 JPA 엔티티 책임만 남아있음.
    - @Entity User 클래스는 JPA엔티티 책임만있기때문에 getAuthorities()를 직접 오버라이딩하지않고 “private String authority; ”로 명시만 해놓고 SecurityUser클래스를 구현해 User 엔티티를 래핑한다.
    
    ```java
    public class SecurityUser implements UserDetails{
    	private final User user;
    
    	public SecurityUser(User user){
    		this.user = user;
    	}
    
    	@Override
    	public String getUsername(){
    		return user.getUsersname;	
    	}
    	
    	@Override
    	public String getUsername(){
    		return user.getUsersname;	
    	}
    
    	@Override
    	public String getPassword(){
    		return user.getPassword;	
    	}
    	@Override
    	public Collection<? extends GrantedAuthority> getAuthroties(){
    		return List.of(() -> user.getAuthority());	
    	}
    }
    ```
    
    - SecurityUser 클래스는 시스템에서 사용자 세부 정보를 스프링 시큐리티가 이해하는 UserDetails 계약에 매핑하는 일만 함.
    - SecurityUser에 User 엔티티가 반드시 필요하다는 것을 지정하기위해 필드를 final로 지정

## 3.3 스프링 시큐리티가 사용자를 관리하는 방법 지정

- UserDetailsService : 특정 구성 요소로 인증 프로세스가 사용자 관리를 위임
- UserDetailsService 계약에 기술된 책임을 구현해 사용자 관리가 작동하는 방식
- UserDetailsManager 인터페이스로 UserDetailsService로 정의된 계약에 더 많은 동작 추가 가능

### 3.3.1 UserDetailsService 계약의 이해

```java
public interface UserDetailsService{
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

- 인증 구현은 loadUserByUsername(String username) 메서드를 호출해 주어진 사용자 이름을 가진 사용자의 세부 정보를 얻는다.
- 사용자의 이름은 고유하다고 간주함
- 이 메서드가 반환하는 사용자는 UserDetails 계약의 구현임.
- 사용자가 존재하지 않을시 메서드가 UsernameNotFoundException 투척.
    - UsernameNotFoundException은 RuntimeException이다. 인증프로세스와 연관된 모든 예외의 상위 항목인 AuthenticationException 형식에서 곧바로 상속하는데 AuthenticationException 은 추가로 RuntimeException 클래스를 상속한다.
- AuthenticationProvider는 인증논리에서 UserDetailsService를 이용해 사용자 세부 정보를 로드함 (loadUserByUsername) → 데이터베이스, 외부시스템, 볼트 등에서 사용자를 로드하도록 UserDetailsService를 구현할 수있음.

### 3.3.2 UserDetailsService 계약 구현

- 애플리케이션은 사용자의 자격증명과 다른 측면의 세부 정보를 관리하는데, 이런 정보는 데이터베이스에 저장하거나 웹 서비스 또는 기타 방법으로 접근하는 다른 시스템에서 관리할 수 있다. 시스템이 어떻게 작동하는지와 관계없이 스프링 시큐리티에 필요한 것은 사용자 이름으로 사용자를 검색하는 구현을 제공하는 것.
- isAccounteNotExpired() → true 반환할 경우 계정은 만료되거나 잠기지 않음.

```java
public class InMemoryUserDetailsService implements UserDetailsService {

    private final List<UserDetails> users; // UserDetailsService는 메모리 내 사용자 목록을 관리한다.

    public InMemoryUserDetailsService(List<UserDetails> users) {
        this.users = users;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return users.stream()
                .filter(u -> u.getUsername().equals(username)) // 사용자 목록에서 요청된 사용자 이름과 일치하는 항목 필터링.
                .findFirst()// 일치하는 사용자가 있으면 반환
                .orElseThrow(() -> new UsernameNotFoundException("User not found")); // 없을 경우 예외 반환
    }
}
```

- loadUserByUsername(String username) 메서드는 주어진 사용자 이름으로 사용자의 목록을 검색하고 원하는 UserDetails 인스턴스를 반환하고 주어진 사용자 이름이 발견되지 않으면 UsernameNotFoudException 예외 반환
    - 위의 코드를 시큐리티 config 구성 클래스에 빈으로 추가

### 3.3.3 UserDetailsManager 계약 구현

- UserDetailsManager는 UserDetailsService 계약을 확장하고 메서드를 추가한다.
- 스프링 시큐리티가 인증을 수행하려면 UserDetailsService 계약이 필요함.
    - 일반적으로 애플리케이션에는 사용자를 관리하는 기능이 필요하고 대부분의 앱은 최소한 새 사용자를 추가하거나 기존 사용자를 삭제 할 수 있어야함. 이 때 스프링 시큐리티에 정의된 더 구체적인 인터페이스인 UserDetailsManager를 구현.
    - → 애플리케이션에서 사용자를 관리하는 기능을 구체적으로 할수 있게 해주는 계약
- 사용자 관리에 JdbcUserDetailsManager 이용
    - SQL 데이터베이스에 저장된 사용자를 관리하며 JDBC를 통해 데이터베이스에 직접 연결한다.
    - JdbcUserDetailsManager가 데이터베이스에 연결되려면 DataSource 가 필요.
    - 애플리케이션의 엔드포인트에 접근하려면 데이터베이스에 저장된 사용자중 하나와 HTTP Basic 인증을  이용해야함.
    - 구현에 이용되는 모든 쿼리 변경 가능
- 사용자 관리에 LdapUserDetailsManager 이용
    - 사용자 관리를 위해 LDAP 시스템을 통합해야할 때 유용
    - LDAP란?
        
        LDAP( Lightweight Directory Access Protocol) 서버는 경량 디렉터리 액세스 프로토콜을 사용하여 디렉터리 서비스를 제공하는 서버입니다. 디렉터리 서비스는 데이터를 계층적으로 저장하고 검색할 수 있는 서비스를 말하며, 일반적으로 조직의 사용자, 그룹, 자원 등과 관련된 정보를 저장합니다.
        
        LDAP는 주로 사용자 인증, 권한 부여 및 디렉터리 검색과 같은 작업에 사용됩니다
        

## 요약

- UserDetails 인터페이스는 스프링 시큐리티에서 사용자를 기술하는데 이용되는 계약
- UserDetailsService 인터페이스는 애플리케이션이 사용자 세부 정보를 얻는 방법을 설명하기 위해서 스프링 시큐리티의 인증 아키텍쳐에서 구현해야하는 계약
- UserDetailsManager 인터페이스는 UserDetailsService를 확장하고 사용자 생성, 변경, 삭제와 관련된 동작을 추가한다.
- 스프링 시큐리티는 UserDetailsManager 계약의 여러 구현을 제공한다. 이러한 구현에는 InMemoryUserDetailsManager, JdbcUserDetailsManager, LdapUserDetailsManager가 있다.
- JdbcUserDetilasManager는 JDBC를 직접 이용하므로 애플리케이션이 다른 프레임워크에 고정되지 않는다는 이점이 있다.
-