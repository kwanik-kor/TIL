# Enum(열거 타입)

#### 2021.03.25

Java 공부를 할 때 Enum의 존재에 대해서만 알고 있었으나, 제대로 알지는 못했으며 이를 활용해 본 적이 없었다. 최근 프로젝트를 진행하면서 Enum을 적극적으로 활용해봄으로써 코드를 보다 직관적이며 안정적으로 짤 수 있었다. 하지만 이는 Enum을 완벽하게 활용했다고 보기는 어려우며, 어떻게 보면 상수(Constants)의 대체재로 사용한 감이 없지 않아 있다. 이에 Enum에 대해 조금 더 깊게 파헤쳐 보고자 한다.

> Oracle Java Documentation <br> > [1. Enum](https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html) <br> > [2. EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html) <br> > [3. EnumMap](https://docs.oracle.com/javase/8/docs/api/java/util/EnumMap.html)

---

### Reference

1. Joshua Bloch,『Effective Java 3/E』, 이복연, 프로그래밍 인사이트(2018).
