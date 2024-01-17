# 02부 9장 필터 구현

생성자: 이루리
생성 일시: January 16, 2024 4:14 PM

### 이 단원의 내용

- 필터 체인 이용하기
- 맞춤형 필터 정의
- Filter 인터페이스를 구현하는 스프링 시큐리티 클래스 이용

![Untitled](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/Untitled.png)

- 스프링 시큐리티는 필터체인을 원하는 방식으로 모델링 할 수 있는 유연성을 제공한다.
    - 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가해 필터 체인을 맞춤 구성할 수 있다. 이 방식으로 인증은 물론 요청과 응답에 적용되는 프로세스를 맞춤 구성할 수 있다.

### 9.1 스프링 시큐리티 아키텍처의 필터 구현

- 스프링 시큐리티 아키텍처 필터는 일반적인 HTTP  필터
- doFilter() 메서드를 재정의해 논리를 구현
    - ServletRequest, ServletResponse,FilterChain 매개변수를 받음
    
    | ServletRequest | HTTP 요청을 나타냄. ServletRequest객체를 이용해 요청에 대한 세부 정보를 얻음 |
    | --- | --- |
    | ServletResponse | HTTP 응답을 나타냄. ServletResponse 객체를 이용해 응답을 클라이언트로 다시 보내기 전에 더 나아가 필터체인에서 응답을 변경 |
    | FilterChain | 필터 체인을 나타낸다. FilterChain 객체는 체인의 다음 필터로 요청을 전달 |
- 필터 체인 → 필터가 작동하는 순서가 정의된 필터의 모음을 나타냄
    
    
    | BasicAuthenticationFilter | HTTP Basic 인증 처리 |
    | --- | --- |
    | CsrfFilter | CSRF(사이트 간 요청 위조) 처리 |
    | CorsFilter | CORS(교차 출처 리소스 공유) 권한 부여 규칙 처리 |
- HTTP Basic 인증 방식 이용하려면 HttpSecurity 클래스의 httpBasic() 메서드 호출 해야 하는데 이 때 필터 체인BasicAuthenticationFilter가 추가된다.
- 이와 비슷하게 개발자가 작성하는 구성에 따라 필터 체인의 정의가 영향을 받음
    
    ![스크린샷 2024-01-16 오후 11.08.33.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.08.33.png)
    
- 새 필터는 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가할 수 잇다. 각 위치는 인덱스이며 순서라고도 할 수 있음.
- 같은 위치에 필터를 두 개 이상 추가할 수 있는데 여러 필터가 같은 위치에 있으면 필터가 호출되는 순서는 정해지지 않는다.

### 9.2 체인에서 기존 필터 앞에 필터 추가

- 기존 필터 앞에 맞춤형 HTTP 필터를 추가하는 과정
- RequestValidationFilter 클래스 : 요청에 필요한 헤더가 있는지 확인
    - configure() 메서드 재정의해 필터 체인에 필터 추가
    
    ```java
    public class RequestValidationFilter implements Filter{
      @Override
    	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
    											 FitlerChain filterChain) throws IOException, ServletException{
    		// 메서드내에 필터 논리 정의
    			var httpRequest = (HttpServletRequest) request ;
    			var httpResponse = (HttpServletResponse) responsej
    			String requestId = httpRequest.getHeader ("Request—Id");
    			if (requestld == null || requestId.isBlank()) {
    			httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
    	 
    			return; // 헤더가 없으면 HTTP 상태가 '400 잘못된 요청'으로 바뀌고 요청이 필터 체인의 다음 필터로 전달되지 않음
    		}
    
    		filterChain.doFilter(request,response);
    		// 헤더가 있으면 요청을 필터 체인의 다음 필터로 전달
    	}
    }
    ```
    
    - Request-id 헤더 있는지 확인 후 헤더가 있으면 doFilter 메서드 호출→ 체인의 다음필터로 요청 전달
- HttpSecurity객체의 addFilterBefore() 메서드
    - 매개변수 두개를 받는다.
        
        1) 필터체인에 추가할 맞춤형 필터의 인스턴스
        
        2) 새 인스턴스를 추가할 위치(기존 필터)
        
        ![스크린샷 2024-01-16 오후 11.23.16.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.23.16.png)
        
    

### 9.3 체인에서 기존 필터 뒤에 필터 추가

- 인증  후 요청을 기록하도록 BasicAuthenticationFilter 뒤에 AuthenticationLogginFilter를 추가
    
    ![스크린샷 2024-01-16 오후 11.25.07.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.25.07.png)
    
- AuthenticationLogginFilter 정의 후 configure() 메서드에 addFilterAfter() 메서드 호출
    
    ![스크린샷 2024-01-16 오후 11.26.18.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.26.18.png)
    

### 9.4 필터 체인의 다른 필터 위치에 필터 추가

- 정적 키 값을 읽고 Authorization 헤더 값과 같은지 확인
    
    ![스크린샷 2024-01-16 오후 11.30.47.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.30.47.png)
    
    - 필터 정의 후 addFilterAt() 메서드 이용해 필터 체인의 BasicAuthenticationFilter클래스의 위치에 추가
    - 특정 위치에 필터 추가해도 스프링 시큐리티는 이 위치에 있는 필터가 하나라고 가정하지 않음.
    - 실행되는 순서도 보장하지 않는다. 즉, 필터 체인에 필요 없는 필터는 아예 추가하지 않아야함.

### 9.5 스프링 시큐리티가 제공하는 필터 구현

- GenericFilterBean 클래스 확장 시 필요할 때 web.xml 파일에 정의하여 초기화 매개변수 이용 가능
- OncePerRequestFilter는 GenericFilterBean을 확장하는 더 유용한 클래스 ⇒ doFilter() 메서드가 요청당 한 번만 실행되도록 논리 구현
    - ex) 로깅 기능
        
        ![스크린샷 2024-01-16 오후 11.36.11.png](02%E1%84%87%E1%85%AE%209%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%90%E1%85%A5%20%E1%84%80%E1%85%AE%E1%84%92%E1%85%A7%E1%86%AB%206b00e202df7d41309335cfa10ae37a40/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-01-16_%25EC%2598%25A4%25ED%259B%2584_11.36.11.png)
        
    - 유의 사항
        
        1) HTTP 요청만 지원하지만 사실은 항상 이것만 이용한다.
        
        : 이 클래스의 장점은 형식을 형 변환하며 HttpServletRequest 및 HttpServletResponse로 직접 요청을 수신한다는 것. Filter 인터페이스의 경우에는 요청과 응답을 형변환 해야한다?
        
        2) 필터가 적용될지 결정하는 논리 구현 가능
        
        : 필터 체인에 추가한 필터가 특정 요청에는 적용되지 않는다고 결정할 수 있음. shoudNotFilter(HttpServletRequest)메서드 재정의
        
        3) OncePerRequestFilter는 기본적으로 비동기 요청이나 오류 발송 요청에는 적용되지 않는다.
        
        : 이 동작 변경하려면 shodNotFilterAsyncDispatch() 및 shouldNotFilterErrorDispatch() 메서드 재정의
        

### 요약

- 웹 애플리케이션의 첫번째 계층은 HTTP 요청을 가로채는 필터체인이다. 스프링 시큐리티 아키텍처의 다른 구성 요소는 요구사항에 맞게 맞춤 구성할 수 있다.
- 필터 체인에서 기존 필터 위치 또는 앞이나 뒤에 새 필터를 추가해 필터체인을 맞춤 구성할 수 있다.
- 기존 필터와 같은 위치에 여러 필터를 추가할 수 있으며, 이 경우 필터가 실행되는 순서는 정해지지 않는다.
- 필터 체인을 변경하면 애플리케이션의 요구사항에 맞게 인증과 권한 부여를 맞춤 구성하는 데 도움이 된다.