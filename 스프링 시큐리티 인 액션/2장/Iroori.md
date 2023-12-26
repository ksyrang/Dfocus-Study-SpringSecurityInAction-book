# 01부 2장. 안녕! 스프링 시큐리티

생성자: 이루리
생성 일시: December 26, 2023 3:10 PM

## 안녕! 스프링 시큐리티

- 스프링 시큐리티 첫번째 프로젝트
- 인증과 권한 부여를 위한 기본 구성 요소로 간단한 기능 설계
- 이러한 구성 요소가 서로 어떻게 연관 되는지 이해하기 위한 기본 계약 적용
- 주 책임에 대한 구현 작성
- 스프링 부트의 기본 구성 재정의

---

- SpringSecurity인증 처리 흐름
    
    ![2wkd.png](01%E1%84%87%E1%85%AE%202%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%AB%E1%84%82%E1%85%A7%E1%86%BC!%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%89%E1%85%B5%E1%84%8F%E1%85%B2%E1%84%85%E1%85%B5%E1%84%90%E1%85%B5%201b19a241e06741d4808861a32f7bd578/2wkd.png)
    

## 첫 번째 프로젝트 시작

- spring security, spring-boot-starter-web 종속성 추가
- HTTP Basic을 인증을 이용해 엔드포인트 보호하는 방법
    - 엔드포인트 호출 시 password필요한데 이는 기본적으로 스프링 시큐리티가 제공하는 사용자 이름과 제공된 암호를 사용하면 됨.
    - 사용시 올바른 응답 반환

## 기본 구성

인증과 권한 부여를 처리하는데 참여하는 주 구성 요소

- UserDetailsService
    - 사용자의 세부 정보는 UserDetailsService 계약을 구연하는 객체가 관리
    - 기본 자격 증명 사용자 이름 ‘user’ 이며 기본 암호는 UUID 형식 → 스프링 컨텍스트가 로드 될 때 자동 생성
    - 자격 증명을 메모리에 보관. 즉, 애플리케이션은 자격 증명을 보존하지 않는다.
- PasswordEncoder
    - 암호 인코딩
    - 암호가 기존 인코딩과 일치하는지 확인

✍️ HTTP Basic 인증은 자격증명의 기밀성을 보장하지 않음. 단지 전송의 편의를 위한 인코딩 방법 이용. 암호화나 해싱 방법이 아니라 전송 중 자격 증명 가초래 시 누구나 볼 수 있음.

- AuthenticationProvider
    - 인증 논리 정의 및 사용자와 암호의 관리를 위임
    - UserDetailsService와 PasswordEncoder 제공된 기본 구현 이용

## 기본 구성 요소 재정의

UserDetailsService 와 PasswordEncoder

- 콘솔에 자동 생성 암호 출력되지 않음

NoOpPasswordEncoder 인스턴스는 암호에 암호화나 해시 적용하지 않고 일반 텍스트처럼 처리

```java
@Configuration // # 클래스를 구성 클래스로 표시
public class ProjectConfig{
	@Bean // 반한된 값을 스프링 컨텍스트에 빈으로 추가하도록 스프링에 지시
	public UserDetailsService userDetailsService(){
		var userDetailsService = new InMemoryUserDetailsManager();
		
		var user = User.withUsername("john");
										.password("12345")
										.authorities("read")
										.build();
		userDetailsService.createUser(user);		

		return userDetilsService;
	}

	@Bean
	public passwordEncoder passwordEncoder(){
		return NoOpPasswordEncoder.getInstance();
	}
}
```

## 엔드포인트 권한 부여 구성 재정의

- WebSecurityConfigureAdapter 확장 → 최신 6.0?이상 버전에서 사용 못함
    - configure(HttpSecurity http) 메서드 재정의
    
    ```java
    @Configuration
    public class ProjectConfig extends WebSecurityConfigurerAdapter{
    	@Override
    	protected void configure(HttpSecurity http) throws Exception {
    		http.httpBasic());
    		http.authorizeRequests()
    				// .anyRequest().authenticated(); // 모든 엔드포인드 인증 필요
    				.anyRequest().permitAll(); // 인증 없이 요청 가능
    	}
    }
    ```
    

## 다른 방법의 기본 구성 요소 재정의(configure)

- 두 객체를 빈으로 정의하는 대신 configure(AuthenticationManagerBuilderauth)메서드로 설정 가능

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter{
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception{
		var userDetailsService = new InMemoryUserDetailManager();  // 사용자 메모리에 저장하기 위한 선언
		var user = User.withUsername("john");
										.password("12345")
										.authorities("read")
										.build();
		userDetailsService.createUser(user);		
		
	// AuthenticationManagerBuilder에 매개변수를 userDetailsService와 passwordEncoder를 설정
		auth.userDetailsService(userDetailsService)
				.passwordEncoder(NoOpPasswordEncoder.getInstance());
	}
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic());
		http.authorizeRequests()
				 .anyRequest().authenticated(); // 모든 엔드포인드 인증 필요
				//.anyRequest().permitAll(); // 인증 없이 요청 가능
	}
}
```

## AuthenticationProvider 구현 재정의

- AuthenticationManager에서 요청을 받은 후 사용자를 찾는 작업을 UserDetilsService에 암호를 검증하는 작업을 PasswordEncoder에 위임

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider{
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationExceptioN {
		//인증논리 추가
		String username = authentication.getName(); // Principal 인테퍼에스에서 상속 userId
		String password = String.valueOf(authentication.getCredentials()); // credential user의 비밀 정보 
		
		if("john".equlas(username)&&"12345".equlas(password)){
			return new UsernamePasswordAuthenticationToken(username,password,Arrays.asList());
		}	else{
			throw new AuthenticationCredentialsNotFoundException("Error in authentication");
	}
}
	
	@Override
	public boolean supports(Class<?> authenticationType) {
		// Authentication 형식의 구현을 추가할 위치
				return new UsernamePasswordAuthenticationToken.class
									.isAssignableFrom(authenticationType);
	}
}
```

## 요약

- 스프링 시큐리티를 애플리케이션 종속성으로 추가하면 스프링부트가 기본 구성을 일부 제공
- 인증과 권한 부여를 위한 기본 구성요소 UserDetailsService, PasswordEncoder, AuthenticationProvider 재구현 가능
- 스프링 시큐리티는 UserDetailService의 간단한 구현인 InMemoryUserDetailsManager제공 → UserDetailsService의 인스턴스와 같은 사용자를 추가해 메모리에서 관리 가능 → 메모리에서 관리해~~
- NoOpPasswordEncoder는 PasswordEncoder 구현하는데 암호를 일반 텍스트로 처리→ 일반 애플리케이션에서는 적합 x
- AuthenticationProvider 재정의를 통해 애플리케이션의 맞춤형 인증 논리 구현 가능
- 구성을 작성하는 방법은 여러가지이지만 한 애플리케이션에서 한 방법을 선택하고 고수해야하는 코드를 깔끔하고 이해하기 쉽게 만들수 있음.