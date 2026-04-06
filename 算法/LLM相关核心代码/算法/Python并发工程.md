# Python并发工程

LLM 应用开发经常涉及高并发调用 API。

## 核心考点

- **高频题：** 使用 `asyncio` 实现一个支持限流（Rate Limiting）的并发 API 调用器。
- **考核点：** 异步编程能力、对 `Semaphore`（信号量）的理解。
- **建议：** 熟悉 `asyncio.Semaphore` 的使用方式，以及 `gather` 调度多个任务的写法。

## Asyncio 并发调用 API（带限流）

**背诵要点：** 使用 `asyncio.Semaphore` 控制并发数。

```python
import asyncio

async def call_llm_api(semaphore, prompt):
    # 使用信号量控制并发数
    async with semaphore:
        # 模拟 API 网络请求
        print(f"Generating for: {prompt}")
        await asyncio.sleep(1)
        return f"Response for {prompt}"

async def main():
    prompts = ["你好", "介绍下 FastAPI", "什么是 RAG"]
    # 限制同时只有 2 个任务在跑
    sem = asyncio.Semaphore(2)

    tasks = [call_llm_api(sem, p) for p in prompts]
    results = await asyncio.gather(*tasks)
    print(results)

# 运行：asyncio.run(main())
```
