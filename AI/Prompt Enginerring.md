> 关于如何提供清晰并有效的指令给 GPT-3 和 CodeX 的工程。

我们约定使用 `{text}` 来表示需要 AI 理解的文本。

### 使用 `###` 或者 `"""` 来分隔内容

让 AI 能更清晰的理解上下文和它确切需要处理的文本部分。

❌ 低效提示：
```plain
Summarize the text below as a bullet point list of the most important points.  
  
{text}
```

✔️ 正确示范：
```plain
Summarize the text below as a bullet point list of the most important points.

Text: """
{text}
"""
```

此外，用来分隔内容的符号还有：
- Triple quotes: `"""`
- Triple backticks: <code>```</code>
- Triple dashes: `---`
- Angle brackets: `< >`
- XML tags: `<tag> </tag>`


### 更具体的描述

使用更具体的描述来约束 AI 的输出。

❌ 低效提示：

```plain
Write a poem about OpenAI.
```

✔️ 正确示范：

```plain
Write a short inspiring poem about OpenAI, focusing on the recent DALL-E product launch in the style of a {famous poet}
```

### 提供一些示例
GPT 是通用的大型语言模型，一些特定的场景可能得不到我们想要的输出结果，这时可以通过举例的方式，让 AI 理解我们的需求，从而输出特定的结果。

❌ 低效示范：
```plain
Extract the entities mentioned in the text below. Extract the following 4 entity types: company names, people names, specific topics and themes.  
  
Text: {text}
```

✔️ 正确示范：
```plain
Extract the important entities mentioned in the text below. First extract all company names, then extract all people names, then extract specific topics which fit the content and finally extract general overarching themes  
  
Desired format:  
Company names: <comma_separated_list_of_company_names>  
People names: -||-  
Specific topics: -||-  
General themes: -||-  
  
Text: {text}
```

### 提示少量样本+微调
从无样本开始，到提供少量样本，如果还不满意，则可进行微调（fine-tune)。

✔️ 无样本提示：
```plain
Extract keywords from the below text.  
  
Text: {text}  
  
Keywords:
```

✔️ 少量样本提示：
```plain
Extract keywords from the corresponding texts below.  
  
Text 1: Stripe provides APIs that web developers can use to integrate payment processing into their websites and mobile applications.  
Keywords 1: Stripe, payment processing, APIs, web developers, websites, mobile applications  
##  
Text 2: OpenAI has trained cutting-edge language models that are very good at understanding and generating text. Our API provides access to these models and can be used to solve virtually any task that involves processing language.  
Keywords 2: OpenAI, language models, text processing, API.  
##  
Text 3: {text}  
Keywords 3:
```

fine-tune 相关资料：[Classification best practices (Access: Open to All) - Google 文档](https://docs.google.com/document/d/1h-GTjNDDKPKU_Rsd0t1lXCAnHltaXTAzQ8K2HRhQf9U/edit#)

### 不使用模糊的和不明确的描述

❌ 低效示范：
```plain
The description for this product should be fairly short, a few sentences only, and not too much more.
```

✔️ 正确示范：
```plain
Use a 3 to 5 sentence paragraph to describe this product.
```

### 使用“做什么”去代替“不要做”

❌ 低效示范：
```plain
The following is a conversation between an Agent and a Customer. DO NOT ASK USERNAME OR PASSWORD. DO NOT REPEAT.  
  
Customer: I can’t log in to my account.  
Agent:
```

✔️ 正确示范：
```plain
The following is a conversation between an Agent and a Customer. The agent will attempt to diagnose the problem and suggest a solution, whilst refraining from asking any questions related to PII. Instead of asking for PII, such as username or password, refer the user to the help article www.samplewebsite.com/help/faq  
  
Customer: I can’t log in to my account.  
Agent:
```

### 正确地使用请求参数
参考：[OpenAI API](https://platform.openai.com/docs/api-reference/completions/create)