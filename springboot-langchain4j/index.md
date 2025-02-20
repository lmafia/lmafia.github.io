# 在 SpringBoot 中接入 LangChain4J 开发指南


<!--more-->

## 简介

LangChain4j 是一个强大的 Java AI 开发框架，它可以帮助我们快速构建基于大语言模型的应用。本文将详细介绍如何在 SpringBoot 项目中集成 LangChain4J，并实现一个简单的 AI 聊天应用。

## 项目架构

{{< mermaid >}}
graph TD
A[SpringBoot Application] --> B[LangChain4J Config]
B --> C[ChatLanguageModel]
B --> D[MessageMemory]
C --> E[OpenAI API]
A --> F[ChatService]
F --> C
F --> D
A --> G[ChatController]
G --> F

style A fill:#f9f,stroke:#333,stroke-width:2px
style B fill:#bbf,stroke:#333,stroke-width:2px
style C fill:#dfd,stroke:#333,stroke-width:2px
style D fill:#dfd,stroke:#333,stroke-width:2px
style E fill:#fdd,stroke:#333,stroke-width:2px
{{< /mermaid >}}

## 环境准备

- JDK 17+
- SpringBoot 3.x
- Maven/Gradle
- OpenAI API Key

## 项目配置

### Maven 依赖

首先在 `pom.xml` 中添加必要的依赖：

```xml
  <dependencies>
    <!-- SpringBoot 基础依赖 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- LangChain4j 核心依赖 -->
    <dependency>
      <groupId>dev.langchain4j</groupId>
      <artifactId>langchain4j</artifactId>
      <version>0.27.1</version>
    </dependency>

    <!-- LangChain4j Spring 集成 -->
    <dependency>
      <groupId>dev.langchain4j</groupId>
      <artifactId>langchain4j-spring-boot-starter</artifactId>
      <version>0.27.1</version>
    </dependency>

    <!-- Lombok -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
  </dependencies>
```

### 配置文件

在 `application.yml` 中添加相关配置：

```yaml
langchain4j:
  chat-model:
    provider: openai
    api-key: ${OPENAI_API_KEY}
    model-name: gpt-4
    temperature: 0.7
    timeout: 60s
    base-url: https://your-gpt4-mini-api-endpoint # 如果使用自定义 API
```

## 核心实现

### 项目结构

```bash
  src/main/java/com/example/demo/
  ├── config/
  │   └── LangChain4jConfig.java
  ├── service/
  │   ├── ChatService.java
  │   └── PromptService.java
  ├── controller/
  │   └── ChatController.java
  ├── model/
  │   ├── ChatRequest.java
  │   └── ChatResponse.java
  └── DemoApplication.java
```

### 配置类

```java
  @Configuration
  @Slf4j
  public class LangChain4jConfig {

    @Value("${langchain4j.chat-model.api-key}")
    private String apiKey;

    @Value("${langchain4j.chat-model.base-url}")
    private String baseUrl;

    @Bean
    public ChatLanguageModel chatLanguageModel() {
      return OpenAiChatModel.builder()
        .apiKey(apiKey)
        .modelName("gpt-4")
        .baseUrl(baseUrl)
        .timeout(Duration.ofSeconds(60))
        .temperature(0.7)
        .logRequests(true)
        .logResponses(true)
        .build();
    }

    @Bean
    public MessageMemory messageMemory() {
      return MessageMemory.withMaxMessages(10);
    }
  }
```

### 服务实现

```java
  @Service
  @Slf4j
  public class ChatService {

    private final ChatLanguageModel chatModel;
    private final MessageMemory memory;

    public ChatService(ChatLanguageModel chatModel, MessageMemory memory) {
      this.chatModel = chatModel;
      this.memory = memory;
    }

    public String chat(String userMessage) {
      try {
        memory.add(UserMessage.from(userMessage));
        String response = chatModel.generate(memory.messages());
        memory.add(AiMessage.from(response));
        return response;
      } catch (Exception e) {
        log.error("AI 服务调用失败", e);
        throw new AIException("AI 服务暂时不可用，请稍后重试");
      }
    }
  }
```

## 高级特性

### 对话流程

{{< mermaid >}}
sequenceDiagram
participant Client
participant Controller
participant ChatService
participant OpenAI

Client->>Controller: POST /api/chat
Controller->>ChatService: chat(message)
ChatService->>ChatService: 添加用户消息到记忆
ChatService->>OpenAI: 生成回复
OpenAI-->>ChatService: 返回回复
ChatService->>ChatService: 添加 AI 回复到记忆
ChatService-->>Controller: 返回回复
Controller-->>Client: 返回响应

note over Client,OpenAI: 完整的对话流程
{{< /mermaid >}}

### Prompt 模板使用

```java
  @Service
  public class PromptService {

    private final PromptTemplate promptTemplate = PromptTemplate.from(
      "你是一个{{role}}，请用专业的角度回答：{{question}}"
    );

    private final ChatLanguageModel chatModel;

    public String getResponse(String role, String question) {
      String prompt = promptTemplate.apply(Parameters.from(
        "role", role,
        "question", question
      ));
      return chatModel.generate(prompt);
    }
  }
```



## 最佳实践

{{< admonition type=tip title="API Key 安全性" open=true >}}

1. 使用环境变量或配置中心存储 API Key
2. 避免直接在代码中硬编码敏感信息
3. 定期轮换 API Key
   {{< /admonition >}}

{{< admonition type=note title="性能优化" open=true >}}

1. 使用异步处理长时间运行的 AI 请求
2. 实现请求缓存减少 API 调用
3. 合理设置超时时间和重试策略
   {{< /admonition >}}

{{< admonition type=warning title="成本控制" open=true >}}

1. 实现 Token 计数和使用统计
2. 设置调用限额
3. 监控 API 调用成本
   {{< /admonition >}}

## 常见问题

{{< admonition type=question title="如何处理 API 超时？" open=true >}}
建议设置合理的超时时间，并实现重试机制：

```java
  @Retryable(
    value = {TimeoutException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000)
  )
  public String chatWithRetry(String message) {
    return chatService.chat(message);
  }
```

{{< /admonition >}}

## 总结

通过本文的介绍，我们学习了如何在 SpringBoot 项目中集成 LangChain4J，实现了一个基础的 AI 聊天应用。主要包括：

1. 基础配置和依赖管理
2. 核心功能实现
3. 高级特性使用
4. 性能优化和监控
5. 安全性考虑

在实际开发中，建议根据具体需求选择合适的功能特性，同时注意安全性、性能和成本控制等方面的考虑。

## 参考资料

- [LangChain4j 官方文档](https://docs.langchain4j.dev)
- [SpringBoot 官方文档](https://spring.io/projects/spring-boot)
- [OpenAI API 文档](https://platform.openai.com/docs/api-reference)

