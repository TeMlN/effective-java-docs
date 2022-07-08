### Private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계해라.

싱글턴이란 객체를 하나만 만들 수 있는 클래스이다.

그렇다면 객체를 싱글턴하게 생성할 수 있게 하는 방법은 무엇일까?

<br>

#### public final 필드를 이용한 싱글턴
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}
}
```
이렇게 새로운 객체를 만드는 `생성자`를 INSTANCE 필드를 초기화 할때 한번 호출하고 외부에서 생성자를 호출 할 수 없으니 (생성자가 private 접근 지시자를 가졌기 때문) 이 Elvis 객체는 싱글턴으로 구현된 것입니다. 

그렇다면 이 Elvis객체는 완전무결한 싱글턴 객체일까요? 그렇다고 볼 수 있지만 이 싱글턴을 무너뜨릴 수 있습니다 그 방법은 바로 `reflection`을 이용한 방법인데요.

바로 `AccessibleObject.setAccessible(true)`로 해당 클래스에 대한 access를 허용해주면 private 메소드이든, 필드이든 자유롭게 접근이 가능합니다.

이런 종류의 공격을 방어하고 싶다면 private생성자를 다시 한번 호출 했을떄, 이미 INSTANCE 필드가 초기화 되어 객체가 생성되었다면 예외를 던지게 설계하면 됩니다.

<br>

#### public 접근 제어자를 가진 static factory method를 이용하는 방법

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

앞써 언급한 방법과 유사하지만 차이점은 INSTANCE 필드를 private으로 변경하고 public한 static factory method를 사용했다는 점 입니다. 

성능은 전의 방법보다 떨어집니다 그 이유는 JVM이 inline처리를 해버리기 때문인데 이 inline은 추후에 정리해보도록 하겠습니다.

그리고 이 정적 팩토리 메서드로 싱글턴을 구현할때의 장점은 바로
API를 변경하지 않고도 싱글턴 패턴을 포기할 수 있는것입니다. 가령 스레드 마다 별도의 객체를 반환할 수 있도록 팩터리 메서드를 수정하는 것도 간단합니다. 

또한 직렬화한 객체를 역직렬화를 할때 이 방법은 싱글턴 패턴을 지키기에도 적절합니다. 그 방법은 바로 Serializable 인터페이스를 구현하고 readResolve 란 메소드를 정의를 하면 됩니다 바로

```java
private Object readResolve() {
    return INSTANCE;
}
```
이렇게 정의하시면 됩니다 그 이유는 바로 Serializable에는 readResolve로 직렬화할 필드를 불러오게 되는데, 이미 클래스 내부에 readResolve() 메소드가 존재하면 해당 메소드로 객체를 불러오기 때문에 이 경우에도 항상 같은 인스턴스를 반환하게 되는겁니다.

<br>

#### 싱글턴의 단점
싱글턴의 단점은 바로 테스트에 용이하지 않다는 점입니다. 그 이유는 싱글턴은 항상 똑같은 인스턴스로 생성되기 때문에 싱글턴이 어떤 인터페이스를 구현하고 있는것이 않으면 가짜 구현으로 대체할 수 없기 때문이다.
