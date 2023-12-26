# 2장. 안녕! 스프링 시큐리티

1. 스프링 시큐리티와 웹 종속성만 있는 프로젝트를 만들어서 추가된 구성이 없으면 어떻게 작동하는지 확인
2. 기본 구성을 재정의하고 맞춤형 사용자와 암호를 정의 → 사용자 관리 기능을 추가
3. 애플리케이션이 기본적으로 모든 엔드포인트를 인증하는 것을 확인하고 이 동작을 맞춤 구현
4. 모범 사례를 이해할 수 있도록 동일한 구성에 대해 다른 스타일을 적용

### 2.1 첫 번째 프로젝트 시작

```jsx
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 스프링 컨텍스트의 기본 구성을 적용
- 스프링 애플리케이션을 구동시키면 콘솔에 HTTP Basic 인증을 위한 암호가 생성

```jsx
Using generated security password : xxxxxx-xxxxx-x-x-xxx-xxx

curl -u user:xxxxx-xxxx-xxx-x-x-x- http://localhost:8080/hello

echo -n user:xxxx-xxxx-xxxx-x--x-x-x |base64

curl -H "Authorization: Basic xxxxxxxxxxxxxxxxxxxxxx=" http://localhost:8080/hello
```

### 2.2 기본 구성이란?

![IMG_2DDE5A6BD1AA-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/ded66099-ae3b-4042-872a-9b8c0260f986/IMG_2DDE5A6BD1AA-1.jpeg)

- 인증 필터
    - 인증 요청을 인증 관리자에게 위임
    - 응답이 오면 응답을 바탕으로 보안 컨텍스트를 구성
- 인증 관리자
    - 인증 공급자를 이용해 인증을 처리
- 인증 공급자
    - 인증 논리를 구현
    - 사용자 관리 책임을 구현하는 사용자 세부 정보 서비스를 인증 논리에 이용
    - 암호 관리를 구현하는 암호 인코더를 인증 논리에 이용
- 보안 컨텍스트
    - 인증 프로세스 후 인증 데이터를 유지
- UserDetailsService
    - UserDetailsService의 구현 객체가 사용자에 관한 세부 정보를 관리
        - 기본 구현은 내부 메모리에 기본 자격 증명을 등록하는 일만 수행
- PasswordEncoder
    - 암호를 인코딩
    - 암호가 기존 인코딩과 일치하는지 확인
    - UserDetailsService의 기본 구현을 대체할 때는 PasswordEncoder도 지정해야 함
- AuthenticationProvider
    - 인증 논리를 정의하고 사용자와 암호의 관리를 위임
    - UserDetailsService 및 PasswordEncoder에 제공된 기본 구현을 이용

### 2.3 기본 구성 재정의

**2.3.1 UserDetailsService 구성 요소 재정의**

**2.3.2 엔드포인트 권한 부여 구성 재정의**

```jsx
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
		http.authorizeRequests()
			.anyRequest().authenticated();
			// .anyRequest().permitAll();
	}
}
```

**2.3.4 AuthenticationProvider 구현 재정의**

- AuthenticationProvider로 맞춤 구성 인증 논리를 구현
- authenticate() 메서드는 인증의 전체 논리를 나타냄
- configuration 파일에서 재정의한 AuthenticationProvider를 등록

```jsx
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {

	@Autowired
  private CustomAuthenticationProvider authenticationProvider;

	@Override
	protected void configure(AuthenticationManager auth) {
		auth.authenticationProvider(authenticationProvider);
	}
}
```

**2.3.5 프로젝트에 여러 구성 클래스 이용**

- 구성 클래스도 책임을 분리하는 것이 좋다
    - 항상 한 클래스가 하나의 책임을 맡도록 하는 것이 바람직
    - 단 분리한 클래스들이 모두 WebSecurityConfigurerAdapter를 확장할 수는 없으며 그렇게 하면 종속성 주입이 실패
        - 어노테이션으로 주입의 우선순위를 설정하면 종속성 주입 문제는 해결할 수 있지만 구성이 병합되지 않고 서로를 제외하므로 가능상 작동 X

### 요약

- 스프링 시큐리티를 애플리케이션의 종속성에 추가하면 스프링 부트가 약간의 기본 구성을 제공
- 구성을 작성하는 방법은 여러 가지가 있지만, 한 애플리케이션에서는 한 방법을 선택하고 고수해야 코드를 깔끔하고 이해하기 쉽게 만들 수 있다
