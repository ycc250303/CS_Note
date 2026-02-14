# 聊天和语言模型

* 两种LLM API：
  * `LanguageModel`。它们的 API 非常简单 - 接受 `String` 作为输入并返回 `String` 作为输出。 这种 API 现在正在被聊天 API（第二种 API 类型）所取代。
  * `ChatLanguageModel`。这些接受多个 `ChatMessage` 作为输入并返回单个 `AiMessage` 作为输出。 `ChatMessage` 通常包含文本，但某些 LLM 也支持其他模态（例如，图像、音频等）。
* 其他类型：
  * `EmbeddingModel` - 这种模型可以将文本转换为 `Embedding`。
  * `ImageModel` - 这种模型可以生成和编辑 `Image`。
  * `ModerationModel` - 这种模型可以检查文本是否包含有害内容。
  * `ScoringModel` - 这种模型可以对查询的多个文本片段进行评分（或排名）， 本质上确定每个文本片段与查询的相关性。这对 [RAG](https://docs.langchain4j.info/tutorials/rag) 很有用。 

## ChatMessage类型

* `UserMessage`：这是来自用户的消息。 用户可以是您应用程序的最终用户（人类）或您的应用程序本身。`UserMessage` 可以只包含文本（`String`）， 或[其他模态](https://docs.langchain4j.info/tutorials/chat-and-language-models#multimodality)。
* `AiMessage`：这是由 AI 生成的消息，通常是对 `UserMessage` 的回应。 正如您可能已经注意到的，generate 方法返回一个包装在 `Response` 中的 `AiMessage`。 `AiMessage` 可以包含文本响应（`String`）或执行工具的请求（`ToolExecutionRequest`）。
* `ToolExecutionResultMessage`：这是 `ToolExecutionRequest` 的结果。
* `SystemMessage`：这是来自系统的消息。 通常，您作为开发人员应该定义此消息的内容。 通常，您会在这里写入关于 LLM 角色是什么、它应该如何行为、以什么风格回答等指令。 LLM 被训练为比其他类型的消息更加关注 `SystemMessage`。 通常，它位于对话的开始。
* `CustomMessage`：这是一个可以包含任意属性的自定义消息。这种消息类型只能由支持它的 `ChatLanguageModel` 实现使用。

## 多模态

* `UserMessage` 不仅可以包含文本，还可以包含其他类型的内容。 `UserMessage` 包含 `List<Content> contents`。 `Content` 是一个接口，有以下实现：
  * `TextContent`：文本内容，最简单的 `Content` 形式，表示纯文本并包装单个 `String`
  * `ImageContent`：图像内容，可以从**远程**图像的 URL 创建（见上面的示例）， 或从 Base64 编码的二进制数据创建
  * `AudioContent` ：音频内容。
  * `VideoContent` ：视频内容。
  * `PdfFileContent` ： PDF 文件的二进制内容。

# 聊天记忆

* 核心：`ChatMemory`，功能：

  * 淘汰策略
  * 持久化
  * 对 `SystemMessage`的特殊处理
  * 对[工具](https://docs.langchain4j.info/tutorials/tools)消息的特殊处理

## 记忆与历史

* 历史保持用户和AI之间的**所有**消息 **完整无缺** 。历史是用户在UI中看到的内容。它代表实际对话内容。
* 记忆保存 **一些信息** ，这些信息呈现给LLM，使其表现得好像"记住"了对话。 记忆与历史有很大不同。根据使用的记忆算法，它可以以各种方式修改历史： 淘汰一些消息，总结多条消息，总结单独的消息，从消息中删除不重要的细节， 向消息中注入额外信息（例如，用于RAG）或指令（例如，用于结构化输出）等等。

## 淘汰策略

淘汰策略是必要的，原因如下：

* 为了适应LLM的上下文窗口。LLM一次可以处理的令牌数量是有上限的。 在某些时候，对话可能会超过这个限制。在这种情况下，应该淘汰一些消息。 通常，最旧的消息会被淘汰，但如果需要，可以实现更复杂的算法。
* 控制成本。每个令牌都有成本，使每次调用LLM的费用逐渐增加。 淘汰不必要的消息可以降低成本。
* 控制延迟。发送给LLM的令牌越多，处理它们所需的时间就越长。

2种开箱即用的实现：

* 较简单的一种，`MessageWindowChatMemory`，作为滑动窗口运行， 保留最近的 `N`条消息，并淘汰不再适合的旧消息。 然而，由于每条消息可能包含不同数量的令牌， `MessageWindowChatMemory`主要用于快速原型设计。
* 更复杂的选项是 `TokenWindowChatMemory`， 它也作为滑动窗口运行，但专注于保留最近的 `N`个 **令牌** ， 根据需要淘汰旧消息。 消息是不可分割的。如果一条消息不适合，它会被完全淘汰。 `TokenWindowChatMemory`需要一个 `Tokenizer`来计算每个 `ChatMessage`中的令牌数。

# 流式传输

* 对于 `ChatLanguageModel` 和 `LanguageModel` 接口，有相应的 `StreamingChatLanguageModel` 和 `StreamingLanguageModel` 接口。 这些接口有类似的 API，但可以流式传输响应。
* ```java
  public interface StreamingChatResponseHandler {

      void onPartialResponse(String partialResponse);

      void onCompleteResponse(ChatResponse completeResponse);

      void onError(Throwable error);
  }
  ```
* 当生成下一个部分响应时：调用 `onPartialResponse(String partialResponse)`。 部分响应可以由单个或多个标记组成。 例如，您可以在标记可用时立即将其发送到 UI。
* 当 LLM 完成生成时：调用 `onCompleteResponse(ChatResponse completeResponse)`。 `ChatResponse` 对象包含完整的响应（`AiMessage`）以及 `ChatResponseMetadata`。
* 当发生错误时：调用 `onError(Throwable error)`。

# AI Services

* AI 服务。 其思想是将与 LLM 和其他组件交互的复杂性隐藏在简单的 API 后面。

```java
interface Assistant {
    String chat(String userMessage);
}

ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName(GPT_4_O_MINI)
    .build();

Assistant assistant = AiServices.create(Assistant.class, model);

String answer = assistant.chat("Hello");
System.out.println(answer); // Hello, how can I help you?
```

# 工具

* 它允许 LLM 在必要时调用一个或多个可用的工具，通常由开发者定义。
* 工具可以是任何东西：网络搜索、调用外部 API 或执行特定代码片段等。
* LLM 实际上不能自己调用工具；相反，它们在响应中表达调用特定工具的意图（而不是以纯文本形式响应）。 作为开发者，我们应该使用提供的参数执行这个工具，并将工具执行的结果反馈回来。

## 抽象级别

* 低级：使用 `ChatLanguageModel` 和 `ToolSpecification` API
* 高级：使用 [AI 服务](https://docs.langchain4j.info/tutorials/ai-services)和带有 `@Tool` 注解的 Java 方法

## 低级工具

* 在创建 `ChatRequest` 时指定一个或多个 `ToolSpecification`。`ToolSpecification` 是一个包含工具所有信息的对象：

  * 工具的 `name`（名称）
  * 工具的 `description`（描述）
  * 工具的 `parameters`（参数）及其描述

## 高级工具


# RAG

* RAG 是一种在发送给 LLM 之前，从你的数据中找到并注入相关信息片段到提示中的方法。 这样 LLM 将获得（希望是）相关信息，并能够使用这些信息回复， 这应该会降低产生幻觉的概率。

## 实现过程

### 索引

* 在索引阶段，文档会被预处理，以便在检索阶段进行高效搜索。
* 对于向量搜索，这通常涉及清理文档、用额外数据和元数据丰富文档、 将文档分割成更小的片段（也称为分块）、嵌入这些片段，最后将它们存储在嵌入存储（也称为向量数据库）中。

![1762879418388](image/教程笔记/1762879418388.png)

### 检索

* 检索阶段通常在线进行，当用户提交一个应该使用索引文档回答的问题时。
* 对于向量搜索，这通常涉及嵌入用户的查询（问题） 并在嵌入存储中执行相似度搜索。 然后将相关片段（原始文档的片段）注入到提示中并发送给 LLM。

![1762879705735](image/教程笔记/1762879705735.png)

## 三种RAG

* [Easy RAG](https://docs.langchain4j.info/tutorials/rag/#easy-rag)：开始使用 RAG 的最简单方式
* [Naive RAG](https://docs.langchain4j.info/tutorials/rag/#naive-rag)：使用向量搜索的基本 RAG 实现
* [Advanced RAG](https://docs.langchain4j.info/tutorials/rag/#advanced-rag)：一个模块化的 RAG 框架，允许额外的步骤，如 查询转换、从多个来源检索和重新排序

### 简单RAG

* 加载文档
