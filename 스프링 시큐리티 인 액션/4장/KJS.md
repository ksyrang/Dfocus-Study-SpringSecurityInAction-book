# 4장 암호 처리

## 이 단원의 내용

- PasswordEncoder의 구현 및 이용
- 스프링 시큐리티 암호화 모듈에 있는 Tool 이용

## 시작에 앞서…

- 스프링 시큐리티로 구현한 애플리케이션에서 암호와 비밀을 관리하는 방법을 배운다.
- PasswordEncoder 계약을 살펴보고 암호 관리를 위한 스프링 시큐리티 암호화 모듈(Spring Security Crypto module; SSCM)의 Tool을 소개한다.

## 4.1 PasswordEncoder 계약의 이해

- PasswordEncoder의 인증 프로세스
    
    ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled.png)
    

### 4.1.1 PasswordEncoder 계약의 정의

- 인터페이스 정의
    
    ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%201.png)
    
- PasswordEncoder는 암호가 유효한지를 확인한다.
- 모든 시스템은 어떤 방식으로든 인코딩된 암호를 저장하며 아무도 암호를 읽을 수 없게 해시를 저장하는 것이 좋다.
- PasswordEncoder는 암호를 인코딩 할 수 있다.
- PasswordEncoder는 두개의 추상 메서드와 기본 구현된 메서드 하나를 정의하고 있다.
    - 추상 : encode(), matches()
        - encode : 주어진 평문 문자열을 변환 후 반환한다.
            - 스프링 시큐리티 기능의 관점에서 encode()는 주어진 암호의 해시를 제공하거나 암호화를 수행한다.
        - matches : 인코딩 된 문자열이 원시암호와 일치하는지 확인하는 메소드이다.
            - 평문 문자열과 암호화된 비교 대상을 매개변수로 한다.
            - 반환하는 값은 Boolean형이다.
    - 기본 구현 : upgradeEncoding

### 4.1.2 PasswordEncoder 계약의 구현

- 단순한 구현 예시
    
    ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%202.png)
    
- encode(), matches()는 밀접한 관계가 있다.
- 이들 메서드를 재정의하려면 기능 면에서 항상 일치해야 한다.
- 즉 encode에서 반환된 암호화 문자열은 matches로 검증할 수 있어야 한다.

### 4.1.3 PasswordEncoder의 제공된 구현 선택

- 스프링 시큐리티에서 이미 구현한 유용한 메서드들이 있다.
    - NoOpPasswordEncoder : 암호를 인코딩하지 않고 일반 문자열로 유지한다. 실제로는 사용해서는 안된다!
        - 싱글톤 설계로 클래스 밖에서는 인스턴스 호출로 인스턴스를 얻을 수 있다.
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%203.png)
        
    - StandardPasswordEncoder : SHA-256을 이용해 암호화 한다. 현재는 구식이어서 사용하면 안되지만 일부 SHA-256으로 구현된 환경에서 이용되는 경우가 있다.
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%204.png)
        
    - Pbkdf2PasswordEncoder : PBKDF2 방식을 이용하는 암호화
        - PBKDF2 : 반복 횟수 인수만큼 HMAC를 수행하는 아주 단순한 느린 해싱 함수.
        - 키의 값, 암호 인코딩 반복 횟수, 해시 크기 순으로 매개변수를 가진다.
        - 뒤의 암호 인코딩 반복 횟수와 해시 크기를 입력 하지 않으면 기본적으로 횟수는 18500을 해시 크기는 256이 입력된다.
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%205.png)
        
    - BCrytPasswordEncoder : bcrypt 강력 해싱 함수로 암호화
        - 로그 아운드라는 강도 계수를 지정할 수 있다(4~31)
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%206.png)
        
    - SCrytPasswordEncoder : scrypt 해싱 함수로 암호화
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%207.png)
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%208.png)
        

### 4.1.4 DelegatingPasswordEncoder를 이용한 여러 인코딩 전

- 인증 흐름에 암호 일치를 위한 다양한 구현을 적용해야 할 때를 설명.
- PasswordEncoder로 작동하는 Tool 적용 방법을 배운다.
    - Tool 자체 구현은 없고 다른 객체에 위임한다.
- 인코딩 알고리즘 변경된 경우 기존과 신규, 즉 여러 종류의 해시를 지원해야 할 때 좋은 선택이다.
- DelegatingPasswordEncoder는 PasswordEncoder의 인터페이스의 한 구현이며 자체 인코딩 알고리즘을 구현하는 대신 같은 계약의 다른 구현 인스턴스에 작업을 위임한다.
- 암호의 접두사를 기준으로 올바른 PasswordEncoder 구현에 작업을 위임한다.
- 작업 위임 플로어
    - NoOpPasswordEncoder일 때
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%209.png)
        
    - BCtyptPasswordEncoder일 때
        
        ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%2010.png)
        
- DelegatingPasswordEncoder 정의 방법
    - 해시를 정의할 때 “{noop}” 형태임을 잊지 말자 즉 중괄호가 중요하다.
    
    ```java
    @Configuration
    public class ProjectConfig {
    // 섕략된 코드
    	@Bean
    	public PasswordEncoder passwordEncoder() {
    		Map<String, PasswordEncoder> encoders = new HashMap<>();
    		encoders.put("noop",NoOpPasswordEncoder.getInstance());
    		encoders.put("bcrypt",new BCrytPasswordEncoder ());
    		encoders.put("scrypt",new SCrytPasswordEncoder ());
    
    		return new DelegatingPasswordEncoder("bcrypt",encoders); // 기본 인코더 설정
    	}
    }
    ```
    
- 스프링 시큐리티에서는 편의를 위해 모든 표준 제공 PasswordEncoder가 구현된 맵을 가지는 DelegatingPasswordEncoder를 제공한다.
    
    ```java
    PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
    ```
    
- 

## 4.2 스프링 시큐리티 암호화 모듈에 관한 추가 정보

- 이절에서의 배울 내용
    - 암호화 및 복호화 함수와 키 생성 기능은 자바에서 제공해 주지 않는다.
    - 개발자를 위해 스프링 시큐리티는 별도의 자체 솔루션을 제공한다.
    - 4.1에서 구현한 Encoder들도 SSCM의 일부분이다.
    - SSCM이 제공하는 필수 기능 2가지를 알려준다.
        - 키 생성기 : 해싱 및 암호화 알고리즘을 위한 키 생성 객체
        - 암호기 : 데이터를 암호화 및 복호화하는 객체

### 4.2.1 키 생성기 이용

- 특정 종류의 키를 생성하는 개체로서 암호화나 해싱 알고리즘에 필요.
- BytesKeyGenerator 및 StringKeyGenerator는 키 생성기의 주요 인터페이스들이며 팩토리 클래스 KeyGenerators로 직접 만들 수 있다.
    - StringKeyGenerator
        
        ```java
        StringKeyGenerator keyGenrator = KeyGenrator.string();
        String salt = keyGenrator.generateKey();
        ```
        
    - BytesKeyGenerator
        
        ```java
        StringKeyGenerator keyGenrator = KeyGenrator.secureRandom();
        //KeyGenrator.secureRandom(x); <= x는 원하는 8바이트 단위의 길이 키를 생성할 수 있다.
        byte [] key = keyGenrator.generateKey();
        int keyLength = keyGenrator.getKeyLength();
        ```
        

### 4.2.2 암호화와 복호화 작업에 암호기 이용

- 암호화, 복호화는 보안을 위한 공통 기능으로 애플리케이션에서 필요한 기능.
- 시스템 구성 요소 간 데이터를 전송 및 저장할 때 많이 사용.
- 암호기는 암호화와 복호화 작업을 지원하면 SSCM에는 이를 위한 BytesEncryptor 및 TextEncryptor라는 두가지 암호기가 정의되어 있다.
    - BytesEncryptor
        - 바이트 배열로 데이터를 관리한다.
        - 내부적으로 표준 바이트 암호기는 AES256 암호화 방식을 이용해 암호화 한다.
        - 기본 인터페이스
            
            ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%2011.png)
            
        - 기본 구현 예시
            
            ```java
            String salt = KeyGenrator.string().generateKey();
            String password = "secret";
            String valueToEncrypt = "HELLO";
            //CBC(암호 블록 체인)방식
            BytesEncryptor e = Encryptors.standard(password , salt);
            byte[] encrypted = e.encrypt(valueToEncrypt.getBytes());
            byte[] decrypted = e.decrypt(encrypted);
            ```
            
        - 기본 구현보다 강한 암호화 인스턴스를 만들려면  Encryptors.stronger() 메소드를 사용한다.
            
            ```java
            // GCM(갈루아/카운터 모드) 방식
            BytesEncryptor e = Encryptors.stronger(password,salt);
            ```
            
    - TextEncryptor
        - 데이터를 문자열로 관리한다.
        - 입력 받은 문자열을 문자열로 출력한다.
        - 기본 인터페이스
            
            ![Untitled](4%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8B%E1%85%A1%E1%86%B7%E1%84%92%E1%85%A9%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20cc754cc0b59c42659773322118a6d579/Untitled%2012.png)
            
        - 4가지의 주요 형식이 있다.
            - Encryptors.text()
            - Encryptors.delux()
            - Encryptors.queryableText()
            - Encryptors.noOpText() → 더미 TextEncryptor를 반환
        - 구현 예시
            
            ```java
            //noOpText
            String valueToEncrypt = "HELLO";
            TextEncryptor e = Encryptors.noOpText();
            String encrypted = e.encrypt(valueToEncrypt);
            
            //text, delux
            String salt = KeyGenrator.string().generateKey();
            String password = "secret";
            String valueToEncrypt = "HELLO";
            TextEncryptor e = Encryptors.text(password , salt);
            String encrypted = e.encrypt(valueToEncrypt);
            String decrypted = e.decrypt(encrypted);
            
            //queryableText
            String salt = KeyGenrator.string().generateKey();
            String password = "secret";
            String valueToEncrypt = "HELLO";
            TextEncryptor e = Encryptors.queryableText(password , salt);
            String encrypted = e.encrypt(valueToEncrypt);
            ```
            

### 요약

- PasswordEncoder는 인증 논리에서 암호를 처리하는 가장 중요한 책임을 담당한다.
- 스프링 시큐리티는 해싱 알고리증에 여러 대안을 제공하므로 필요한 구현을 선택하기만 하면 된다.
- 스프링 시큐리티 암호화 모델(SSCM)에는 키 생성기와 암호기를 구현하는 여러 대안이 있다.
- 키 생성기는 암호화 알고리즘에 이용되는 키를 생성하도록 도와주는 유틸리티 객체다.
- 암호기는 데이터 암호화와 복호화를 수행하도록 도와주는 유털리티 객체다.
