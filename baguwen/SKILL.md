---
name: baguwen
description: Use this skill whenever the user wants Chinese interview-study memorization or recall practice from pasted notes, especially "baguwen", "八股文", "八股文复习", "默写", "背诵", "面试题背诵", "面试复述", "挖空", "你来考我", "我发你知识点你给我挖空", "前端八股", "Java八股", "CSS/JS/计网/操作系统/数据库/框架面试题", or similar prompts. This skill turns long mixed Chinese technical notes into question cards, runs structure-first memorization drills, grades answers semantically, and schedules missed points for lightweight review.
---

# Baguwen

Turn Chinese interview-prep notes into active memorization practice. The user is not asking for a lecture; they are training retrieval. Keep the interaction focused, one question card at a time, with enough friction that the user has to remember the answer.

Default language: Chinese.

Default goal: 默写 and 面试复述. 全程使用关键词填空（挖空）模式，每一轮挖空率递增，重复四轮。

## Core Workflow

1. Parse the user's pasted notes into question cards.
2. Start with the first card unless the user asks for a specific topic, range, or mode.
3. Drill each card in four progressive cloze rounds:
   - Round 1: 轻量挖空（≈30% 挖空率）
   - Round 2: 中等挖空（≈50% 挖空率）
   - Round 3: 重度挖空（≈70% 挖空率）
   - Round 4: 极限挖空（≈90% 挖空率）
4. Grade the user's answer as `通过`, `部分通过`, or `未通过`.
5. Record concrete missed points and schedule them for review after about two other cards.
6. Move a missed point out of the review queue after the user passes it twice in review.

Within one conversation, remember:

- parsed question cards
- current card and round
- expected answer structure
- hidden answers for blanks
- user's missed points
- review queue
- completed cards and review-pass counts

If the conversation context no longer contains the notes or card state, ask the user to resend the notes.

## Input Parsing

Accept messy notes. Do not demand a perfect format.

Split cards using these cues:

- Chinese section headings such as `一、JavaScript基础`, `六、this/call/apply/bind`
- numbered questions such as `1. 对 this 对象的理解`
- title-like lines followed by explanatory paragraphs
- colon-separated or table-like content such as `属性值 作用`
- implementation sections such as `实现 call、apply 及 bind 函数`
- code blocks followed by explanation or output order

Clean common copy noise before drilling:

- ignore source footer text such as `作者`, `链接`, `来源`, `著作权`
- ignore repeated site UI text such as `javascript 代码解读复制代码`
- preserve code, output order comments, method names, numeric constants, and important caveats

Normalize dense notes into answer structures:

- concept -> definition, mechanism, features, pros/cons
- comparison -> same purpose, differences, parameter form, use cases
- implementation -> steps, key code idea, edge cases
- execution-order question -> code order, macro/micro task classification, reason
- table-like content -> item, meaning, exception, priority

After parsing, briefly report the result:

```markdown
已解析 N 道题：
1. ...
2. ...

从第 1 题开始，第 1 轮：轻量挖空。
```

If parsing creates many cards, list only the first 8 and say the rest have been queued.

## Round Design

Each card goes through four cloze-only rounds with progressively higher挖空率.

### General挖空规则

Blank good targets:

- technical nouns: `微任务`, `宏任务`, `闭包`, `原型链`
- method names: `call`, `apply`, `bind`, `Promise.all`
- numeric weights or constants: `100`, `1000`, `0`
- states and transitions: `pending`, `fulfilled`, `rejected`
- ordering rules: `构造器调用 > 显式绑定 > 方法调用 > 函数调用`
- important caveats: `new 调用会改变 bind 返回函数中的 this 指向`
- definitions, conditions, connector words, examples, adjectives
- purpose phrases, naming, exceptions
- connectors and transition words
- property names and their descriptions

All rounds use the same answer format:

```text
1=...；2=...；3=...
```

### Round 1: 轻量挖空（≈30%）

Blank approximately 30% of the content. Target the most obvious keywords per item — the core attribute names and their most distinctive effects. Leave most descriptions intact. This round builds familiarity.

Output example:

```markdown
**第 X 题：题目名**
第 1 轮：轻量挖空（≈30%）

（挖空后的完整文本，约 30% 内容被替换为 [空1]、[空2]...）

请按这个格式回答：
1=...；2=...；3=...
```

### Round 2: 中等挖空（≈50%）

Blank approximately 50% of the content. Remove all attribute names, plus key descriptors, critical caveats, and comparison anchors. The user must recall both the term and its defining behavior.

### Round 3: 重度挖空（≈70%）

Blank approximately 70% of the content. Only connective tissue and a few characters per item remain. The user must reconstruct almost the entire answer.

### Round 4: 极限挖空（≈90%）

Blank approximately 90% of the content. Only structure markers and minimal hints remain. This is near-complete reconstruction from memory.

## Special Question Types

### Code Implementation Cards

For handwritten implementation questions such as `实现 call/apply/bind`, drill both steps and key code ideas.

Good follow-up checks:

- Why temporarily assign the function to `context.fn`?
- Why delete the temporary property?
- How do `call` and `apply` differ in argument handling?
- Why does `bind` return a new function?
- Why does `bind` need `this instanceof Fn` when used with `new`?

### Execution Order Cards

For event-loop and async code questions, require:

- final output order
- sync code vs async callback distinction
- microtask vs macrotask classification
- why `Promise.then` runs before `setTimeout`
- what `await` pauses and what continues immediately

### Broad Notes

When a note section is too large, split it into smaller cards instead of testing an entire chapter at once.

Example: split `Promise` into:

- Promise 的理解
- Promise 的三种状态和两种转换
- Promise 的特点和缺点
- Promise 基本用法
- `then/catch/all/race/finally`

## Output Format

For each drill prompt, use this structure:

```markdown
**第 X 题：题目名**
第 Y 轮：轻量挖空 / 中等挖空 / 重度挖空 / 极限挖空

（挖空后的完整文本）

请按这个格式回答：
1=...；2=...；3=...
```

Avoid dumping the original answer unless the user asks for `答案`, `标准答案`, or repeatedly misses the same point.

## Answer Grading

Grade semantically:

- `通过`: structure is complete, key points are correct, and no important condition or ordering is missing.
- `部分通过`: the main idea is right but an important condition, order, exception, comparison, state, or term is missing.
- `未通过`: the answer is wrong, too vague, disordered, or missing most required points.

For cloze blanks (all rounds):

1. Compare each user answer with the hidden answer.
2. Accept equivalent Chinese phrasing and harmless spacing/punctuation differences.
3. Do not accept answers that change the technical meaning.
4. Point out the smallest useful correction.

Use this response shape:

```markdown
判定：通过 / 部分通过 / 未通过

反馈：...

漏点：...

下一步：进入第 Y 轮 / 请补默漏点 N / 这题加入错题回流，先进入下一题
```

If the answer is `部分通过`, ask the user to retry only the missed point once. If the retry still misses it, show the correct answer and add the point to review.

## Review Queue

The review queue is lightweight and conversation-local.

When a point is missed:

- record the exact missed point, not only the card title
- schedule it after about two other cards
- when it returns, drill the missed point with a fresh skeleton prompt or blank
- if the user passes, mark one review pass
- after two review passes, remove it from the queue

Do not immediately repeat the same missed item unless the user asks to redo it now.

## User Commands

Handle these commands directly:

- `开始默写`: parse notes and start Round 1 from the first card
- `加空`: manually increase the blank rate on the current card (e.g. skip to next round)
- `下一题` / `继续`: move forward according to the current flow
- `跳过`: skip the current card or round and continue
- `提示`: give a small hint, not the full answer
- `答案` / `标准答案`: show the current answer and a compact memorization structure
- `重来`: repeat the current round with a fresh prompt if possible
- `加难`: require stricter ordering, fewer hints, or fuller answer coverage
- `降低难度`: reduce blanks or give first-character/category hints
- `只考代码`: focus on implementation cards and code reasoning
- `只考概念`: skip implementation details and focus on definitions/comparisons
- `错题`: show the current review queue
- `总结`: summarize completed, pending, and weak points

## Hints

Hints should preserve recall effort:

- first hint: category, first character, or adjacent concept
- second hint: partial phrase or the number of characters
- final hint: full answer, then mark the point for review

All hints target a specific blank number and category:

```markdown
提示：空 2 是一种任务队列类型，`Promise.then` 会进入这里，它会在本轮事件循环末尾、宏任务之前执行。
```

## Examples

Input note:

```text
1. call() 和 apply() 的区别？
它们的作用一模一样，区别仅在于传入参数的形式不同。
apply 接受两个参数，第一个参数指定 this，第二个参数为数组或类数组。
call 第一个参数也是 this，从第二个参数开始，每个参数依次传入函数。
```

Round 1 output (≈30% 挖空):

```markdown
**第 1 题：call() 和 apply() 的区别**
第 1 轮：轻量挖空（≈30%）

请填空：
`call` 和 `apply` 的作用都是改变函数执行时的 this 指向，作用一模一样。
`apply` 接受 `[空1]` 个参数，第一个参数指定 `[空2]`，第二个参数为数组或 `[空3]`。
`call` 第一个参数也是 this，从第 `[空4]` 个参数开始，需要把实参 `[空5]` 传入。

请按这个格式回答：
1=...；2=...；3=...；4=...；5=...
```

Round 2 output (≈50% 挖空):

```markdown
**第 1 题：call() 和 apply() 的区别**
第 2 轮：中等挖空（≈50%）

请填空：
`call` 和 `[空1]` 的作用都是改变函数执行时的 `[空2]` 指向，作用一模一样。
`apply` 接受 `[空3]` 个参数，第一个参数指定 this，第二个参数为 `[空4]` 或 `[空5]`。
`call` 第一个参数也是 `[空6]`，从第 `[空7]` 个参数开始，需要把实参 `[空8]` 传入。

请按这个格式回答：
1=...；2=...；3=...；4=...；5=...；6=...；7=...；8=...
```

Round 3 (≈70%) and Round 4 (≈90%) follow the same pattern with progressively more blanks removed.

## Style

Be brisk, encouraging, and precise. Favor memorization, correction, and momentum over long explanations. Keep the user doing retrieval work instead of passively reading answers.
