# 7장. 권한 부여 구성: 액세스 제한

- 권한 부여(Authorization)
    - 식별된 클라이언트가 요청된 리소스에 액세스할 권한이 있는지 시스템이 결정하는 프로세스
    - 인증(Authentication) 이후에 수행
    - 스프링 시큐리티는 인증을 완료한 후 요청을 권한 부여 필터에 위임

![IMG_8E1A0683061C-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/e062476f-c2b5-4138-ac9e-2c8c118938ad/IMG_8E1A0683061C-1.jpeg)

**7.1 권한과 역할에 따라 접근 제한**

- 권한과 역할
    - 애플리케이션의 엔드포인트를 보호하는데 이용
    - 사용자는 가진 권한에 따라 특정 작업한 실행 가능
        - 사용자를 나타내는 UserDetails 객체는 GrantedAuthority 인스턴스의 컬렉션을 가짐
        - getAuthorities() 메서드는 GrantedAuthority 인스턴스의 컬렉션을 반환
    - 권한
        - 사용자가 시스템 리소스로 수행할 수 있는 작업
        - GrantedAuthority 인터페이스의 getAuthority() 메서드로부터 얻을 수 있음

**7.1.1 사용자 권한을 기준으로 모든 엔드포인트에 접근 제한**

- 사용자의 권한을 기반으로 엔드포인트에 대한 액세스를 구성하는 메서드
    - hasAuthority()
        - 하나의 권한만 매개변수로 받음
        - 해당 권한이 있는 사용자만 엔드포인트 호출 가능
    - hasAnyAuthority()
        - 권한을 하나 이상 받을 수 있음
        - `주어진 권한 중 하나라도 있을 때` 엔드포인트 호출 가능
    - access()
        - SpEL(Spring Expression Language) 기반으로 권한 부여 규칙 구축
        - 액세스를 구성하는데 무한한 가능성 있지만 디버깅이 어려움(단점)
        - 위의 두 메서드를 적용할 수 없는 경우에만 사용하는 것이 바람직
        - 유연하다는 건 장점!
            - 접근을 허가하는 조건을 복잡한 식으로 작성해야 하는 시나리오가 있다면, access() 메서드가 아니면 시나리오 구현이 어려울 수 있음

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
		
		http.authorizeRequests()
				.anyRequest().permitAll();
				// 모든 요청에 대해 액세스 허용

		http.authorizeRequests()
				.anyRequest()
				.hasAuthority("WRITE");
				// 사용자가 엔드포인트에 접근하기 위한 조건 지정

		http.authorizeRequests()
				.anyRequests()
				.hasAnyAuthority("WRITE", "READ");
				// WRITE 및 READ 권한이 있는 사용자의 요청을 모두 허용

		http.authorizeRequest()
				.anyRequests()
				.access("hasAuthority('WRITE')");
				// WRITE 권한이 있는 사용자의 요청에 권한 부여

		String expressions = "hasAuthority('read') and !hasAuthority('delete')";

		http.authorizeRequests()
				.anyRequets()
				.access(expressions);
				// 읽기 권한이 있지만 삭제 권한은 없는 사용자의 요청을 모두 허용
	}
}
```

**7.1.2 사용자 역할을 기준으로 모든 엔드포인트에 대한 접근을 제한**

- 역할
    - 사용자가 수행할 수 있는 작업을 나타내는 다른 방법 ⇒ 작업 그룹에 속한 이용 권리를 제공
    - 역할을 이용하면 권한을 정의할 필요 X
        - 권한은 개념상으로 존재
    - 역할을 정의할 때 역할의 이름은 접두사 _ROLE로 시작해야 함
        - 역할을 선언할 때는 접두사를 사용하지만 역할을 이용할 때는 접두사를 제외함!
    - 사용자 역할에 대한 제약 조건을 설정하기 위한 메서드
        - hasRole()
            - 요청을 승인할 하나의 역할 이름을 매개 변수로 받음
        - hasAnyRole()
            - 요청을 승인할 여러 역할 이름을 매개 변수로 받음
        - access()
            - 요청을 승인할 역할을 스프링 식으로 지정

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	@Bean
	public UserDetailsService = userDetailsService() {
		var manager = new InMemoryUserDetailsManager();

		var user1 = User.withUsername("john")
										.password("12345")
										.authorities("ROLE_ADMIN")
										.build();
										// ROLE 접두사가 있으므로 GrantedAuthority는 역할을 나타냄

		var user2 = User.withUsername("jain")
										.password("12345")
										.roles("ADMIN")
										.build();
										// roles() 메서드에는 접두사가 없음에 유의!
										// 접두사가 포함되면 메서드가 예외를 투척

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
		
		http.authorizeRequests()
				.anyRequest().hasRole("ADMIN");
				// 엔드포인트에 접근할 수 있는 역할 지정
				// 여기에는 ROLE_ 접두사가 없음에 유의!
	}
}
```

**7.1.3 모든 엔드포인트에 대한 접근 제한**
```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
		
		http.authorizeRequests()
				.anyRequest().denyAll();
				// 모든 사용자의 접근을 제한
	}
}
```
