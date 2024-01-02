# 02부 4장. 암호 처리

생성자: 이루리
생성 일시: January 1, 2024 4:28 PM

# 암호 처리

- PasswordEncoder의 구현 및 이용
- 스프링 시큐리티 암호화 모듈에 있는 툴 이용

## 4.1 PasswordEncoder 계약의 이해

- PasswordEncoder가 인증 프로세스의 어떤 부분을 담당하는가?
    - AuthenticationProvider 인증 공급자는 인증 프로세스에서 사용자를 찾은 후 PasswordEncoder를 이용해 암호를 검증한다.

### 4.1.1. PasswordEncoder 계약의 정의

- PasswordEncoder는 인증 프로세스에서 암호가 유효한지를 확인한다.
- PasswordEncoder 도 암호를 인코딩할 수 있다.
    - ecode() , matches()
- PasswordEncoder 인터페이스
    
    ```java
    public interface PasswordEncoder {
        String encode(CharSequence var1);
    
        boolean matches(CharSequence var1, String var2);
    
        default boolean upgradeEncoding(String encodedPassword) {
            return false;
        }
    }
    ```
    
    - 두 개의 추상 메서드와 기본 구현이 있는 메서드 하나를 정의
    - encode(CharSequence var1) 메서드는 주어진 문자열을 변환해서 반환
        - 주어진 암호의 해시를 제공하거나 암호화를 수행
    - matches(CharSequence var1, String var2) 메서드는 인코딩된 문자열이 원시 암호와 일치하는지 확인
    - upgradeEncoding(String encodedPassword) 메서드는 기본값 false 반환
        - true를 반환하도록 메서드를 재정의하면 인코딩된 암호를 보안 향상을 위해 다시 인코딩.

### 4.1.2 PasswordEncoder 계약의 구현

- encode() 메서드에서 반환된 문자열은 항상 같은 PasswordEncoder의 match() 메서드로 검증할 수 있어야함.
- PasswordEncoder를 구현할 수 있으면 앱의 인증 프로세스에서 암호를 관리하는 방법을 선택할 수 있음.
- 해싱 알고리즘 SHA-512 이용하는 PasswordEncoder 구현
    
    ```java
    public class Sha512PasswordEncoder implements PasswordEncoder{
    	@Override
    	public String encode(CharSequence rawPassword){
    		return hashWithSHA512(rawPassword.toString());
    	}
    
    	@Override
    	public boolean matches(CharSequence rawPassword, String encodedPassword){
    		String hashedPassword = encode(rawPassword);
    		return encodedPassword.equals(hashedPAssword);
    	}
    }
    ```
    
    - encode() 메서드는 password를 hash,
    - matches() 메서드는 입력된 원시 암호를 해시하고, 주어진 해시와 비교해 검증
- 입력을 SHA-512 로 해시하는 메서드 구현

### 4.1.3 PasswordEncoder의 제공된 구현 선택

- 스프링 시큐리티의 PasswordEncoder 구현 옵션
    - NoOpPasswordEncoder
        
        : 암호를 인코딩하지 않고 일반 텍스트로 유지, 실제 시나리오에서는 절대 쓰지 말아야함
        
        - 싱글톤으로 설계돼서 클래스 바깥에서 생성자를 직접 호출할 수 없음.
        - NoOpPasswordEncoder.getInstance() 메서드로 클래스의 인스턴스 얻을 수 있음.
    - StandardPasswordEncoder
        
        :SHA-256을 이용해 암호를 해시. 이 구현은 구식! 새 구현에서는 쓰지 말아야함. 
        
        - 해싱 프로세스에 적용할 비밀을 지정할 수 있다?
        - 비밀의 값은 생성자의 매개변수로 전달. 인수가 없는 생성자 호출시 빈문자열이 키값이 됨
    - Pbkdf2PasswordEncoder
        
        : PBKDF2f를이용
        
        - 반복횟수 인수만큼 HMAC를 수행하는 아주 단순하고 느린 해싱 함수
        - 마지막 호출의 세 매개변수는 각각 인코딩 프로세스에 이용되는 키의 값, 암호 인코딩의 반복 횟수, 해시의 크기
        - 해시가 길수록 암호는 더 강력해짐.
        - BUT . 성능에 영향을 주며 반복 횟수를 늘리면 애플리케이션이 소비하는 리소스 증가
        - 해시 생성에 사용되는 리소스와 필요한 인코딩 강도 사이에서 절충해야함
        - 기본값 18500,256
        
        ```java
        PasswordEncdoer p = new Pbkdf2PasswordEncoder();
        PasswordEncdoer p = new Pbkdf2PasswordEncoder("secret");
        PasswordEncdoer p = new Pbkdf2PasswordEncoder("secret",185000,256);
        ```
        
    - BCryptPasswordEncoder
        
        : bycrpt 강력 해싱 함수로 암호를 인코딩
        
        - 인스턴스를 만들려면 인수가 없는 생성자를 호출해도 됨
        - 인코딩 프로세스에 이용되는 로그 라운드를 나타내는 강도 계수를 지정할 수도 있음
        - 이유
            
            보안적인 측면에서 로그 라운드를 높이면, 비밀번호를 해싱하는 데 필요한 계산 작업이 더 많아져서 무차별 대입 공격(예: 무차별 대입 공격 또는 무차별 대입 공격을 통한 비밀번호 크래킹)에 대한 강력한 방어를 제공합니다. 따라서 로그 라운드를 높이면 해시 함수의 강도가 높아져서 공격자가 비밀번호를 찾는 데 더 많은 시간이 걸리게 됩니다.
            
        - 인코딩에 이용되는 SecureRandom 인스턴스도 변경 가능
    - ScryptPasswordEncoder
        
        : script 해싱 함수로 암호를 인코딩
        
        - 다섯개의 매개변수를 받아 CPU 비용, 메모리 비용, 키 길이, 병렬화 개수, 솔트 길이를 구성할수 있게 해줌.

### 4.1.4 DelegatingPasswordEncoder를 이용한 여러 인코딩 전략

- 인증흐름에 암호 일치를 위해 다양한 구현을 적용해야 할 때 설명
- 자체 구현이 없고 PasswordEncdoer 인터페이스를 구현하는 다른 객체에 위임함
- 운영 단계에서 DelegatingPasswordEncoder 가 사용되는 일반적인 시나리오는 특정 애플리케이션 버전부터 인코딩 알고리즘이 변경된 경우
- DelegatingPasswordEncoder
    1. DelegatingPasswordEncoder는 각 인스턴스를 맵에 저장.
    2. NoOpPasswordEncoder에는 키 noop 할당 / BCryptPasswordEncoder 구현에는 키 bcrypt가 할당 각각 접두사 {noop}, {bcrypt}
- 해시의 접두사를 기준으로 암호를 비교하기 위해 올바른 PasswordEncoder 구현을 선택
- {접두사} 는 인코더 맵에서 이용할 암호 인코더를 식별하는 키

## 4.2 스프링 시큐리티 암호화 모듈에 관한 추가 정보

- SSCM의 두가지 필수 기능
    - 키 생성기 : 해싱 및 암호화 알고리즘을 위한 키를 생성하는 객체
    - 암호기 : 데이터를 암호화 및 복호화하는 객체

### 4.2.1 키 생성기 이용

- 키 생성기
    
    : 특정한 종류의 키를 생성하는 객체로서 일반적으로 암호화나 해싱 알고리즘에 필요. 
    
    - BytesKeyGenerator
    - StringKeyGenerator
        - 문자열 키 생성기를 이용해 문자열 키를 얻을 수 있는데 이 키는 해싱 또는 암호화 알고리즘의 솔트값으로 이용된다.
        - 솔트
            
            암호화 알고리즘에서 "솔트(salt)"는 매번 다른 값을 사용하여 암호화를 수행하는 데 사용되는 임의의 데이터입니다. 솔트는 주로 비밀번호 해싱에서 사용되며, 암호화된 해시 값을 만들 때 사용자의 비밀번호와 결합됩니다.
            

### 4.2.2. 암호화 복호화 작업에 암호기 이용

- 암호기
    
    : 암호화 알고리즘을 구현하는 객체
    
    : 암호화, 복호화 작업을 지원
    
    - 두 유형의 암호기 → 다른 데이터 형식을 처리
    - BytesEncryptor
        - 더 범용적이며 바이트 배열로 입력을 받음
        
        ```java
        String salt = KeyGenerators.string().generateKey();
        String password = "secret";
        String valueToEncrypt ="Hello";
        // 내부적으로 표준 바이트 암호기는 256 바이트 AES 암호화를 이용해 입력 암호화
        BytesEncryptor e = Encryptors.standard(password,salt);
        byte[] encrypted = e.encrypt(valueToEncrypt.getBytes());
        byte[] decrypted = e.decrypt(encrypted);
        
        // 더 강력한 바이트 암호기 인스턴스를 만들려면 Encryptors.stronger() 메서드 호출
        // BytesEncryptor e = Encryptors.stronger(password,salt);
        
        ```
        
    - TextEncryptor
        - 데이터를 문자열로 관리
        - 문자열을 입력으로 받고 문자열을 출력으로 반환
        - 세가지 주요 형식과 더미 TextEncyptor 반환 메서드
            - Encryptors.text()
                
                : Encryptors.standard() 메서드로 암호화 작업 관리
                
            - Encryptors.delux()
                
                : Encryptors.stroger() 인스턴스 이용
                
            
            → Encryptors.text(), Encryptors.delux()는 같은 입력으로 encrypt() 메서드를 반복 호출해도 다른 출력이 반환된다. 그 이유는 암호화 프로세스에 임의의 초기화 벡터가 생성되기 때문
            
            - Encryptors.queryableText()
                
                : 순차 암호화 작업에서 입력이 같으면 같은 출력을 생성하도록 보장
                
            - Encryptors.noOptext() → 암호화 시간 소비하지 않고 앱의 성능을 테스트하기를 원할때
        

## 요약

- PasswordEncoder는 인증 논리에서 암호를 처리하는 가장 중요한 책임 담당
- 스프링 시큐리티는 해싱 알고리즘 여러 대안을 제공. 필요한 구현을 선택.
- 스프링 시큐리티 암호화모듈(SSCM) 에는 키 생성기와 암호기를 구현하는 여러 대안이 있음
- 키 생성기는 암호화 알고리즘에 이용되는 키를 생성하도록 도와주는 유틸리티 객체
- 암호기는 데이터 암호화와 복호화를 수행하도록 도와주는 유틸리티 객체