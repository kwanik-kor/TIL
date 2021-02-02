# Client에서 Content-Disposition을 받지 못했다?

#### 2021.01.12

> Browser가 접근하지 못한 Header정보는 CORS 문제로, Server에서 CORS Setting을 해주거나, @CrossOrigin Annotation을 통해 Header 정보에 접근하게 해주자

프로젝트에서 Excel File을 생성해서 Client 측에 내려주거나, 대량의 Excel File들을 zip으로 묶어서 내려주는 API를 개발해야 했다. 단일 Excel File의 경우에는 ByteArrayOutputStream을, zip은 ZipOutputStream으로 ResponseEntity에 Resource를 실어 내려보냈다.

File을 Export 하기 위한 기본적인 Setting은 아래와 같이 지정했다.

```java
HttpHeaders headers = new HttpH8eaders();
headers.setContentDisposition(ContentDisposition.builder("attachment")
        .filename(filename, StandardCharsets.UTF_8)
        .build());
headers.add("Content-Type", "application/download; charset=UTF-8");
headers.add("Content-Transfer-Encoding", "binary");
```

Swagger에서 Test를 진행할 때나 Client에서 요청 결과로 Network를 살펴보면

```
Content-Disposition: attachment; filename*=UTF-8''%EB%82%9%EB%8A%94%20%EC%84%A4%EC%A0%95%EA%B0%92%EC%9D%B4%EB%8B%A4._20210112.xlsx
```

와 같이 Content-Disposition과 attachment 뒤에 encoding된 파일 명이 붙어 있는 것을 확인할 수 있었다.

또한, 클라이언트 측에서 API를 때렸을 때, Response Header에 해당 정보가 실려 있는 것을 확인했으나 fileName을 읽어 들일 수가 없다는 것이었다.

---

**문제의 원인은 CORS 였다.**

Content-Disposition은 Browser가 기본적으로 접근가능한 헤더가 아니었고, 이를 접근할 수 있게 서버에서 접근을 허용해줘야만 했던 것이다.

서버의 CORS Option은 인증/권한 으로 인해 이미 'exposedHeaders'를 Setting 해준 상황이라, 파일을 다운로드 해야 하는 특정 API들에만 Annotation으로 exposedHeaders를 부가적으로 setting해주기로 결정했고 아래와 같이 처리했다.

```java
@GetMapping("/API명칭")
@CrossOrigin(value="*", exposedHeaders={"Content-Disposition"})
public ResponseEntity<Resource> exportFile(...) {
    ...
}
```

---

#### References

- [CORS Support in spring](https://spring.io/blog/2015/06/08/cors-support-in-spring-framework)
