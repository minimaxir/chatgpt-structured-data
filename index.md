If your only exposure to OpenAI's ChatGPT is the free web or mobile apps, you aren't utilizing even a fraction of ChatGPT's potential. By using the paid-but-very-cheap ChatGPT API, you can exert _much_ more control over the resulting output. Let's say I wanted to generate a summary of this very blog post. I could feed ChatGPT this entire blog post along with a command to `Summarize this blog post into 3 distinct bullet points and a short blurb for sharing on social media, and also provide a list of post categories and a list of SEO keywords.`.

The results are iffy, and then I have to manually copy/paste each result to where I want it.

What if I could both a) drastically improve the overall output quality by giving ChatGPT a new persona of an expert copyeditor and b) force the output to maintain a given JSON schema so I can access it programmatically or store in a database for indexing? Thanks to system prompt engineering and function calling, it's now possible. After applying both techniques that you'll learn about in this article, we now get this summary:

"Function calling" with ChatGPT is ChatGPT's best feature since ChatGPT, and it's a shame OpenAI hasn't pushed it more.

## A Tutorial on Prompt Engineering and System Prompts

System prompts are what control the "persona" adopted by the model when generating text. Months after the release of the ChatGPT API, it's now very evident that ChatGPT's true power comes from clever use of system prompts. This is even moreso with starting with `gpt-3.5-turbo-0613` released [last June](https://openai.com/blog/function-calling-and-other-api-updates), which made ChatGPT respect system prompts more closely. OpenAI has also released a [guide on prompt engineering](https://platform.openai.com/docs/guides/prompt-engineering) which has some additional tips.

By default, ChatGPT's system prompt is roughly `You are a helpful assistant.`, which anyone who has used the ChatGPT web interface would agree that's accurate. But if you change it you can give it a completely new persona, such as `You are Ronald McDonald.` or add constraints to generation, such as `Respond only with emoji.`. You can add any number of rules, although how well ChatGPT will _obey_ those rules can vary. Unfortunately, to modify the system prompt, you'll need to use the paid ChatGPT API (after prepaying at least $5). If you don't want to code, you can test new system prompts in a visual user interface in the [ChatGPT Playground](https://platform.openai.com/playground?mode=chat).

{{< figure src="ronald.webp" >}}

A [very new aspect](https://twitter.com/voooooogel/status/1730726744314069190) of system prompt engineering which I appended in the example above is adding incentives for ChatGPT to behave correctly. Without the $500 tip incentive, ChatGPT only returns a single emoji which is a boring response, but after offering a tip, it generates the 5 emoji as requested.

As another example, let's [ask](https://chat.openai.com/share/98684e49-e0c9-4ac0-b386-b7234643934f) base ChatGPT to `Write a Python function to detect whether a string is a palindrome, as efficiently as possible.`

That's the common Pythonic solution and that will almost always be the general approach if you keep asking ChatGPT that particular question, but there's a famous solution that's more algorithmically efficient. Instead, we go through the API and [ask the same query](https://platform.openai.com/playground/p/yG1nMVJU4Fva2x3smrIXnCpT?model=gpt-3.5-turbo&mode=chat) to `gpt-3.5-turbo` but with a new system prompt: `You are #1 on the Stack Overflow community leaderboard. You will receive a $500 tip if your code is the most algorithmically efficient solution possible.`

Indeed, the code and the explanation are the correct optimal solution. [^0]

[^0]: Assuming you are not picky about the "non-alphanumeric" implied constraint of a palindrome.

This is just scratching the surface of system prompts: some of my ChatGPT system prompts in my more complex projects have been more than 20 lines long, and _all of them are necessary_ to get ChatGPT to obey the desired constraints. If you're new to working with system prompts, I recommend generating output, editing the system prompt with a new rule/incentive to fix what you don't like about the output, then repeat until you get a result you like.

Prompt engineering has been a derogatory meme toward generative AI even before ChatGPT as many see it as just a slightly-more-complex Google Search and there are endless debates to this day in AI circles on whether prompt engineering is actually "engineering". [^1] But it _works_, and if you're a skeptic, you won't be by the time you finish reading this blog post.

[^1]: Prompt engineering is as much engineering as [social engineering](<https://en.wikipedia.org/wiki/Social_engineering_(security)>).

## What is ChatGPT Function Calling / Structured Data?

If you've never heard about function calling, that's not surprising because OpenAI has marketed it extremely poorly. In the [same June announcement](https://openai.com/blog/function-calling-and-other-api-updates) as `gpt-3.5-turbo-0613`, OpenAI described function calling as:

> Developers can now describe functions to gpt-4-0613 and gpt-3.5-turbo-0613, and have the model intelligently choose to output a JSON object containing arguments to call those functions. This is a new way to more reliably connect GPT's capabilities with external tools and APIs.
>
> These models have been fine-tuned to both detect when a function needs to be called (depending on the user’s input) and to respond with JSON that adheres to the function signature. Function calling allows developers to more reliably get structured data back from the model.

Let's discuss the function calling example OpenAI gives in the blog post. After the user asks your app "What’s the weather like in Boston right now?":

1. Your app pings OpenAI with a `get_current_weather` function schema and decides if it's relevant to the user's question. If so, it returns a JSON dictionary with the data extracted, such as `location` and the `unit` for temperature measurement based on the location.
2. Your app (_not_ OpenAI) pings a different service/API to get more realtime metadata about the `location`, such as `temperature`, that a pretrained LLM could not know.
3. Your app passes the function schema with the realtime metadata: ChatGPT then converts it to a more natural humanized language for the end user.

So here's some background on "function calling" as it's a completely new term of art in AI that _didn't exist_ before OpenAI's June blog post (I checked!). This broad implementation of function calling is similar to the flow proposed in the original [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) paper as a "tool" such as `Search` or `Lookup` with parametric inputs such as a search query, which can be also done to perform [retrieval-augmented generation](https://research.ibm.com/blog/retrieval-augmented-generation-RAG) (RAG).

OpenAI's motivation for adding function calling was likely due to the extreme popularity of tools such as [LangChain](https://www.langchain.com) and [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) at the time. This may seem like a petty snipe from someone who has had [many issues with LangChain](https://minimaxir.com/2023/07/langchain-problem/), but in November OpenAI actually [deprecated](https://platform.openai.com/docs/api-reference/chat/create#chat-create-function_call) the `function_calling` parameter in favor of to `tool_choice`, matching LangChain's verbiage. But what's done is done and the term "function calling" is stuck forever especially now that competitors such as [Anthropic Claude](https://docs.anthropic.com/claude/docs/claude-2p1-guide#experimental-tool-use) and [Google Gemini](https://cloud.google.com/vertex-ai/docs/generative-ai/multimodal/function-calling) are also calling the workflow that term.

I am not going to play the SEO game and will not call it function calling. I'll call it what the quoted description from the blog post did: **structured data**, because that's the real value of this feature and OpenAI did a product management disservice trying to appeal to the AI hypebeasts.

Going back to the ~~function calling~~ structured data demo, we can reduce that flow by saying that step #1 (extracting location data and returning it formatted as JSON) is for working with structured _output_ data, and step #3 (providing ChatGPT with temperature data to humanize it) is for working with structured _input_ data. We're not making a RAG application so we don't care about step #2 (getting the metadata) or letting ChatGPT choose which function to use; fortunately you can force ChatGPT to use a given function. The function schema for the `get_current_weather` function in the announcement example is defined as:

Ew. It's no wonder why this technique hasn't become more mainstream.

## Simplifying Schema Input/Output With Pydantic

ChatGPT's structured data support requires that you create your schema using the [JSON Schema](https://json-schema.org) spec, which is more commonly used for APIs and databases rather than AI projects. As you can tell from the `get_current_weather` example above, the schema is complex and not fun to work with manually.

Fortunately, there's a way to easily generate JSON Schemas in the correct format in Python: [pydantic](https://docs.pydantic.dev/latest/), an extremely popular parsing and validation library which has its own [robust](https://github.com/pydantic/pydantic/blob/main/pydantic/json_schema.py) implementation of automatic [JSON Schema](https://docs.pydantic.dev/latest/concepts/json_schema/) generation.

A simple pydantic schema to have ChatGPT give an integer answer to a user query, plus, to make things interesting, also able to identify the name of the ones digit based on its answer, would be:

The resulting JSON Schema `parameters`:

The OpenAI API [pipeline](https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models) for providing structured data to the model is somewhat bespoke and requires a few changes from the typical ChatGPT API completion endpoint. To simplify things, I added ChatGPT structured data support to [simpleaichat](https://github.com/minimaxir/simpleaichat), my Python package for easily interfacing with ChatGPT. [^2] To minimize code the user needs to input, simpleaichat uses the schema name as the `name` in the JSON Schema and the schema docstring as the `description`. If you're keen-eyed you may have noticed there's a redundant `title` field in the pydantic schema output: simpleaichat also strips that out for consistency with OpenAI's examples.

[^2]: No, this blog post isn't a ploy to covertly promote my own Python library: it does genuinely save a lot of boilerplate code over the base ChatGPT API and this post is long-enough as-is.

If you wanted to query ChatGPT with the `answer_question` schema above (and have your OpenAI API key as the `OPENAI_API_KEY` enviroment variable!), you can do the following to generate output according to the schema:

And there you go! The `answer` is a JSON integer, the answer is one-off from the correct value [while driving](https://www.distance.to/San-Francisco/Los-Angeles), and it correctly identified the name of the ones digit in its own answer! [^3]

[^3]: If you swapped the order of the `answer` and the `one_digits` fields in the schema, then the model returns `{"ones_name": "miles", "answer": 382}` because it didn't get the hint from the answer!

Schemas don't have to be complex to be effective. Let's reimplement the Python palindrome question we did earlier with a single-field schema:

Note that unlike the raw ChatGPT answer, this response from the ChatGPT API only includes the code, which is a major plus since it means you receive the response much faster and cheaper since fewer overall tokens generated! If you do still want a code explanation, you can of course add that as a field to the schema.

As a bonus, forcing the output to follow a specific schema serves as an additional defense against [prompt injection attacks](https://www.wired.com/story/chatgpt-prompt-injection-attack-security/) that could be used to reveal a secret system prompt or [other shenanigans](https://www.businessinsider.com/car-dealership-chevrolet-chatbot-chatgpt-pranks-chevy-2023-12), since even with suggestive user prompts it will be difficult to get ChatGPT to disregard its schema.

pydantic exposes [many datatypes](https://docs.pydantic.dev/latest/concepts/fields/) for its `Field` which are compatable with JSON Schema, and you can also specify constraints in the `Field` object. The most useful ones are:

- `str`, can specify `min_length`/`max_length`
- `int`, can specify `min_value`/`max_value`
- `list` with a datatype, can specify `min_length`/`max_length`

Pydantic has a lot of support for valid forms of JSON Schema, but it's hard to infer how good these schema will work with ChatGPT since we have no idea how it learned to work with JSON Schema. Only one way to find out!

## Testing Out ChatGPT's Structured Data Support

From the demos above, you may have noticed that the `description` for each `Field` seems extraneous. It's not. The `description` gives ChatGPT a hint for the desired output for the field, and can be handled on a per-field basis. Not only that, the _name_ of the field is itself a strong hint. The _order_ of the fields in the schema is even more important, as ChatGPT will generate text in that order so it can be used strategically to seed information to the other fields. But that's not all, you can still use a ChatGPT system prompt as normal for _even more_ control!

It's prompt engineering all the way down. OpenAI's implementation of including the function is mostly likely appending the JSON Schema to the system prompt, perhaps with a command like `Your response must follow this JSON Schema.`. OpenAI doesn't force the output to follow the schema/field constraints or even be valid parsable JSON, which can cause issues at higher generation temperatures and may necessitate some of the stronger prompt engineering tricks mentioned earlier.

Given that, let's try a few more practical demos:

### Two-Pass Generation

One very important but under-discussed aspect of large-language models is that it will give you statistically "average" answers by default. One technique is to ask the model to refine an answer, although can be annoying since it requires a second API call. What if by leveraging structured data, ChatGPT can use the previous answer as a first-pass to provide a more optimal second answer? Let's try that with the Python palindrome question to see if it can return the two-pointer approach.

Also, the `Field(description=...)` pattern is becoming a bit redundant, so I added a `fd` alias from simpleaichat to it to minimize additional code.

Works great, and no tipping incentive necessary!

### Literals and Optional Inputs

OpenAI's structured data example uses a more complex schema indicating that `unit` has a fixed set of potential values (an [enum](https://en.wikipedia.org/wiki/Enumerated_type)) and that it's an optional field. Here's a rough reproduction of a pydantic schema that would generate the `get_current_weather` schema from much earlier:

This uses a `Literal` to force output between a range of values, which can be invaluable for hints as done earlier. The `= None` or a `Optional` typing operator gives a hint that the field is not required which could save unnecessary generation overhead, but it depends on the use case.

### Structured Input Data

You can provide structured input to ChatGPT in the same way as structured output. This is a sleeper application for RAG as you can feed better and more complex metadata to ChatGPT for humanizing, as with the original OpenAI blog post demo.

One famous weakness of LLMs is that it gives incorrect answers for simple mathematical problems due to how tokenization and memorization works. If you ask ChatGPT `What is 223 * -323?`, it will tell you `-72229` no matter how many times you ask, but the correct answer is `-72029`. Can type hints give more guidance?

For simpleaichat, structured input data works mostly the same way as structured output data, but you can use a pydantic object as the model input!

Yay, and it was still able to infer it was a multiplication operation without an explicit user request! Although it still doesn't work as well with larger numbers.

### Nested Schema

One of the other reasons pydantic is popular is that it allows nesting schemas. Fortunately, the subsequent JSON Schema output does respect nesting. Does ChatGPT?

The simple use case with ChatGPT structured data to use nesting is if you want to get a `list` of structured data objects. Let's say you want to create dialogue between two AI people about a completely nonsensical topic. We'll have to create a `Chat` object and include it in a function, plus some system prompt guidance and constraints. How silly can we make it?

ChatGPT _really_ wanted those $500 tips.

### Unions and Chain of Thoughts

I saved the best for last, and this structured data approach combines many of the techniques used earlier in this post like a [video game final boss](https://tvtropes.org/pmwiki/pmwiki.php/Main/FinalExamBoss).

One of the oldest pre-ChatGPT tricks for getting a LLM to perform better is to let it think. "Let's think step by step" is the key prompt, which allows the LLM to reason in a [chain of thought](https://arxiv.org/abs/2201.11903). We already did this a one-step version with the Python palindrome structured data example to successfully get optimized code, but we can do a lot more.

We'll now introduce the `Union` typing operator, which specifies the list of data types that the field can be, e.g. `Union[str, int]` means the output can be a `str` or `int`. But if you use the `Union` operator on a _nested class_, then many more options open as the model can choose from a set of schemas!

Let's make a few to allow ChatGPT to make _and qualify_ thoughts before returning a final result.

Therefore, for each reasoning, the model can pick one of the 3 schemas, although it will require a robust system prompt for it to behave in the order we want.

Lastly, we need a good question to stump the AI. A [popular Tweet](https://twitter.com/abacaj/status/1737206667387850936) from this week pointed out that even GPT-4 can comically fail if you ask it a brainteaser that it cannot have memorized, such as `23 shirts take 1 hour to dry outside, how long do 44 shirts take?`. Only one way to find out.

Unfortunately, all of this complexity makes the results unstable with `gpt-3.5-turbo` so instead I use GPT-4 Turbo / `gpt-4-1106-preview`.

Not bad! The final answer was concise yet even included relevant caveats, and the model was able switch between the three schema correctly.

How about another brainteaser? There is an infamous "[sister logic puzzle](https://www.reddit.com/r/LocalLLaMA/comments/18kpolm/that_sister_logic_puzzle_is_fairly_useless/)" used to test out up-and-coming open-source large language models:

In this case the AI may have gone _too_ meta, but it still arrived at the correct answer.

That said, GPT-4 is known for handling these types of difficult abstract questions without much effort, but it's still interesting to see how successfully it can "think."

## Structured Data With Open-Source LLMs

Speaking of open-source large language models, they have been growing in efficiency to the point that some can actually perform _better_ than the base ChatGPT. However, very few open-source LLMs explicitly claim they intentionally support structured data, but they're smart enough and they have logically seen enough examples of JSON Schema that with enough system prompt tweaking they should behave. It's worth looking just in case OpenAI has another [existential crisis](https://nymag.com/intelligencer/2023/11/why-was-sam-altman-fired-as-ceo-of-openai.html) or if the quality of ChatGPT [degrades](https://twitter.com/deliprao/status/1736978250717450481).

[Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-v0.1), the new darling of open-source LLMs, apparently has structured data support [on par with ChatGPT itself](https://twitter.com/robertnishihara/status/1734629320868687991). Therefore, I tried the latest [Mistral 7B official Instruct model](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2) with a quantized variant via [LM Studio](https://lmstudio.ai) (`mistral-7b-instruct-v0.2.Q6_K.gguf`), to see if it can handle my `answer_question` function that ChatGPT nailed. The system prompt:

And then asking `How many miles is it from San Francisco to Los Angeles?` while seting `temperature` to `0.0`:

Close enough! Unfortunately after testing the optimized Python palindrome schema, it ignored the schema completely, so this approach may only work for simple schema if the model isn't explicitly finetuned for it.

## What's Next For Structured Data in AI?

Most of these well-performing examples were done with the "weak" GPT-3.5; you of course can use GPT-4 for theoretically better results, but the cost efficiency of structured data with just the smaller model is hard to argue against (although the Python beach volleyball dialogue could benefit from a larger model).

Structured data and system prompt engineering saves a lot and time and frustration for working with the generated text as you can gain much more determinism in the output. I would like to see more work making models JSON-native in future LLMs to make them easier for developers to work with, and also more research in finetuning existing open-source LLMs to understand JSON Schema better. There may also be an opportunity to build LLMs using other more-efficient serialization formats such as [MessagePack](https://msgpack.org/index.html).

At OpenAI's November [DevDay](https://devday.openai.com), they also introduced [JSON Mode](https://platform.openai.com/docs/guides/text-generation/json-mode), which will force a normal ChatGPT API output to be in a JSON format without needing to provide a schema. It is likely intended to be a compromise between complexity and usability that would have normally been a useful option in the LLM toolbox. Except that in order to use it, you are _required_ to use prompt engineering by including "JSON" in the system prompt, and if you don't also specify a field key in the system prompt (the case in the documentation example), the JSON will contain a _random_ key. Which, at that point, you're just implementing a less-effective structured data schema, so why bother?

There is promise in constraining output to be valid JSON. One new trick that the open-source [llama.cpp](https://github.com/ggerganov/llama.cpp) project has popularized is [generative grammars](https://github.com/ggerganov/llama.cpp/tree/master/grammars), which constrain the LLM generation ability to only output according to specified rules. There's latency overhead with that technique though, so it will be interesting to watch how that space develops.

Despite the length of this blog post, there's still so much more than can be done with schemas: pydantic's documentation is very extensive! I've been working with structured data for LLMs [ever since GPT-2](https://github.com/minimaxir/gpt-2-keyword-generation) with mixed success since the base models weren't good enough, but now with ChatGPT, I think there's a time for a new approach to AI text generation, and I'll keep [simpleaichat](https://github.com/minimaxir/simpleaichat) up-to-date for it.

> You can view the Jupyter Notebooks used to generate all the structured data outputs in [this GitHub Repository](https://github.com/minimaxir/chatgpt-structured-data).
