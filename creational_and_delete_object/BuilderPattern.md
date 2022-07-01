## 생성자 인자가 많을때는 Builder 패턴을 고려하라
만약 Blog란 클래스가 존재한다고 가정하자
```java
public class Blog {
    private String title
    private String description
    private String List<String> hashTags = new ArrayList<>();
    private Integer count; 
}
```
여기서 필수 옵션은 title과 description이고 hashTags와 count는 선택 옵션이다 그렇다면 필수 옵션인 title과 description에 값만 저장하고 객체를 생성하려면

```java
public Blog(String title, String description) {
    this.title = title;
    this.description = description;
}
```
해당 두 필드의 값을 할당하는 생성자가 필요할것이다.
하지만 다른 두 옵션의 할당도 필요한 경우라면? 그중에서도 한개의 선택옵션만 할당해야 된다면?

생성자를 signature별로 생성자를 만들어 놓는 방법이 있다. (signature 라고 설명은 하였지만 파라미터의 순서를 바꾸는 행위로 받는 파라미터는 같은 signature라고 가정하고 설명하겠다.)
그러면 

```java
public Blog(String title) {
    this.title = title;
}

public Blog(String title, String description) {
    this.title = title;
    this.description = description;
}

public Blog(String title, String description, List<String> hashTags) {
    this.title = title;
    this.description = description;
    this.hashTags = hashTahgs
}

public Blog(String title, String description, List<String> hashTags, Integer count) {
    this.title = title;
    this.description = description;
    this.hashTags = hashTahgs
    this.count = count;
}
```
이런식으로 각각의 signature별로 생성자를 만드는 방법이 있다. 이것을 점층적 생성자 패턴이라고 말한다. 하지만 생성자의 경우 어떤 필드에 어떤 값이 들어가는지 한눈에 파악하기가 힘들다 그래서 필드의 갯수가 늘어날 경우 점층적 생성자 패턴을 점점 안티패턴이 되어가고 클라이언트 코드에서 읽기가 어려워 질것이다. 여러분도 개발할때 생성자에 어떤 값이 어떤 필드와 매칭되는지 일일이 확인하는 리소스는 낭비하고 싶지 않을것이다.

이를 해결하기 위한 첫번째 방법은 JavaBeans Pattern이다. JavaBeans Pattern을 처음 들어볼 수도 있다. 하지만 Java 개발자 여러분들은 이미 알고 있다 바로 Setter를 이용하여 필드의 값을 할당하는게 바로 JavaBeans Pattern이다. Setter는 모두가 알고 있을거라 생각하고 따로 코드를 적어놓진 않겠다.

하지만 이 JavaBeans Pattern에는 치명적인 단점이있다. 한번의 함수호출로 객체의 생성을 끝낼 수 없다는 점이다.

이해하기 쉽게 설명하자면 4개의 필드가 있는데 Setter는 각각의 메소드에서 한개의 필드씩 값을 할당한다.

그렇기 때문에 한개의 Setter를 호출한다해서 객체의 생성이 끝맺음을 맺지 못한다는 점이다. 그렇기 때문에 4번의 Setter를 다 호출하기 전까지 일시적으로 객체 일관성이 깨진다. 또한 JavaBeans Pattern으로는 불변 클래스를 만들수 없다는것도 단점이다 이것또한 쉽게 얘기하자면 필드의 값이 언제든지 Setter를 호출해 값을 변경할 수 있다는점이다. 

그렇다면 이 모든 단점을 소화시킬 Pattern은 없을까? 있다. 바로 이글의 주인공 Builder Pattern이다. 그럼 Builder Pattern은 어떻게 구현할까?

```java
public class Blog {
    private String title;
    private String description;
    private List<String> hashTags = new ArrayList<>();
    private Integer count;

    public Blog(Builder builder) {
        this.title = builder.title;
        this.description = builder.description;
        this.hashTags = builder.hashTags;
        this.count = builder.count;
    }

    public static class Builder {
        private String title;
        private String description;
        private List<String> hashTags = new ArrayList<>();
        private Integer count;

        public Builder titleAndDescription(String titles, String description) {
            this.title = titles;
            this.description = description;
            return this;
        }

        public Builder hashTagas(List<String> hashTags) {
            this.hashTags = hashTags;
            return this;
        }

        public Builder count(Integer count) {
            this.count = count;
            return this;
        }

        public Blog build() {
            return new Blog(this);
        }
    }

}
```
이렇게 Builder Pattern을 구현해줄 수 있다. 여기서 Builder Pattern의 장점을 몇가지 들여다 보겠다. 

우선 첫번째. Setter는 필드의 값을 하나씩 할당하지만 Builder Pattern에서는 처음 예제 필수 옵션 title과 description을 동시에 값을 할당 시켜줄 수 있게 설계할 수 있다.

두번째. 핵심은 바로 각 필드를 할당하는 메소드, 지금 예제로 보면 static inner class인 Builder class안의 titleAndDescription, hashTags, count메소드들이 무엇을 반환하고 있는지 보이는가? 바로 Builder static inner class를 반환하고 있다 그 이유는 바로 

```java
    Blog.Builder()
        .titleAndDescription(titles, description);
```
이렇게 title과 description을 할당 시켰다고 가정하자 그러면 여기서 객체의 생성이 끝이 나는가? 아니다 이렇게 끝이난다면 JavaBeans Pattern과 다를것이 없다. Builder Pattern의 차이점은 바로 Builder 반환하기 때문에 또다른 Builder의 메소드 hashTags(), count()등을 이용하여 연속적으로 객체의 필드에 값을 할당 시킬 수 있게한다. 

세번째. Builder Pattern을 통해 생성이 끝난 객체는 더이상 Builder Pattern을 통해 값을 변경할 수 없다.