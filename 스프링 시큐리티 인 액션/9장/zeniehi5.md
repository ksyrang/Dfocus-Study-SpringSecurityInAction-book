# 9장. 필터 구현

- HTTP 필터
    - 책임의 *체인을 형성*
    - HTTP 요청에 적용해야 하는 각 책임을 관리
    - 요청을 수신, 논리를 실행, 최종적으로 체인의 *다음 필터에 요청을 위임*

## 9.1 스프링 시큐리티 아키텍처의 필터 구현

- 스프링 시큐리티 아키텍처의 필터는 일반적인 *HTTP 필터*
    - javax.servlet 패키지의 Filter 인터페이스를 구현
    - `doFilter()` 메서드를 재정의 → 논리를 구현
    - ServletRequest, ServletResponse, FilterChain을 매개변수로 받음
        - ServletRequest - HTTP 요청. *요청에 대한 세부 정보*를 얻을 수 있음
        - ServletResponse - HTTP 응답. 필터 체인에서 *응답을 변경* 가능
        - FilterChain - 체인의 *다음 필터로 요청을 전달*
- 필터 체인
    - 필터가 작동하는 *순서*가 정의된 필터의 모음
        - BasicAuthenticationFilter - HTTP Basic 인증을 처리
        - CsrfFilter - CSRF(사이트 간 요청 위조)를 처리
        - CorsFilter - CORS(교차 출저 리소스 공유) 권한 부여 규칙을 처리
    - 애플리케이션이 필터 체인에 모든 필터의 인스턴스를 가질 필요는 X
    - 필터 체인은 애플리케이션을 구성하는 방법에 따라 길이가 상이
    - (참고) 개발자가 작성하는 구성에 따라 필터 체인의 정의가 영향을 받음
        - 예시) Basic 인증 방식에서 HttpSecurity 클래스의 `httpBasic()` 메서드 호출 → 필터 체인에 BasicAuthenticationFilter가 추가됨
    - 새로운 필터는 필터 체인의 다른 필터를 기준으로 추가됨
        - 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가
        - 각 위치는 인덱스(숫자) → 곧 “순서”가 됨
        - 같은 위치에 두 개 이상의 필터를 추가 가능
            - 단, 스프링 시큐리티는 같은 위치에 있는 필터가 호출되는 순서를 보장 X

## 9.2 체인에서 기존 필터 앞에 필터 추가

- 필터 체인에서 기존 필터 앞에 맞춤형 HTTP 필터를 추가하는 과정
    - 필터 구현
        - Filter 인터페이스를 구현, `doFilter()` 메서드 재정의
        - 해당 메서드 안에 필터의 논리를 작성
        - 논리에 부합하면 `doFilter()` 메서드를 이용해 다음 필터로 요청을 전달
        - 부합하지 않으면 HTTP 상태 ‘400 잘못된 요청’ 반환
        
        ```java
        public class RequestValidationFilter implements Filter {
        																							// 필터 인터페이스 구현
        	@Override
        	public void doFilter(                       // 메서드 재정의
        		ServletRequest request,
        		ServletReponse response,
        		FilterChain filterChain)
        		throws IOException, ServletException {
        			...
        			...
        			if (...) {
        				httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
        				return;
        				// 조건에 부합하지 않으면 HTTP 상태가 '400 잘못된 요청'으로 바뀌고
        				// 요청이 필터 체인의 다음 필터로 전달 X
        			}
        
        			filterChain.doFilter(request, response);
        			// 요청에 부합하면 요청을 필터 체인의 다음 필터로 전달
        	}
        }
        ```
        
    - 필터 체인에 필터를 추가
        - 구성 클래스에서 `configure()` 메서드를 재정의, 필터 체인에 필터 추가
        - HttpSecurity 객체의 `addFilterBefore()` 메서드 이용
            - 필터 체인에 추가할 맞춤형 필터의 인스턴스와 새 인스턴스를 추가할 위치(기존 필터)를 매개변수로 받음
        
        ```java
        @Configuration
        public class ProjectConfig extends WebSecurityConfigurerAdapter {
        	@Override
        	protected void configure(HttpSecurity http) throws Exception {
        		http.addFilterBefore(
        					new RequestValidationFilter(),
        					BasicAuthenticationFilter.class)
        				.authorizeRequests()
        					.anyRequest().permitAll();
        	}
        }
        ```
        

## 9.3 체인에서 기존 필터 뒤에 필터 추가

- 필터 체인에서 기존 필터 뒤에 맞춤형 필터를 추가하는 과정
    - *간단한 로깅과 추적 목적*을 위해 특정한 인증 이벤트 이후 다른 시스템에 알림을 전달하는 사례
        
        ```java
        public class AuthenticationLoggingFilter implements Filter {
        																							// 필터 인터페이스 구현
        	@Override
        	public void doFilter(                       // 메서드 재정의
        		ServletRequest request,
        		ServletReponse response,
        		FilterChain filterChain)
        		throws IOException, ServletException {
        			...
        			...
        			logger.info("...");                     // 로그 기록 등의 작업 수행
        
        			filterChain.doFilter(request, response);
        			// 요청을 필터 체인의 다음 필터로 전달
        	}
        }
        ```
        
    - 기존 필터 다음에 맞춤형 필터 추가를 위해 HttpSecurity의 `addFilterAfter()` 메서드 호출
    
    ```java
    @Configuration
    public class ProjectConfig extends WebSecurityConfigurerAdapter {
    	@Override
    	protected void configure(HttpSecurity http) throws Exception {
    		http.addFilterBefore(
    					new RequestValidationFilter(),
    					BasicAuthenticationFilter.class)
    				**.addFilterAfter(
    					new AuthenticationLogginFilter(),
    					BasicAuthenticationFilter.class)**
    				.authorizeRequests()
    					.anyRequest().permitAll();
    	}
    }
    ```
    

## 9.4 필터 체인의 다른 필터 위치에 필터 추가

- 필터 체인에서 다른 필터 위치에 맞춤형 필터를 추가하는 과정
    - 스프링 시큐리티의 기존 필터가 수행하는 책임에 대해 다른 구현을 제공하고자 할 때 사용하는 접근법
    - 사용자 이름과 암호 대신 사용자를 인증하기 위한 작업 증명으로 적용 가능한 시나리오
        - 인증을 위한 정적 헤더 값에 기반을 둔 식별
            - HTTP 요청 헤더에 항상 동일한 문자열 하나를 앱으로 전달
            - 애플리케이션은 데이터베이스나 비밀 볼트에 이러한 값을 저장
        
        ![IMG_8C2469F18C5A-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/5ed0544d-aec9-4099-a4a3-d57bbb1b15cf/IMG_8C2469F18C5A-1.jpeg)
        
        - 대칭 키(또는 비대칭 키)를 이용해 인증 요청 서명
            - 클라이언트와 서버가 키를 공유
            - 클라이언트는 이 키로 요청의 일부에 서명 - 특정 헤더 값에 서명
            - 서버는 같은 키로 서명이 유효한지 검증 - 각 클라이언트의 개별 키를 저장
            
            ![IMG_6D0C51CCCB3F-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/a9295d64-a57d-4e20-9e3c-4a6ece52cd7f/IMG_6D0C51CCCB3F-1.jpeg)
            
        - 인증 프로세스에 OTP(일회용 암호) 이용
            - 사용자는 문자 메시지 또는 Google Authenticator 같은 인증 공급자 앱으로 OTP 발급
            
            ![IMG_3578D6C5F93D-1.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/1790de9f-d75e-40e3-a0f6-903a32336120/IMG_3578D6C5F93D-1.jpeg)
            

```java
@Component
public class StaticKeyAuthenticationFilter implements Filter {
																							// 필터 인터페이스 구현
	@Value("${authorization.key}")
	private String authorizationKey;

	@Override
	public void doFilter(                       // 메서드 재정의
		ServletRequest request,
		ServletReponse response,
		FilterChain filterChain)
		throws IOException, ServletException {
			...
			...
			if (authozationKey.equals(authentication)) {
				filterChain.doFilter(request, response);
			} else {
				httpResponse.setStatue(HttpServletResponse.SC_UNAUTHORIZED);
			
	}
}
```

```java
@Configuration
public class ProjectConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	private StaticKeyAuthenticationFilter filter;
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.**addFilterAt(filter
					BasicAuthenticationFilter.class)**
				.authorizeRequests()
					.anyRequest().permitAll();
	}
}
```

- 주의 사항
    - *특정 위치에 필터를 추가해도 기존 필터는 대체되지 X*
        - 필터 체인의 같은 위치에 필터를 추가 가능
        - 단, 필터가 실행되는 순서를 보장할 수 없음
            - 같은 위치에 여러 필터를 추가하는 것은 권장되지 X
            - 필터 체인에 필요 없는 필터는 아예 추가하지 말자!

## 9.5 스프링 시큐리티가 제공하는 필터 구현

- 스프링 시큐리티에 기본적으로 존재하는 Filter 인터페이스를 구현한 추상 클래스
    - 필터 정의를 확장해 유용한 기능을 추가 가능
        - GenericFilterBean
            - web.xml 파일에 정의하여 초기화 매개 변수 이용 가능
        - OncePerRequestFilter
            - GenericFilterBean을 확장
            - 필터 체인에 추가한 필터가 요청 당 한 번만 실행되도록 보장
            - `doFilter()` 메서드가 요청 당 한 번만 실행되도록 논리 구현

```java
public class AuthenticationLoggingFilter extends OncePerRequestFilter {
																							// OncePerRequestFilter 클래스 확장
	@Override
	public void doFilterInternal(                       // 메서드 재정의
		HttpServletRequest request,                       // OncePerRequestFilter는
		HttpServletReponse response,                      // HTTP 필터만 지원
		FilterChain filterChain)
		throws IOException, ServletException {
			...
			...
			logger.info("...");                     // 로그 기록 등의 작업 수행

			filterChain.doFilter(request, response);
			// 요청을 필터 체인의 다음 필터로 전달
	}
}
```

- OncePerRequestFilter 클래스에 관해 알아둘 사항
    - HTTP 요청만 지원
        - HttpServletRequest, HttpServletResponse로 직접 요청을 수신
        - 형 변환할 필요가 없음
    - 필터가 적용될지 결정하는 논리 구현 가능
        - 필터 체인에 추가한 필터가 특정 요청에는 적용되지 않는다고 결정 가능
        - `shouldNotFilter(HttpServletRequest)` 메서드 재정의
    - 비동기 요청이나 오류 발송 요청에는 적용 X
        - 이 동작을 변경하고자 하면 `shouldNotFilterAsyncDispatch()` 및 `shouldNotFilterErrorDispatch()` 메서드 재정의

## 요약

- 웹 애플리케이션의 첫 번째 계층은 HTTP 요청을 가로채는 필터 체인
- 필터 체인에서 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가 → 필터 체인 맞춤 구성이 가능
- 기존 필터와 같은 위치에 여러 필터 추가 가능
    - 필터의 실행 순서는 보장 X
- 필터 체인을 변경하여 애플리케이션의 요구 사항에 맞게 인증과 권한 부여를 맞춤 구성 가능
