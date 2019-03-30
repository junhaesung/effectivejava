# Effective Java 3rd item 1 ~ 10

1장 들어가기 (Introduction)
2장 객체의 생성과 파괴 (Creating and Destroying Objects)
3장 모든 객체의 공통 메서드 (Methods Common to All Objects)

## Item 1: 생성자 대신 정적 팩터리 메서드를 고려하라 (Consider static factory methods instead of constructors)

### 장점

1. 이름을 가질 수 있다.
2. 호출할 때 마다 인스턴스를 새로 만들지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있다
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적팩터리메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    * 서비스 제공자 프레임워크
        * ex) JDBC API

### 단점

1. 정적 팩터리 메서드만 제공하면 상속을 할 수 없다.
    * 상속을 하려면 public 이나 protected 생성자가 필요하기 때문이다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
    * javadoc을 지원하지 않는다.
    * 문서를 잘 쓰고, 이름을 잘 지어준다.

### 자주 사용하는 정적 팩터리 메서드 이름

* from : 매개변수 하나, 해당 타입
* of : 매개변수 여러개, 적절한 타입
* valueOf : from 과 of 의 자세한 버전
* instance, getInstance : 매개변수를 받으면, 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다. 
* create, newInstance : 매번 새로운 인스턴스를 반환할 것을 보장한다. 
* getType : getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 정적 팩터리 메서드를 작성할 때 사용 
* newType : newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 정적 팩터리 메서드를 작성할 때 사용 
* type : getType, newType 의 간결한 버전

### 찾아보기
- 동반 클래스 (companion class) : 자바 8 전에는 .. 이름이 Type인 인터페이스를 반환하는 정적 메서드가 필요하면 Types라는 인스턴스화 불가한 동반 클래스를 만들어 그 안에 정의하는 것이 관례였다. ... 자바 8부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸기 때문에 인스턴스화 불가 동반 클래스를 둘 이유가 별로 없다. 동반 클래스에 두었던 퍼블릭 정적 멤버들 상당수를 그냥 인터페이스 자체에 두면 되는 것이다. 

- 서비스 제공자 프레임워크
- https://en.wikipedia.org/wiki/Service_provider_interface
- https://www.baeldung.com/java-spi
```
A service provider framework is a system in which providers implement a service, and the system makes the implementations avilable to clients, decoupling the clients from the implementations.
서비스 프로 바이더 프레임 워크는, 프로 바이더가 서비스를 구현하는 시스템으로, 시스템은 클라이언트에게 구현을 사용 가능하게 해, 클라이언트를 구현으로부터 분리합니다.
```
- spring security 도 서비스 제공자 프레임워크 패턴을 사용하는걸까? 
    - ProviderManager, ~Provider, UserDeatilsService 등 provider, service 이름이 보여서 질문.
    - 근데, 스프링시큐리티는 서비스 제공자 프레임워크라고 보기 어렵다고 답변 받음. (동묘)
        - 서비스 제공자 프레임워크는 provider를 사용하는 것 뿐만 아니라 discovery, register 하는 책임도 가지고 있어야 하는데, 
        - 스프링시큐리티는 provider만 가지고있고, third-party 에서 제공해주는 라이브러리 같은건 없음
        - provider를 구현해서 직접 주입하고 사용해야 함
        - 진짜 서비스 제공자 프레임워크가 되려면, 'ㅇㅇ'인증방식으로 사용하고 싶어! 라고 이름정도만 넘기면, 프레임워크가 슥슥 찾고, 알아서 해주어야 됨
        - 시큐리티는, 직접 구상체(코드)를 넘겨주어야 하니깐, 역할이 좀 약한 것 같다. 
        - 그리고 provider, service는 꼭 서비스 제공자 프레임워크에서만 사용하는 이름은 아니다. 
- toast-iam/.../configuration/security/SecurityConfiguration.java

## Item 2: 생성자에 매개변수가 많다면 빌더를 고려하라 (Consider a builder when faced with many constructor parameters)

### 패턴 비교

1. 점층적 생성자 패턴
    * 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.
2. 자바빈즈 패턴
    * 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되지 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.
    * 클래스를 불변으로 만들 수 없다. 스레드 안전성을 얻으려면 추가 작업을 해야한다.
3. 빌더 패턴
    * (파이썬과 스칼라에 있는) 명명된 선택적 매개변수(named optional parameters)를 흉내 낸 것이다.
    * build 메서드에가 호출하는 생성자에서 여러 매개변수에 걸친 불변식(invariant)을 검사하자
    * 잘못된 점을 발견하면, 어떤 매개변수가 잘못되었는지를 알려주는 메시지를 담아 IllegalArgumentException 을 던지면 된다.

### 빌더 패턴의 장점

1. 계층적 구조의 클래스에서 사용하기 좋다
2. 가변인수 매개변수를 여러 개 사용할 수 있다. (생성자 대비 이점)
3. 유연하다
    * 빌더 하나로 여러 객체를 순회하면서 만들 수 있다.
    * 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.
    * 일련번호 등은 빌더가 알아서 채우도록 할 수도 있다.

### 빌더 패턴의 단점

1. 객체를 만들려면, 빌더부터 만들어야 한다.
    * 성능에 민감한 상황에서는 문제가 될 수 있다.
2. 점층적 생성자 패턴보다 코드가 장황하다.
    * 매개변수가 4개 이상 되어야 값어치를 한다.

```
The builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, ...
```

### lombok 사용하기
- https://projectlombok.org/features/Builder
- @Builder annotation
- AllArgsContructor 생성됨
- AllArgsContructor 생성자가 없으면 @Builder 가 잘 동작하지 않음

### lombok-builder, jackson 
- jackson 이 deserialize 하려면 기본 생성자가 필요함
- 인자 없는 private 생성자도 ok
- @AllArgsConstructor(access = AccessLevel.PRIVATE)
- @NoArgsConstructor(access = AccessLevel.PRIVATE)

### 빌더 객체를 만드는게 성능에 어느정도 영향을 주는지 테스트
- 아래 코드를 사용해서 생성자/빌더 각각
    1. 약 1분동안 3천번 (100 * 10 * 3) API 호출
    2. 약 5분동안 1만번 (100 * 10 * 10) API 호출
- gc 결과는 별 차이 없다. 
- 테스트 방법이 부정확한 것 같기도 하다. 

```java
@Builder
@Getter
@ToString
public class Person {
    private String name;
    private Integer age;
    private String address;
    private String nickname;
}
```
```java
@RestController
public class HelloController {

    @ModelAttribute("helloRequest")
    public Person createHelloRequest(@RequestParam("type") String constructorType,
                                     @RequestParam String name,
                                     @RequestParam int age,
                                     @RequestParam String address,
                                     @RequestParam String nickname) {
        switch(ConstructorType.valueOf(constructorType.toUpperCase())) {
            case CONSTRUCTOR:
                return new Person(name, age, address, nickname);
            case BUILDER:
                return Person.builder()
                        .name(name)
                        .age(age)
                        .address(address)
                        .nickname(nickname)
                        .build();
            default:
                throw new IllegalArgumentException("'constructorType' is not supported. constructorType: " + constructorType);
        }
    }

    @GetMapping("/hello")
    public String hello(@ModelAttribute("helloRequest") Person person) {
        return person.toString();
    }
}
```
```java
public enum ConstructorType {
    CONSTRUCTOR, BUILDER
}
```

## Item 3: private 생성자나 열거 타입으로 싱글턴임을 보증하라 (Enforce the singleton property with a private constructor or an enum type)
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. 
- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 

### 싱글턴 생성
- public static final 필드 방식
    - 권한이 있는 클라이언트는 accessableObject.setAccessible 을 사용해 private 생성자를 호출할 수 있다. 
- 정적 팩터리 메서드를 public static 멤버로 제공. 
- 원소가 하나인 enum 타입 선언

### 직렬화/역직렬화
역직렬화할 때 여러 인스턴스가 생김

jdk 11 부터 setAccessable warning
이후에는 error 일 것

## Item 4: 인스턴스화를 막으려거든 private 생성자를 사용하라 (Enforce noninstantiability with a private constructor)
* 추상클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. 
* private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다. 
* 생성자인데 생성을 못하게 하면 직관적이지 않으니, 주석을 달아두자. 
* lombok - @UtilityClass

## `````new````` Item 5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 (Prefer dependency injection to hardwiring resource) 
- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다. 
- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다. 
- 의존 관계 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있다. 
    - 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다. 

### String Literal
https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.10.5
```
A string literal is a reference to an instance of class String (§4.3.1, §4.3.3).
문자열 리터럴은 String 클래스의 인스턴스에 대한 참조입니다.

Moreover, a string literal always refers to the same instance of class String. This is because string literals - or, more generally, strings that are the values of constant expressions (§15.28) - are "interned" so as to share unique instances, using the method String.intern.
게다가 문자열 리터럴은 항상 String 클래스의 동일한 인스턴스를 참조합니다. 이는 문자열 리터럴 또는 더 일반적으로 상수 표현식 (15.28 절)의 값인 문자열이 String.intern 메서드를 사용하여 고유 한 인스턴스를 공유하도록 "인턴드"되기 때문입니다.
```

### 의존 관계 주입
http://www.mimul.com/pebble/default/2018/03/30/1522386129211.html
- DI(의존성 주입)가 필요한 이유
- Spring에서 Field Injection보다 Constructor Injection이 권장되는 이유
    1. 단일 책임의 원칙
    2. 테스트 용이성
    3. Immutability
    4. 순환 의존성
    5. 의존성 명시
## Item 6: 불필요한 객체 생성을 피하라 (Avoid creating unnecessary objects)
- 생성 비용이 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하자. 
    - String.matches -> Pattern 인스턴스 캐싱
    - 지연 초기화는 권하지 않는다. 코드가 복잡해지는데, 성능은 크게 개선되지 않을 때가 많기 때문

### 덜 명확하거나, 직관에 반대되는 상황
#### 어댑터 객체
- 뒷단 객체 하나당 어댑터 하나씩 만들어지면 충분하다. 
### 불필요한 객체를 만들어내는 예
#### 오토박싱(auto boxing)
- 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
- 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자. 

thread safe를 확인하자 (문서)

### 정리
- 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다. 
- 방어적 복사(defensive copy)와 대조적이다. 

## Item 7: 다 쓴 객체 참조를 해제하라 (Eliminate obsolete object references)
### 메모리 누수
객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체...)를 회수해가지 못한다. 그래서 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있다. 

### 해법
- 해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다. 
- 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다. 

### 캐시
역시 메모리 누수를 일으키는 주범이다. 
WeakHashMap을 사용해 캐시를 만들자. 다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다. 
백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있다. 

weakHashMap ? 

ibm heap profiler 돈안내고써도됨
https://www.ibm.com/developerworks/community/groups/service/html/communityview?communityUuid=4544bafe-c7a2-455f-9d43-eb866ea60091

## Item 8: finalizer 와 cleaner 사용을 피하라 (Avoid finalizers and cleaners)
- finallizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 
- cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다. 

### 주의점
- finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다. 
- 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다. 
- finalizer와 cleaner는 심각한 성능 문제도 동반한다. 
- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. 

### 대안
- AutoCloseable 을 구현해주고, 인스턴스를 다 쓰고나면 close를 호출하면 된다. 

## `````new````` Item 9: try-finally 보다는 try-with-resources 를 사용하라 (Prefer try-with-resources to try-finally) 
try-with-resources 에서도 catch 절을 쓸 수 있다. 

### try-with-resources
https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20.3
```
The type of a variable declared in a ResourceSpecification must be a subtype of AutoCloseable, or a compile-time error occurs.
ResourceSpecification에 선언 된 변수의 유형은 AutoCloseable의 부속 유형이어야하며 그렇지 않으면 컴파일 타임 오류가 발생합니다.
```

### 자바 퍼즐러 pp132-134
- 이 책 저자가, try-finally 잘못 사용한 예제 코드가 있다고 해서 본문 첨부. 

- 틀린 부분 설명해주는 .. stackoverflow 답변
    - https://stackoverflow.com/questions/48449093/what-is-wrong-with-this-java-puzzlers-piece-of-code 

#### 41번째 퍼즐 - 스트림
아래의 copy() 메서드는 파일을 복사하는 메서드입니다. 이 메서드는 실행될 때마다 try 블록에서 스트림을 열고 finally 블록에서 스트림을 닫게 설계되어 있습니다. 그런데 정말 그렇게 될까요? 안 된다면 어떻게 고쳐야 할까요?

```java
static void copy(String src, String dest) throws IOException {
    InputStream in = null;
    OutputStream out = nul;
    try {
        in = new FileInputStream(src);
        out = new FileOutputStream(dest);
        byte[] buf = new byte[1024];
        int n;
        while((n == in.read(buf)) > 0)
            out.write(buf, 0, n);
    } finally {
        if (in != null) in.close();
        if (out != null) out.close();
    }
}
```
#### 풀이41 - 스트림
스트림 관련 변수 in과 out은 모두 null로 초기화되었으며 생성될 때 스트림을 할당합니다. 그리고 finally 블록에서는 변수 in과 out이 null이 아닌 경우에는 close() 메서드를 호출해서 스트림을 닫습니다. 파일을 복사하는 도중에 IOException이 발생할 수는 있습니다. 하지만 finally 블록은 당연히 실행되므로 딱히 문제가 없어 보입니다. 그런데 문제가 있다니, 어디가 잘못 된 걸까요?

이번 퍼즐의 문제는 close() 메서드도 IOException 이라는 예외가 발생할 수 있다는 것입니다. 만약 in.close() 메서드를 호출하다가 IOExcetion 이 발생하면 코드가 중단되어 out.close() 메서드가 실행되지 않습니다. 이렇게 되면 출력 스트림이 닫히지 않습니다. 

그리고 이번 퍼즐은 36번째 퍼즐에서 배운 내용을 위반하고 있습니다. close() 메서드를 호출하면 finally 블록이 중단될 수 있습니다. 하지만 현재 코드에서 IOExcetion 을 throw 하므로 컴파일러는 이 부분에 문제가 있다는 것을 알려주지 않습니다. 

이런 문제를 해결하려면 close() 메서드 호출하되는 부분을 try 블록으로 감싸주어야 합니다. 다음과 같이 코드를 수정하면 변수 in과 out 스트림 모두 정상적으로 닫힐 것입니다. 
```java
...
} finally {
    if (in != null) {
        try {
            in.close();
        } catch (IOException ex) {
            // close() 메서드가 실패해도 딱히 할 일이 없습니다. 
        }
    }
    
    if (out != null) {
        try {
            out.close();
        } catch (IOException ex) {
            // close() 메서드가 실패해도 딱히 할 일이 없습니다. 
        }
    }
}
```
추가적으로 자바 5부터는 다음과 같이 Closeable 인터페이스의 기능을 활용할 수 있습니다. 
```java
..
} finally {
    closeIgnoringException(in);
    closeIgnoringException(out);
}

private static void closeIgnoringException(Closeable c) {
    if (c != null) {
        try {
            c.close();
        } catch (IOException ex) {
            // close() 메서드가 실패해도 딱히 할 일이 없습니다. 
        }
    }
}
```

정리하겠습니다. finally 블록 내부에서 close() 메서드를 호출할 때는 try-catch 블록으로 예외를 처리하기 바랍니다. 추가로 finally 블록 내부에서 발생할 수 있는 예외도 throw 로 처리하기보다는 finally 블록 내부에서 모두 처리하기 바랍니다. 그리고 36번째 퍼즐도 다시 한 번 살펴보기 바랍니다. 


## Item 10: equals 는 일반 규약을 지켜 재정의하라 (Obey the general contract when overriding equals)

### Float, Double은 compare를 사용하자. equals을 사용하면 autoBoxing이 일어날 수 있어서 성능상 좋지 않다. 
https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.21.1

### lombok - @EqualsAndHashCode
https://projectlombok.org/features/EqualsAndHashCode
```java
import java.time.ZonedDateTime;

public class LombokTester {
    int id;
    Integer boxedId;
    long longId;
    Long boxedLongId;
    float version;
    double money;
    String name;
    ZonedDateTime createdAt;
    Object companionObject;
    Class aClass;

    public LombokTester() {
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof LombokTester)) {
            return false;
        } else {
            LombokTester other = (LombokTester)o;
            if (!other.canEqual(this)) {
                return false;
            } else if (this.id != other.id) {
                return false;
            } else {
                label97: {
                    Object this$boxedId = this.boxedId;
                    Object other$boxedId = other.boxedId;
                    if (this$boxedId == null) {
                        if (other$boxedId == null) {
                            break label97;
                        }
                    } else if (this$boxedId.equals(other$boxedId)) {
                        break label97;
                    }

                    return false;
                }

                if (this.longId != other.longId) {
                    return false;
                } else {
                    label89: {
                        Object this$boxedLongId = this.boxedLongId;
                        Object other$boxedLongId = other.boxedLongId;
                        if (this$boxedLongId == null) {
                            if (other$boxedLongId == null) {
                                break label89;
                            }
                        } else if (this$boxedLongId.equals(other$boxedLongId)) {
                            break label89;
                        }

                        return false;
                    }

                    if (Float.compare(this.version, other.version) != 0) {
                        return false;
                    } else if (Double.compare(this.money, other.money) != 0) {
                        return false;
                    } else {
                        label79: {
                            Object this$name = this.name;
                            Object other$name = other.name;
                            if (this$name == null) {
                                if (other$name == null) {
                                    break label79;
                                }
                            } else if (this$name.equals(other$name)) {
                                break label79;
                            }

                            return false;
                        }

                        Object this$createdAt = this.createdAt;
                        Object other$createdAt = other.createdAt;
                        if (this$createdAt == null) {
                            if (other$createdAt != null) {
                                return false;
                            }
                        } else if (!this$createdAt.equals(other$createdAt)) {
                            return false;
                        }

                        Object this$companionObject = this.companionObject;
                        Object other$companionObject = other.companionObject;
                        if (this$companionObject == null) {
                            if (other$companionObject != null) {
                                return false;
                            }
                        } else if (!this$companionObject.equals(other$companionObject)) {
                            return false;
                        }

                        Object this$aClass = this.aClass;
                        Object other$aClass = other.aClass;
                        if (this$aClass == null) {
                            if (other$aClass != null) {
                                return false;
                            }
                        } else if (!this$aClass.equals(other$aClass)) {
                            return false;
                        }

                        return true;
                    }
                }
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof LombokTester;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        int result = result * 59 + this.id;
        Object $boxedId = this.boxedId;
        result = result * 59 + ($boxedId == null ? 43 : $boxedId.hashCode());
        long $longId = this.longId;
        result = result * 59 + (int)($longId >>> 32 ^ $longId);
        Object $boxedLongId = this.boxedLongId;
        result = result * 59 + ($boxedLongId == null ? 43 : $boxedLongId.hashCode());
        result = result * 59 + Float.floatToIntBits(this.version);
        long $money = Double.doubleToLongBits(this.money);
        result = result * 59 + (int)($money >>> 32 ^ $money);
        Object $name = this.name;
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        Object $createdAt = this.createdAt;
        result = result * 59 + ($createdAt == null ? 43 : $createdAt.hashCode());
        Object $companionObject = this.companionObject;
        result = result * 59 + ($companionObject == null ? 43 : $companionObject.hashCode());
        Object $aClass = this.aClass;
        result = result * 59 + ($aClass == null ? 43 : $aClass.hashCode());
        return result;
    }
}
```

### IDE 플러그인 사용 (Intellij)
- Guava equals, hashCode and toString generator
    - guava library 를 사용하는 플러그인
 ```java
 import com.google.common.base.Objects;

import java.time.ZonedDateTime;

public class GuavaTester {
    int id;
    Integer boxedId;
    long longId;
    Long boxedLongId;
    float version;
    double money;
    String name;
    ZonedDateTime createdAt;
    Object companionObject;
    Class aClass;

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;

        GuavaTester that = (GuavaTester) o;

        return Objects.equal(this.id, that.id) &&
               Objects.equal(this.boxedId, that.boxedId) &&
               Objects.equal(this.longId, that.longId) &&
               Objects.equal(this.boxedLongId, that.boxedLongId) &&
               Objects.equal(this.version, that.version) &&
               Objects.equal(this.money, that.money) &&
               Objects.equal(this.name, that.name) &&
               Objects.equal(this.createdAt, that.createdAt) &&
               Objects.equal(this.companionObject, that.companionObject) &&
               Objects.equal(this.aClass, that.aClass);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(id, boxedId, longId, boxedLongId, version, money,
                                name, createdAt, companionObject, aClass);
    }
}

 ```
 
# 메모

### 찾아볼 것

#### item 1

* GoF 팩터리 메서드 (Factory Method)
* GoF 브리지 패턴 (p12)
* 서비스 제공자 프레임워크 패턴
    * jdbc

#### item 2

* lombok builder
* GoF 빌더 패턴 (Builder)
* 빌더 생성 비용 테스트

#### item 3

* 리플렉션 api 사용해서 값을 바꾸는 예제
* GoF 싱글턴

#### item 4

* 없음

#### item 5

* GoF 팩터리 메서드 패턴 (Factory Method Pattern)
* 대거(Dagger), 주스(Guice), 스프링(Spring)

#### item 6

* AutoBoxing

#### item 7

* 없음

#### item 8

* finalizer in (FileInputSystem, FileOutputSystem, ThreadPoolExecutor)
* AuthClosable

#### item 9

#### item 10
* lombok - equalsAndHashCode
* AutoValue Framework
    * https://github.com/google/auto/tree/master/value
