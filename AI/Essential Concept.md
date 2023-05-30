### [Prompt](https://platform.openai.com/docs/introduction/prompts)
Prompt 概念在 OpenAI 中很重要，这也是用来调试模型的重要入口，在对 AI 模型的编程中，开发者只需提供很少的“提示”和举例，就能让模型输出更准确的回答(Completions)。

### [Token](https://platform.openai.com/docs/introduction/tokens)
AI 模型在处理自然语言的时候，会对自然语言进行断句，使之拆分为数量不定的 “Token”，当然这些都是针对于英文语境的分词。例如 “Hamburger” 会被拆分为 "ham", "bur" 和 "ger"。基本上大部分的单词都会被正常地以空格拆分，例如 “hello” 和 “bye”。
一个 `Token` 大概是 4 个英语字母或者 0.75 个英语单词。知道`Token` 的数量很关键，因为不同的模型处理的 `Token` 数量是不定的，在目前的 `gpt-3.5` 中，能接收和发送的 `Token` 数量是 `2048`个，这大概是 `1500`个单词。OpenAI 也提供了获取自然语言中的 `Token` 数量的工具——[Tokenizer](https://platform.openai.com/tokenizer)。

![](http://cdn.liwuhou.cn/tmp/20230409151652.png)

中文转换的 unicode 码会被可视化工具标记为多个 `Token`。

![](http://cdn.liwuhou.cn/tmp/20230409151916.png)

### [Models](https://platform.openai.com/docs/models)
模型，提供各种不同能力和不同侧重点来处理和加工自然语言的语言集合（A set of models)。例如 GPT-4 和 GPT-3.5-turbo 擅长理解和处理自然对话，CodeX 擅长理解和生成代码。
