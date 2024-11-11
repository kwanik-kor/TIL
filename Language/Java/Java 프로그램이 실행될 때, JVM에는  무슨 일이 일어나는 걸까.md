
# 1. JVM은 어떻게 생겨먹은걸까

> JVM 은 크게 

# 2. JVM은 메모리를 어떻게 관리하는가


# 3. Class 는 그럼 어떻게 로드되는가
## 3.1 ClassLoader 구조, 종류


### 3.2 ClassLoad 순서
> 로딩, 링크, 초기화

- 메인 클래스이므로 A 클래스를 로딩
	- A 클래스의 구조, 메서드 정보, 필드, 메서드 및 필드의 초기값 로드
	- 각각 적절한 ClassLoader를 사용해서 로드
	- 코드 정보 역시 메서드 영역의 코드 영역에 저장
	- 링크 과정을 통해 메서드 영역에는 원시타입인 경우 제자리에, 레퍼런스 타입인 경우 힙영역에 생성 후 참조
	- 

--- 
### 궁금증

1. 클래스 로더를 통해 로딩되는데 그럼 클래스 로더는 어떻게 클래스 로딩함?
2. 레퍼런스 타입의 원시타입들은 어디에 저장?
3. 왜 String과 원시타입들이 생성되는 영역이 정적 블록과 생성자로 나뉘어져 있는가?
4. 어떤 파일이 메인이 속한 파일인지 JVM은 어떻게 아는가?

---
### References
1. https://brewagebear.github.io/fundamental-jvm-classloader/
2. https://gusdnd852.tistory.com/206
3. https://codevang.tistory.com/83#google_vignette
4. 
