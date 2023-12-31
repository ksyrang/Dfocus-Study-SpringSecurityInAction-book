# 4장. 암호 처리

---

📌 스프링 시큐리티로 구현한 애플리케이셔에서 암호와 비밀을 관리하는 방법

### 4.1 PasswordEncoder 계약의 이해

![IMG_77E70EE68DC6-1.jpeg](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20f4eb02c7f68d40069e339a5002ac2f9a/IMG_77E70EE68DC6-1.jpeg)

**4.1.1 PasswordEncoder 계약의 정의**

- PasswordEncoder
    - 인증 프로세스에서 암호가 유효한지를 검증
    - 계약에 선언된 encode() 및 matches() 메서드는 계약의 책임을 정의
        - encode() → 주어진 문자열을 변환해 반환
            - 주어진 암호의 해시를 제공하거나 암호화를 수행
        - matches() → 인코딩된 문자열이 원시 암호와 일치하는지 나중에 확인
            - 지정된 암호를 인증 프로세스에서 알려진 자격 증명의 집합을 대상으로 비교
    - upgradeEncoding
        - 메서드 재정의 시 인코딩된 암호를 보안 향상을 위해 다시 인코딩

**4.1.2 PasswordEncoder 계약의 구현**

**4.1.3 PasswordEncoder의 제공된 구현 선택**

- NoOpPasswordEncoder
    - 암호를 인코딩하지 않고 일반 텍스트로 유지
    - 싱글톤 설계 → getInstance() 메서드로 클래스의 인스턴스를 얻음
- StandardPasswordEncoder
    - SHA-256을 이용해 암호를 해시
    - 구식 → 새 구현에는 쓰지 말아야!
        - 강도가 약한 해싱 알고리즘을 사용하기 때문
    - 비밀키 값을 매개 변수로 전달 → 해싱 프로세스에 적용할 비밀키 지정 가능
    - 파라미터 없는 기본 생성자로 생성할 경우 빈 문자열이 키 값으로 이용
- Pbkdf2PasswordEncoder
    - PBKDF2를 이용
        - 반복 횟수 인수만큼 HMAC를 수행하는 단순하고 느린 해싱 함수
        - 생성 시 매개 변수 → 인코딩 키, 인코딩의 반복 횟수, 해시의 크기
            - 해시가 길수록 암호는 강력
            - 반복 횟수를 늘리면 애플리케이션이 소비하는 리소스가 증가 → 성능에 영향
- BCryptPasswordEncoder
    - bcrypt 강력 해싱 함수로 암호 인코딩
    - 매개변수 → 인코딩 프로세스에 이용되는 로그 라운드를 나타내는 강도 계수 지정 가능, 인코딩에 이용되는 SecureRandom 변경 가능
        - 지정하는 로그 라운드 값은 해싱 작업이 이용하는 반복 횟수에 영향
- SCryptPasswordEncoder
    - scrypt 해싱 함수로 암호 인코딩
        - 매개 변수 5개 → CPU 비용, 메모리 비용, 병렬화 계수, 키 길이, 솔트 길이

**4.1.4 DelegatingPasswordEncoder를 이용한 여러 인코딩 전략**

- 다양한 암호 인코더를 갖추고 특정 구성에 따라 선택하는 방식이 유용한 상황
    - 예) 특정 애플리케이션 버전부터 인코딩 알고리즘이 변경된 경우
        - 여러 종류의 해시를 지원해야 함
- DelegatingPasswordEncoder

![IMG_0011.jpeg](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20f4eb02c7f68d40069e339a5002ac2f9a/IMG_0011.jpeg)

- 작업을 위임하는 PasswordEncoder 구현의 목록을 갖고 각 인스턴스를 맵에 저장
    - 암호 앞에 접두사 {OOOO}를 확인해 해당 접두사와 일치하는 구현체에 작업을 위임
        - 해당 접두사는 인코더 맵에서 이용할 암호 인코더를 식별하는 키가 들어있음
            - 중괄호는 필수!
    - 스프링 시큐리티는 편의를 위해 표준 제공 PasswordEncoder 구현체 맵을 가진 DelegatingPasswordEncoder 생성하는 방법을 제공 → createDelegatingPasswordEncoder()
        - bcrypt가 기본 인코더인 구현을 반환

📌 기본 키워드

- 인코딩
    - 주어진 입력에 대한 모든 변환
- 암호화
    - 출력을 얻기 위해 입력 값과 키를 모두 지정하는 특정한 유형의 인코딩
        - 키를 이용해 출력에서 입력을 얻는 것 → 역함수 복호화(Reverse Function Decryption)
        - 암호화에 쓰는 키와 복호화에 쓰는 키가 같다 → 대칭 키
        - 암호화에 쓰는 키와 복호화에 쓰는 키가 다르다 → 비대칭 키(Asymmetric Key)
            - 암호화 키 ⇒ 공개 키(Public Key)
            - 복호화 키 ⇒ 개인 키(Private Key)
- 해싱(Hashing)
    - 함수가 한 방향으로만 작동하는 특정한 유형의 인코딩
    - 즉, 출력 y에서 입력 x를 얻을 수 없다
        - 출력 y가 입력 x에 해당하는지 확인해야 하므로 해시는 인코딩과 일치를 위한 한 쌍의 함수로 볼 수 있음
        - 해싱 함수가 x → y라면 일치 함수 (x, y) → boolean도 존재
    - 해싱 함수는 때때로 입력에 임의의 값을 추가 가능 → 솔트(Salt)
        - 함수를 더 강하게 만들어 결과에서 입력을 얻는 역함수의 적용 난도를 높임

### 4.2 스프링 시큐리티 암호화 모듈에 관한 추가 정보

- 스프링 시큐리티는 프로젝트의 종속성을 줄이기 위해 자체 솔루션을 제공 → SSCM
    - 키 생성기
        - 해싱 및 암호화 알고리즘을 위한 키를 생성하는 객체
    - 암호기
        - 데이터를 암호화 및 복호화하는 객체

**4.2.1 키 생성기 이용**

- 키 생성기
    - 특정한 종류의 키를 생성하는 객체
        - 암호화나 해싱 알고리즘에 필요
    - 팩토리 클래스 KeyGenerators → 키 생성기를 직접 만들 수 있음
        - BytesKeyGenerator
            - generateKey() 메서드
                - byte[] 키를 반환
            - getKeyLength() 메서드
                - 키 길이(바이트 수)를 반환
            - 키 생성기 인스턴스를 얻을 때 KeyGenerators.secureRandom() 메서드 호출
                - 매개변수로 원하는 키 길이를 전달
            - 같은 키 생성기를 호출하면 같은 키 값을 반환하는 구현
                - KeyGenerators.shared(int length) 메서드
        - StringKeyGenerator
            - 해싱 또는 암호화 알고리즘의 솔트 값으로 이용
            - 키 값을 나타내는 문자열을 반환하는 generateKey() 메서드
                - 8바이트 키 생성
                - 16진수 문자열로 인코딩

**4.2.2 암호화와 복호화 작업에 암호기 이용**

- 암호기
    - 암호화 알고리즘을 구현하는 객체
    - SSCM에는 BytesEncryptor 및 TextEncryptor가 정의
    - TextEncryptor
        - 데이터를 문자열로 관리(문자열을 입력 받고 문자열을 출력)
        - 세 가지 주요 형식
            - Encryptors.text()
                - Encryptors.standard() 메서드로 암호화 작업 관리
            - Encryptors.delux()
                - Encryptors.stronger() 인스턴스 이용
            - Encryptors.queryableText()
                - 순차 암호화 작업에서 입력이 같으면 같은 출력을 생성하도록 보장
        - Encryptors.text() 와 Encryptors.delux()
            - 같은 입력으로 encrypt() 메서드 반복 호출해도 다른 출력을 반환
                - 암호화 프로세스에 임의의 초기화 벡터가 생성되기 때문
    - BytesEncryptor
        - 보다 범용적
        - 바이트 배열로 입력 데이터를 받는다
    - 팩토리 클래스 Encryptors
        - Encryptors.standard() 또는 Encryptors.stronger()
            - BytesEncryptor 반환
            - 내부적으로 표준 바이트 암호기는 256 바이트 AES 암호화 사용
            - 더 강력한 암호기 인스턴스를 만들려면 → Encryptors.stronger()
                - 256 바이트 AES 암호화 작업 모드로 GCM(갈루아/카운터 모드) 이용
                - 표준 모드일 경우 CBC(암호 블록 체인) 이용
