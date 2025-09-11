# 1. Chat Model API
> Spring AI는 AI 모델을 분류하고, 같은 분류에 속하는 모델들을 벤더사와 무관하게 통일된 방법으로 사용하기 위한 인터페이스를 제공하고 있다. 인터페이스를 통해 특정 모델에 종속되지 않게 응답을 처리할 수 있으며, 이로 인해 AI 모델 변경에 따른 애플리케이션 변경은 최소화 할 수 있게 된다.

| 모델 구분           | 설명                          | 주요 API          |
| --------------- | --------------------------- | --------------- |
| Chat Model      | Text, Image To Text 모델(LLM) | ChatModel       |
| Image Model     | Text To Image 모델            | ImageModel      |
| Audio Model     | TTS, STT                    | SpeechModel     |
| Embedding Model | Text To Vector 모델           | Embedding Model |
위 주요 모델 중, Chat Model은 AI 기반 대화 기능을 애플리케이션에 통합할 수 있도록 한다.

## 1.2 Chat Model

![Chat Model API](./images/chat_model_api.png)
[Spring AI - Chat Model API](https://docs.spring.io/spring-ai/reference/api/chatmodel.html#_available_implementations)

```java
public interface ChatModel extends Model<Prompt, ChatResponse>, StreamingChatModel {  

	// 매개값으로 주어진 문자열, 메시지, 프롬프트로 LLM에 동기요청
    default String call(String message) {  
        Prompt prompt = new Prompt(new UserMessage(message));  
        Generation generation = this.call(prompt).getResult();  
        return generation != null ? generation.getOutput().getText() : "";  
    }  
    
    default String call(Message... messages) {  
        Prompt prompt = new Prompt(Arrays.asList(messages));  
        Generation generation = this.call(prompt).getResult();  
        return generation != null ? generation.getOutput().getText() : "";  
    }  
    
    @Override
    ChatResponse call(Prompt prompt);  
  
    default ChatOptions getDefaultOptions() {  
        return ChatOptions.builder().build();  
    }  
    
    default Flux<ChatResponse> stream(Prompt prompt) {  
        throw new UnsupportedOperationException("streaming is not supported");  
    }}
    
}

// 비동기 요청
@FunctionalInterface  
public interface StreamingChatModel extends StreamingModel<Prompt, ChatResponse> {  
  
    default Flux<String> stream(String message) {  
       Prompt prompt = new Prompt(message);  
       return stream(prompt).map(response -> (response.getResult() == null || response.getResult().getOutput() == null  
             || response.getResult().getOutput().getText() == null) ? ""  
                   : response.getResult().getOutput().getText());  
    }  
    default Flux<String> stream(Message... messages) {  
       Prompt prompt = new Prompt(Arrays.asList(messages));  
       return stream(prompt).map(response -> (response.getResult() == null || response.getResult().getOutput() == null  
             || response.getResult().getOutput().getText() == null) ? ""  
                   : response.getResult().getOutput().getText());  
    }  
    @Override  
    Flux<ChatResponse> stream(Prompt prompt);  
  
}
```

## 1.2 Prompt

## 1.3 Message

## 1.4 ChatOptions

## 1.5 ChatResponse

## 1.6 Generation

