---
title: CodeReview Function - OpenAI-Powered Code Review Tool
published: 2025-04-24
description: Implement the CodeReview feature, including knowledge related to Java deserialization.
tags: [CI&CD, CodeReview]
category: OpenAI-Powered Code Review Tool
draft: false
lang: zh
---

# CodeReview Function

本篇主要说明Code Review Part如何实现，上一节我已经实现代码检出，而这一节主要做codeReview的实现

```java
        // 1.代码检出
        ProcessBuilder processBuilder = new ProcessBuilder("git", "diff", "HEAD~1", "HEAD");
        processBuilder.directory(new File("."));

        Process process = processBuilder.start();

        BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
        String line;

        StringBuilder diffCode = new StringBuilder();
        while ((line = reader.readLine()) != null){
            diffCode.append(line);
        }

        int exitCode = process.waitFor();
        System.out.println("Exited with code: " + exitCode);
        System.out.println("Diff_Code " + diffCode.toString());

        //2. chatGLM 代码评审
        String log = codeReview(diffCode.toString());
        System.out.println("code Review:" + log);
```

以下是codeReview的代码

```java
   private static String codeReview(String diffCode) throws Exception {

        String apiKeySecret = "你的API KEY";
        String token = BearerTokenUtils.getToken(apiKeySecret);

        URL url = new URL("https://open.bigmodel.cn/api/paas/v4/chat/completions");
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();

        connection.setRequestMethod("POST");
        connection.setRequestProperty("Authorization", "Bearer " + token);
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");
        connection.setDoOutput(true);

        ChatCompletionRequest chatCompletionRequest = new ChatCompletionRequest();
        chatCompletionRequest.setModel(Model.GLM_4_FLASH.getCode());
        chatCompletionRequest.setMessages(new ArrayList<ChatCompletionRequest.Prompt>() {
            private static final long serialVersionUID = -7988151926241837899L;

            {
                add(new ChatCompletionRequest.Prompt("user", "你是一个高级编程架构师，精通各类场景方案、架构设计和编程语言请，请您根据git diff记录，对代码做出评审。代码如下:"));
                add(new ChatCompletionRequest.Prompt("user", diffCode));
            }
        });

        try (OutputStream os = connection.getOutputStream()) {
            byte[] input = JSON.toJSONString(chatCompletionRequest).getBytes(StandardCharsets.UTF_8);
            os.write(input);
        }

        int responseCode = connection.getResponseCode();
        System.out.println(responseCode);

        BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        String inputLine;

        StringBuilder content = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            content.append(inputLine);
        }

        in.close();
        connection.disconnect();

        System.out.println("评审结果：" + content.toString());

        ChatCompletionSyncResponse response = JSON.parseObject(content.toString(), ChatCompletionSyncResponse.class);
        return response.getChoices().get(0).getMessage().getContent();
    }
}
```

# 关键问题

## 1.为什么要自己分装getToken方法

这是因为**通义的 API 使用的是 JWT（Json Web Token）认证机制，不是简单的 API Key 认证。**

也就是说：

- 你拿到的 `apiKey.apiSecret` 只是你的身份凭证
- 真正能用来访问 API 的是：**你用 `apiSecret` 签名生成的 JWT Token**
- 这个 Token 放在请求头中，服务端可以**验证你确实拥有这个密钥**

使用这种API用JWT认证机制的LLM才需要这样做

```java
package com.github.nanshanvv.sdk.types.utils;

import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

import java.nio.charset.StandardCharsets;
import java.util.Calendar;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;

public class BearerTokenUtils {

    // 过期时间；默认30分钟
    private static final long expireMillis = 30 * 60 * 1000L;

    // 缓存服务
    public static Cache<String, String> cache = CacheBuilder.newBuilder()
            .expireAfterWrite(expireMillis - (60 * 1000L), TimeUnit.MILLISECONDS)
            .build();

    public static String getToken(String apiKeySecret) {
        String[] split = apiKeySecret.split("\\.");
        return getToken(split[0], split[1]);
    }

    /**
     * 对 ApiKey 进行签名
     *
     * @param apiKey    登录创建 ApiKey <a href="https://open.bigmodel.cn/usercenter/apikeys">apikeys</a>
     * @param apiSecret apiKey的后半部分 828902ec516c45307619708d3e780ae1.w5eKiLvhnLP8MtIf 取 w5eKiLvhnLP8MtIf 使用
     * @return Token
     */
    public static String getToken(String apiKey, String apiSecret) {
        // 缓存Token
        String token = cache.getIfPresent(apiKey);
        if (null != token) return token;
        // 创建Token
        Algorithm algorithm = Algorithm.HMAC256(apiSecret.getBytes(StandardCharsets.UTF_8));
        Map<String, Object> payload = new HashMap<>();
        payload.put("api_key", apiKey);
        payload.put("exp", System.currentTimeMillis() + expireMillis);
        payload.put("timestamp", Calendar.getInstance().getTimeInMillis());
        Map<String, Object> headerClaims = new HashMap<>();
        headerClaims.put("alg", "HS256");
        headerClaims.put("sign_type", "SIGN");
        token = JWT.create().withPayload(payload).withHeader(headerClaims).sign(algorithm);
        cache.put(apiKey, token);
        return token;
    }

}

```

## 2.ChatCompletionRequest和ChatCompletionSyncResponse是什么（JAVA的反序列化）

拿ChatCompletionSyncResponse举例

你拿到了一段 JSON，比如：

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "这段代码写得很好！"
      }
    }
  ]
}
```

你需要写一个 Java 类，让它能自动接收上面这段结构的数据。

------

###  思维转化口诀

> JSON → Java 类的过程，其实就是：
>
> - **对象 `{}` → Java 类**
> - **数组 `[]` → List 或数组**
> - **字段 `key: value` → 类的成员变量 + getter/setter**

------

我拿到了以下response

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "这段代码写得很好！"
      }
    }
  ]
}
```

**转换思路：**

- 最外层是一个对象 → 类名可以叫 `ChatCompletionSyncResponse`
- 它有一个字段：`choices` → 是数组 → List<Choice>
- 每个 `choice` 里有一个 `message` 字段 → Message 类型
- Message 有两个字段：`role`, `content` → 都是字符串

------

✅ 步骤二：写 Java 类（可以照着 JSON 一层一层写）

```json
public class ChatCompletionSyncResponse {
    private List<Choice> choices;

    // getter/setter
    public List<Choice> getChoices() {
        return choices;
    }

    public void setChoices(List<Choice> choices) {
        this.choices = choices;
    }

    public static class Choice {
        private Message message;

        public Message getMessage() {
            return message;
        }

        public void setMessage(Message message) {
            this.message = message;
        }
    }

    public static class Message {
        private String role;
        private String content;

        public String getRole() {
            return role;
        }

        public void setRole(String role) {
            this.role = role;
        }

        public String getContent() {
            return content;
        }

        public void setContent(String content) {
            this.content = content;
        }
    }
}
```

------

**✅ 步骤三：用 JSON 库反序列化**

你只要加一句：

```java
ChatCompletionSyncResponse response = JSON.parseObject(jsonStr, ChatCompletionSyncResponse.class);
```

你就能直接通过：

```java
response.getChoices().get(0).getMessage().getContent();
```

拿到要的内容了。

**注意：反序列化（或者叫映射）不需要有所有json中的对象，部分对应也行！**
