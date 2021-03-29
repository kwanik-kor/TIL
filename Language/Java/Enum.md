# Enum(열거 타입)

#### 2021.03.25

Java 공부를 할 때 Enum의 존재에 대해서만 알고 있었으나, 제대로 알지는 못했으며 이를 활용해 본 적이 없었다. 최근 프로젝트를 진행하면서 Enum을 적극적으로 활용해봄으로써 코드를 보다 직관적이며 안정적으로 짤 수 있었다. 하지만 이는 Enum을 완벽하게 활용했다고 보기는 어려우며, 어떻게 보면 상수(Constants)의 대체재로 사용한 감이 없지 않아 있다. 이에 Enum에 대해 조금 더 깊게 파헤쳐 보고자 한다.

> Oracle Java Documentation <br> [1. Enum](https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html) <br> [2. EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html) <br> [3. EnumMap](https://docs.oracle.com/javase/8/docs/api/java/util/EnumMap.html)

---

### 1. Enum vs Constants

개발을 하면서 변할 일이 적은(거의 없는) 값들을 상수로 선언해서 사용하고는 한다. 파일 내부에 상수로 선언해두기도 하며, 조금 더 용이한 관리를 위해 별도의 constants파일을 작성하기도 하고, 서비스를 함에 있어서 거의 불변인 상수는 'application.yml'과 같이 Property 파일에 작성해 '@Value' Annotation을 사용해 참조하기도 한다. 상황에 따라 상기의 방법들로 작성해야 할 case도 분명 존재하지만, 열거패턴을 사용하는 것은 분명한 한계가 존재한다.

1.  안정성이 보장되지 않는다.
2.  표현력이 좋지 않다.
3.  프로그램이 깨지기 쉽다.

```java
// Example
public static final int COLUMN = 10;
```

Java의 Enum은 일반적인 열거패턴의 단점을 해결함과 동시에 많은 장점을 제공한다.

1. 리팩토링의 범위를 최소화한다.
2. 타입 안정성을 보장한다.
3. IDE가 적극적으로 지원해준다.
4. Java의 Enum은 Object를 상속받은 클래스다.

1, 2, 3항의 내용들은 C/C++/C#의 Enum에서도 동일하게 취할 수 있는 이점이지만 해당 언어에서는 결국 int 값이다. 하지만 Java의 Enum은 4항으로 인해 가장 강력한 Enum이 되었다. <br>
_(C/C++/C#은 사용해본 적이 없어, 책과 우아한형제들 기술 블로그를 인용하였다)_

---

### Reference

1. Joshua Bloch,『Effective Java 3/E』, 이복연, 프로그래밍 인사이트(2018).
