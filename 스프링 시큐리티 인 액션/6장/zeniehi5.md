# 6장. 실전: 작고 안전한 웹 애플리케이션

### 6.1 프로젝트 요구 사항과 설정

- 사용자가 인증에 성공하면 주 페이지에서 제품 목록을 볼 수 있는 웹 애플리케이션 구현
    - 애플리케이션의 정보와 사용자는 데이터베이스에 저장
    - 각 사용자의 암호는 bcrypt나 scrypt로 해시

![JPEG image-4B43-9B19-76-0.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/82c76c9c-91ab-4fab-a183-126246ae45b2/e19ce810-7af7-4cb0-a9f4-f7c0f226426b/JPEG_image-4B43-9B19-76-0.jpeg)

- 프로젝트를 구현하는 주요 단계
    - 데이터베이스 설정
        - schema.sql
            - 데이터베이스의 구조를 만들고 변경하는 모든 쿼리를 포함
        - data.sql
            - 데이터를 처리하는 모든 쿼리를 저장
    - 사용자 관리 정의
    - 인증 논리 구현
    - 주 페이지 구현
    - 애플리케이션 실행 및 테스트

### 6.2 사용자 관리 구현

- 스프링 시큐리티에서 사용자 관리를 담당하는 구성 요소
    - UserDetailsService
        - 두 해싱 알고리즘을 위한 암호 인코더 객체 정의
        - 인증 프로세스에 필요한 세부 정보를 저장하는 user 및 authority 테이블을 나타낼  JPA 엔티티 정의
        - JpaRepository 인터페이스 정의
        - User  JPA 엔티티에 대해 UserDetails 인터페이스를 구현하는 데코레이터 만들기 → 책임 분릴
        - UserDetailsService 구현체 구현 → JpaUserDetailsService

### 6.3 맞춤형 인증 논리 구현

- AuthenticationProvider 구현 → 스프링 시큐리티 인증 아키텍처에 등록
    - 종속성으로 UserDetailsService 구현과 두 개의 암호 인코더가 필요
    - authenticate() 및 support() 메서드 재정의
        - 인증 구현 형식을 UsernamePasswordAuthenticationToken으로 지정

### 6.4 주 페이지 구현

- product 테이블의 모든 레코드를 표시하는 간단한 페이지
    - 사용자가 로그인해야 볼 수 있다

### 6.5 애플리케이션 실행 및 테스트
