# 8장. 권한 부여 구성: 제한 적용

- 특정한 요청 그룹에만 권한 부여 제약 조건을 적용하려면?
    - 권한 부여 구성을 적용할 요청을 선택하는 선택기 메서드
        - MVC 선택기
            - 경로에 MVC 식을 이용해 엔드포인트를 선택
        - 앤트 선택기
            - 경로에 앤트 식을 이용해 엔드포인트를 선택
        - 정규식 선택기
            - 경로에 정규식(regex)를 이용해 엔드포인트를 선택

**8.1 선택기 메서드로 엔드포인트 선택**

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();

		http.authorizeRequests()
				.mvcMatchers("/hello").hasRole("ADMIN")
				.mvcMatchaers("/ciao").hasRole("MANAGER")
				.anyRequest().permitAll();
				// permitAll()은 나머지 모든 엔드포인트에 대한 요청을 허용
				// .anyRequest().permitAll()을 작성하지 않아도 규칙은 동일하게 적용
				// 모든 규칙을 명시적으로 지정하여 사용자의 접근을 허용한다는 의도를 명확하게 드러냄
	}
}
```

- 특정한 규칙부터 일반적인 규칙의 순서로 지정해야!
    - anyRequest() 메서드를 mvcMatchers() 보다 먼저 호출할 수 없음
- 미인증과 인증 실패
    - 엔드포인트를 모두 접근할 수 있게 설계했다면 인증을 위한 사용자 정보 제공이 없어도 호출 가능
        - 인증 프로세스 생략
    - 하지만 굳이 사용자 이름과 암호를 제공했다면 인증 프로세스를 진행
    - 애플리케이션은 권한 부여 이전에 항상 인증을 수행
        - 권한 부여 필터가 경로에 대한 모든 요청을 허용하더라도 권한 부여보다 먼저 인증 논리를 실행하므로 인증 필터는 요청을 권한 부여 필터로 전달하지 않고 `http 401 권한없음`으로 응답
            
            ![IMG_4DD80F1E5908-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/bff08ffe-0bc2-4b7c-a7f0-889d1b8f2bee/IMG_4DD80F1E5908-1.jpeg)
            

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();

		http.authorizeRequests()
				.mvcMatchers("/hello").hasRole("ADMIN")
				.mvcMatchaers("/ciao").hasRole("MANAGER")
				.anyRequest().authenticated();
				// 인증된 사용자에게만 나머지 모든 요청을 허용
	}
}
```

**8.2 MVC 선택기로 권한을 부여할 요청 선택**

- MVC 선택기
    - 표준 MVC 구문으로 경로를 지정
        - 어노테이션으로 엔드포인트 매핑을 작성할 때의 구문과 동일
    - mvcMatchers(HttpMethod method, String patterns)
        - 제한을 적용할 HTTP 방식과 경로를 모두 지정
        - 같은 경로에 대해 HTTP 방식별로 다른 제한을 적용할 때 유용
    - mvcMatchers(String patterns)
        - 경로만을 기준으로 권한 부여 제한을 적용
        - 해당 경로의 모든 HTTP 방식에 제한 적용

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();

		http.authorizeRequests()
				.mvcMatchers(HttpMethods.GET, "/a")
					.authenticated()
				.mvcMatchers(HttpMethods.POST, "/a")
					.permitAll()
				.anyRequest()
					.denyAll();
				// HTTP GET 방식으로 /a 경로 요청 시 사용자 인증 필요
				// HTTP POST 방식으로 /a 경로 요청 시 모두 허용
				// 나머지 경로에 대한 모든 요청 거부
	}
}
```

![IMG_EABB2459DBF7-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/4281a2f9-661b-4bdf-b0e6-aede025d4c5a/IMG_EABB2459DBF7-1.jpeg)

**8.3 앤트 선택기로 권한을 부여할 요청 선택**

- antMatchers(HttpMethod method, String patterns)
    - 제한을 적용할 HTTP 방식과 경로를 참조할 앤트 패턴을 모두 지정
    - 같은 경로 그룹에 대해 HTTP 방식별로 다른 제한을 적용할 때 유용
- antMatchers(String patterns)
    - 경로만을 기준으로 권한 부여 제한을 적용
    - 모든 HTTP 방식에 자동으로 제한이 적용
- antMatchers(HttpMethod method)
    - antMatchers(HttpMethod, “/**”)과 같은 의미
    - 경로와 관계없이 특정 HTTP 방식을 지정
- MVC 선택기는 컨트롤러 동작에 대한 일치 요청을 스프링이 이해하는 방법으로 지정
    - 스프링은 동일한 작업에 대한 모든 경로 뒤에 다른 /를 추가해도 해석 가능
        - 예시) /hello 와 /hello/ 경로는 자동으로 같은 규칙으로 보호
    - 앤트 선택기 사용 시 → 의도치 않게 보호되지 않은 경로가 방치될 가능성 있음
- MVC 선택기 사용을 권장
    - 스프링의 경로 매핑과 관련한 위험의 예방이 가능
    - 권한 부여 규칙을 위해 경로를 해석하는 방법과 스프링이 경로를 엔드포인트에 매핑하기 위해 해석하는 방법이 같기 때문!

**8.4 정규식 선택기로 권한을 부여할 요청 선택**

- 앤트와 MVC 식으로는 해결할 수 없는 특정한 요구사항이 있다면?
    - “경로에 특정 기호나 문자가 있으면 요청을 거부한다”와 같은 요구사항이 있다면?
- 정규식은 어떤 문자열 형식이든 나타낼 수 있으므로 무한한 가능성 제공
    - 단점 ) 간단한 시나리오를 적용하더라도 읽기 어려움
    - MVC나 앤트 선택기를 우선적으로 이용하고 다른 대안이 없는 경우에만 정규식을 이용하는 방법 권장
- regexMatchers(HttpMethod method, String regex)
    - 제한을 적용할 HTTP 방식과 경로를 참조할 정규식을 모두 지정
    - 같은 경로 그룹에 대해 HTTP 방식별로 다른 제한을 적용할 때 유용
- regexMatchers(String regex)
    - 경로만을 기준으로 권한 부여 제한을 적용
    - HTTP 방식에 자동으로 제한 적용

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {
	
	// 생략된 코드

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();

		http.authorizeRequests()
				.regexMatchers(".*/(us|uk|ca)+/(en|fr).*")
					.authenticated()
				.anyRequest()
					.hasAuthority("premium");
				// 사용자 인증을 위한 경로의 조건으로 정규식을 이용
				// 사용자가 프리미엄 액세스르 이용하는 데 필요한 다른 경로를 구성
				// 프리미엄 권한을 가지고 있으면 모든 요청이 가능
				
	}
}
```

- 정규식은 어떠한 요구 사항이라도 지정할 수 있는 강력한 툴!
- 하지만 읽기 어렵고 상당히 길어질 수 있으므로 최후의 수단으로 남겨두는 것을 권장!
