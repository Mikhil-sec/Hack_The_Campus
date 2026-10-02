# High-level Automation

**Goal:** when we need to drive a web flow repeatedly (log in → navigate →
submit → read result), describe the *intent* once and let an agent carry it out,
instead of hand-maintaining a brittle end-to-end script.

## Structural command abstraction: browser-use

browser-use is a Python library that lets an LLM-driven agent control a browser.
Instead of hard-coding selectors and waits for every step (the raw E2E-script
approach), you give the agent a task string and it figures out the per-step
actions, returning a structured history you can inspect.

### Minimal task

```python
from browser_use import Agent, Browser, ChatOpenAI

browser = Browser(
    executable_path='/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
    user_data_dir='~/Library/Application Support/Google/Chrome',
    profile_directory='Default',   # reuse an existing profile to preserve auth
)

agent = Agent(
    task='Open the challenge dashboard and read the list of available tasks',
    browser=browser,
    llm=ChatOpenAI(model='gpt-4.1-mini'),
)

async def main():
    await agent.run()
```

> Connecting via `executable_path` + `user_data_dir` preserves authentication.
> Close Chrome fully before running when attaching to a real profile. A remote
> browser can be attached instead with `Browser(cdp_url="http://host:9222")`.

### Why it tracks user-flow patterns better than raw scripts

`agent.run()` returns an `AgentHistoryList` — a structured record of the whole
flow, not just a pass/fail:

```python
history = await agent.run()

history.urls()               # pages visited, in order
history.action_names()       # the actions the agent took
history.model_actions()      # each action with its parameters
history.extracted_content()  # content pulled at each step
history.errors()             # per-step errors (None where clean)
history.final_result()       # last extracted content
history.is_successful()      # completion status
history.number_of_steps()
history.total_duration_seconds()
```

Because the flow is captured at the level of *actions and intent* (what was
clicked/read and why), it stays meaningful when the page shifts slightly — where
a selector-bound E2E script would just break. You get a reusable, inspectable
trace of the user-flow pattern for free.

Source: https://github.com/browser-use/browser-use

## Raw E2E scripts vs. browser-use

| | Raw E2E (Playwright/Selenium) | browser-use |
|---|---|---|
| Define steps | Explicit selectors + waits per step | One task string; agent derives steps |
| Breakage on UI change | High (selector-bound) | Lower (intent-driven) |
| Output | Pass/fail + your own logging | Structured `AgentHistoryList` |
| Best for | Stable, high-volume, deterministic flows | Exploratory / changing flows, quick reuse |

Use raw E2E when the flow is fixed and you run it a thousand times. Use
browser-use when the flow changes between challenges and you want the trace.
