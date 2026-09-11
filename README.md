https://whimsical.com/

https://excalidraw.com/


https://www.hackermate.in/   -> sih hackthon skill mates finder


https://www.analyticsvidhya.com/-> good fast api



https://www.amanailab.com/   ml interview


https://www.interviewquery.com/  ml interview



# LLM API Calling Methods — Python Notes

There are several common ways to call an LLM from Python. The main ones are:

1. OpenAI SDK → `chat.completions.create()`
2. OpenAI SDK → `responses.create()`
3. LangChain → `.invoke()`
4. LangChain → `.ainvoke()`
5. LangChain → `.stream()`
6. LangChain → `.astream()`
7. LangChain → `.batch()`
8. LangChain → `.abatch()`
9. Direct HTTP API call

---

## 1. OpenAI SDK — `chat.completions.create()`

The traditional Chat Completions API.

**Client**

```python
import os
from openai import AsyncOpenAI

client = AsyncOpenAI(
    api_key=os.getenv("AICREDICTS_API_KEY"),
    base_url=os.getenv("AICREDICTS_BASE_API"),
)
```

**Call**

```python
response = await client.chat.completions.create(
    model="openai/gpt-4o-mini",
    messages=[
        {"role": "user", "content": "hi sumit"}
    ],
)
```

Because `AsyncOpenAI` is being used, `await` is required.

**Access the response**

```python
text = response.choices[0].message.content
print(text)
```

Structure:

```text
response
   └── choices
        └── [0]
             └── message
                  └── content
```

**Complete example**

```python
response = await client.chat.completions.create(
    model="openai/gpt-4o-mini",
    messages=[
        {"role": "user", "content": "What is Python?"}
    ],
)

answer = response.choices[0].message.content
print(answer)
```

---

## 2. OpenAI SDK — `responses.create()`

A newer OpenAI API style.

**Call**

```python
response = await client.responses.create(
    model="openai/gpt-4o-mini",
    input="What is Python?",
)
```

**Access the response**

```python
answer = response.output_text
print(answer)
```

**Key difference**

| Chat Completions | Responses API |
|---|---|
| `response.choices[0].message.content` | `response.output_text` |

**Complete example**

```python
response = await client.responses.create(
    model="openai/gpt-4o-mini",
    input="What is Python?",
)

answer = response.output_text
print(answer)
```

---

## 3. LangChain — `.invoke()`

First, create a LangChain model:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="openai/gpt-4o-mini",
    api_key=os.getenv("AICREDICTS_API_KEY"),
    base_url=os.getenv("AICREDICTS_BASE_API"),
)
```

**Call**

```python
response = llm.invoke("What is Python?")
```

**Access the response**

LangChain's chat model returns an `AIMessage`:

```python
answer = response.content
print(answer)
```

Structure:

```text
response
   └── AIMessage
         └── content
```

**Complete example**

```python
response = llm.invoke("What is Python?")

answer = response.content
print(answer)
```

---

## 4. LangChain — `.ainvoke()`

Asynchronous version of `.invoke()`.

```python
response = await llm.ainvoke("What is Python?")

answer = response.content
print(answer)
```

**Complete example**

```python
response = await llm.ainvoke("Explain Python in one sentence.")
print(response.content)
```

**Sync vs. async**

```python
response = llm.invoke(...)          # synchronous
response = await llm.ainvoke(...)   # asynchronous
```

---

## 5. LangChain — `.stream()`

Use `.stream()` to receive the answer progressively instead of waiting for the full response.

```python
for chunk in llm.stream("Tell me a story"):
    print(chunk.content, end="")
```

Each `chunk` contains part of the generated response — access it via `chunk.content`.

---

## 6. LangChain — `.astream()`

Asynchronous version of `.stream()`.

```python
async for chunk in llm.astream("Tell me a story"):
    print(chunk.content, end="")
```

**Complete example**

```python
async for chunk in llm.astream("Explain FastAPI."):
    print(chunk.content, end="")
```

---

## 7. LangChain — `.batch()`

Use `.batch()` for multiple independent inputs.

```python
responses = llm.batch([
    "What is Python?",
    "What is FastAPI?",
    "What is Docker?",
])
```

`responses` is now a list.

**Access responses**

```python
for response in responses:
    print(response.content)
```

Or by index:

```python
answer1 = responses[0].content
answer2 = responses[1].content
answer3 = responses[2].content
```

Structure:

```text
responses
   ├── [0] → AIMessage → content
   ├── [1] → AIMessage → content
   └── [2] → AIMessage → content
```

---

## 8. LangChain — `.abatch()`

Asynchronous version of `.batch()`.

```python
responses = await llm.abatch([
    "What is Python?",
    "What is FastAPI?",
    "What is Docker?",
])

for response in responses:
    print(response.content)
```

---

## 9. Direct HTTP Request

You don't need the OpenAI SDK or LangChain — you can call an OpenAI-compatible HTTP endpoint directly, e.g. with `httpx`.

```python
import httpx
import os

api_key = os.getenv("AICREDICTS_API_KEY")
base_url = os.getenv("AICREDICTS_BASE_API")

response = await httpx.AsyncClient().post(
    f"{base_url}/chat/completions",
    headers={
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    },
    json={
        "model": "openai/gpt-4o-mini",
        "messages": [
            {"role": "user", "content": "What is Python?"}
        ],
    },
)
```

**Access the response**

```python
data = response.json()
answer = data["choices"][0]["message"]["content"]
print(answer)
```

---

## Quick Comparison

| Method | Library | Async? | Response text |
|---|---|---|---|
| `chat.completions.create()` | OpenAI SDK | `await` | `response.choices[0].message.content` |
| `responses.create()` | OpenAI SDK | `await` | `response.output_text` |
| `.invoke()` | LangChain | No | `response.content` |
| `.ainvoke()` | LangChain | Yes | `response.content` |
| `.stream()` | LangChain | No | `chunk.content` |
| `.astream()` | LangChain | Yes | `chunk.content` |
| `.batch()` | LangChain | No | `response.content` |
| `.abatch()` | LangChain | Yes | `response.content` |
| HTTP `POST` | `httpx`/`requests` | Depends | `data["choices"][0]["message"]["content"]` |

---

## Mental Model

```text
                  YOUR PYTHON CODE
                        │
          ┌─────────────┴─────────────┐
          │                           │
     OpenAI SDK                    LangChain
          │                           │
   ┌──────┴──────┐             ┌──────┴─────────┐
   │             │             │                │
Chat Completions Responses   invoke()       stream()
   │             │             │                │
create()       create()     ainvoke()        astream()
   │             │             │                │
   └─────────────┴─────────────┴────────────────┘
                        │
                     LLM API
                        │
                      Model
```

---

## The Response Objects Are Different

This is one of the most important things to remember.

**OpenAI Chat Completions**

```python
response = await client.chat.completions.create(...)
response.choices[0].message.content
```

**OpenAI Responses API**

```python
response = await client.responses.create(...)
response.output_text
```

**LangChain**

```python
response = llm.invoke(...)
response.content
```

**LangChain streaming**

```python
for chunk in llm.stream(...):
    chunk.content
```

**Direct HTTP**

```python
data = response.json()
data["choices"][0]["message"]["content"]
```

---

## Which One Should I Use?

**Directly using an OpenAI-compatible API**
```python
await client.chat.completions.create(...)
# or, if supported:
await client.responses.create(...)
```

**Application uses LangChain**
```python
llm.invoke(...)
# or async:
await llm.ainvoke(...)
```

**Need streaming**
```python
llm.stream(...)
# or:
llm.astream(...)
```

**Need many independent prompts**
```python
llm.batch(...)
# or:
await llm.abatch(...)
```

**Need maximum control**

Call the HTTP API directly with `httpx`.

---

## One Final Rule to Remember

Don't choose `.invoke()` just because it looks simpler — look at **what object you have**.

- If you have `client = AsyncOpenAI(...)` → use OpenAI SDK methods: `await client.chat.completions.create(...)` or `await client.responses.create(...)`
- If you have `llm = ChatOpenAI(...)` → use LangChain methods: `llm.invoke(...)` or `await llm.ainvoke(...)`

In short:

```text
AsyncOpenAI  →  create()
ChatOpenAI   →  invoke()
```
