---
layout: post
title: Open AI Assistant can conduct Google Search 
---

 OpenAI gained their global recognition through [ChatGPT](https://openai.com/blog/introducing-chatgpt-and-whisper-apis), a well-known chatbot. However, the capabilities of a Large Language Model (LLM) extend far beyond human-agent messaging. The real measure of an LLM's effectiveness is in the variety of tasks it can perform for users. As of December 12th, 2023, although ChatGPT inherently suports Bing search, the Open AI API does not offer this functionality directly. Nevertheless, it is possible to enable the Open AI assistant to conduct Google searches by using [function calling](https://platform.openai.com/docs/guides/function-calling). This article provides a guide on implementing this in Typescript.    

## Prerequisites:
- An [Open AI API](https://openai.com/blog/openai-api) key 
- A [Google custom search](https://developers.google.com/custom-search/v1/overview) API key and search engine id.

You can test your custom search engine by using curl.

```curl
curl --location 'https://www.googleapis.com/customsearch/v1?key=Google_API_Key&cx=Google_CSE_ID&&q=123'
```

## Step 1:
Create an Open AI assistant and enable function calling. An example can be found below using GPT-4. 

```json
{
    "id": "asst_000000000000000000000",
    "object": "assistant",
    "name": "ABC AI Assistant",
    "model": "gpt-4-1106-preview",
    "instructions": "ABC AI Assistant is a professional helper agent for clients using ABC platform. The agent maintains a professional demeanor.",
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "get_ABC_info",
                "description": "Get information about ABC",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {
                            "type": "string",
                            "description": "query about ABC"
                        }
                    },
                    "required": [
                        "query"
                    ]
                }
            }
        },
    ]
    // Other assistant properties...
}
```

In my case, I aim to develop a support agent tailored for ABC users to facilitate easier adoption. This agent will be programmed to automatically invoke the function whenever it receives a query about ABC. For this purpose, I've opted for a broad description of the function's invoking criteria. It's important to note that this approach is based on my personal preference and varies according to the specific conditions under which one wishes a function to be invoked. 

As shown below, GPT-4 invokes `get_ABC_info` when a question about ABC is present.

![Function Calling]({{ BASE_PATH }}/images/function_calling.png)


## Step 2:

Define a function `query_google`.

```ts
async function query_google(
  query: string
): Promise<string> {
  console.log("Searching google with: " + query);

  const googleSearchEndpoint = "https://www.googleapis.com/customsearch/v1";

  const apiKey = process.env.GOOGLE_API_KEY;
  const cseId = process.env.GOOGLE_CSE_ID;

  const params = {
    key: apiKey,
    cx: cseId,
    q: query,
  };

  try {
    const { data } = await axios.get(googleSearchEndpoint, { params });
    if (data && data.items && data.items.length > 0) {
      const url = data.items[0].link;
      console.log("Google result: ", url);
      return url;
    }
    return '';
  } catch (err) {
    console.error("Unable to search google: ", err);
    return '';
  }
}

```

Note that `query` is set by GPT-4, based on user questions and their contexts.

## Step 3:

In my situation, I need GPT-4 to generate responses based on the content extracted from links provided by Google searches. To achieve this, I'll implement an additional function, `get_url_content`, designed to convert a URL into text. 

```ts
async function get_url_content(url: string) {
  try {
    const { data } = await axios.get(url);
    const $ = cheerio.load(data);

    const text = $.text()
      .split("\n")
      .filter((line) => line.trim() !== "")
      .join("\n");
    return text;
  } catch (err) {
    console.error(`Unable to fetch ${url}: `, err);
    return null;
  }
}
```
**Note** [Cheerio](https://cheerio.js.org/) is a HTML parsing utility, aka jQuery but for nodejs.

## Step 4:

Create a new [run](https://platform.openai.com/docs/api-reference/runs) on a [thread](https://platform.openai.com/docs/api-reference/threads). And then invoke `query_google` and `get_url_content` when a run reaches `require_action` status. Next, submit the result of `get_url_content` back to open AI. Finally, list all messages after the run becomes `completed`.

**Full example**

```ts 
  const openai = new OpenAI();
  // Create a new thread.
  const thread = await openai.beta.threads.create();
  const thread_id = thread.id;

  // Create a new message. 
  await openai.beta.threads.messages.create(thread_id, {
    role: "user",
    content: "Tell me about ABC",
  });

  // Create a new run.
  const run = await openai.beta.threads.runs.create(thread_id, {
    assistant_id: process.env.ASSISTANT_ID,
  }); 
  const run_id = run.id;
  let {status, tool_calls} = run;

  while (
    status === "queued" ||
    status === "in_progress" ||
    status === "requires_action"
  ) {
    if (status === "requires_action") {

      await openai.beta.threads.runs.submitToolOutputs(thread_id, run_id, {
        tool_outputs: await Promise.all(
          tool_calls.map(async (tc) => {
            const url = await query_google(JSON.parse(tc.function.arguments).query);
            const content = await get_url_content(url);
            return ({
              tool_call_id: tc.id,
              output: content,
            });
        })),
      });
    }

    await new Promise((res) => setTimeout(res, 1000));

    ({ status, tool_calls } = await openai.beta.threads.runs.retrieve(thread_id, run_id));
  }

  const messages = await openai.beta.threads.messages.list(thread_id);

  console.log(messages);
```

Hooray! Your open AI assistant just answers a question using information from Google search.  

## Considerations

1. This guide exemplifies **retrieval-augmented generation, RAG**. It merges GPT-4's ability to autonomously generate queries with the dependendability of a search engine. Consequently, GPT-4 can provide responses enriched by information updated beyond Apr 2023.  
2. In contrast to document retrieval, another prevalent RAG technique, using a search engine allows access to much broader range of documents without consuming excessive tokens. 
3. Given that my objective is to create a support agent for ABC company, I have tailored the Google custom search engine to exclusively search our support website. This approach might also be applicable to your use case.

Enjoy coding!

