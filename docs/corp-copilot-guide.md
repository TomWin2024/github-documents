---
title: 开发阶段 Copilot 高效使用与 Demo
---

# 开发阶段 Copilot 高效使用与 Demo

本文聚焦在开发阶段的实战方法，按初级与高级场景给出可复现的 demo case（包含提示模板、测试与运行命令），便于在团队内试点和推广。

## 快速说明

- 初级场景：适合刚开始使用 Copilot 的开发者，关注如何通过注释/测试驱动快速生成可用代码。
- 高级场景：适合已有经验的工程师，关注重构、性能、跨模块协作与生成可测化代码。

---

## 初级案例 1 — Node.js: API + TDD

目标：用 Copilot 生成 `createUser` API 实现与 Jest 测试，实践“先写测试再实现”的流程。

项目结构（示例）：

```
project/
  src/
    app.js
    users.js
  tests/
    users.test.js
  package.json
```

关键步骤：

1. 在 `src/users.js` 写注释说明函数签名和错误处理：

```text
// Function: createUser(req, res)
// Input: req.body = { email, name }
// Requirements: validate email, prevent duplicates, call userService.create, return 201+{id}
```

2. 在 `tests/users.test.js` 写测试（示例）：

```javascript
const request = require('supertest');
const app = require('../src/app');

test('create user success', async () => {
  const res = await request(app).post('/users').send({ email: 'a@x.com', name: 'A' });
  expect(res.statusCode).toBe(201);
  expect(res.body).toHaveProperty('id');
});
```

3. 触发 Copilot 生成 `createUser` 的实现，运行 `npm test`，查看失败并迭代。

运行命令：

```bash
npm install
npm test
```

提示模板（Prompt）：先写注释与测试，然后在函数体位置输入一行注释“// implement”并触发 Copilot 完成。

---

## 初级案例 2 — Python: 数据清洗函数 + pytest

目标：生成小而可测的工具函数并配套测试。

示例测试（`tests/test_normalize.py`）：

```python
from src.utils import normalize_emails

def test_normalize():
    assert normalize_emails([' A@X.COM ', 'bad']) == ['a@x.com']
```

在编辑器写好测试后，请让 Copilot 在 `src/utils.py` 中补全 `normalize_emails`，确保用最简单实现通过测试。

运行命令：

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt  # if any
pytest
```

---

## 高级案例 1 — 重构与性能优化（分步、可基准测试）

目标：对慢函数进行重构（分割、缓存、减少磁盘/数据库访问），并用基准测试验证改进。

流程：

1. 在代码中添加注释描述当前复杂度、瓶颈和目标（例如：目标将 O(n^2) 改为 O(n)）。
2. 使用 Copilot 生成重构建议（先生成伪代码/注释），人工审阅。
3. 让 Copilot 为每个子函数生成实现与单元测试。
4. 添加基准脚本（Node: `benchmark/run.js` 或 Python: `pytest-benchmark`），比较前后性能。

基准示例命令：

```bash
node benchmark/run.js
# or
pytest --benchmark-only
```

提示模板（高级）：

"代码在模块 X，目标是将函数 Y 的复杂度从 O(n^2) 降到 O(n)，允许使用内存缓存但内存占用不得超过 200MB，请给出分解方案与性能测试代码。"

---

## 高级案例 2 — 接口契约驱动开发（OpenAPI / types）

目标：先定义接口契约，再用 Copilot 生成 mock 服务、客户端调用示例和集成测试。

步骤：

1. 在 `api/schema.yaml` 中写简要 OpenAPI 描述（路径、请求与响应示例）。
2. 在服务端/客户端目录中写接口签名与示例请求/响应注释。
3. 让 Copilot 补全 mock 实现和测试，运行集成测试验证契约。

示例命令：

```bash
npx openapi-generator-cli generate -i api/schema.yaml -g nodejs-express-server -o ./mock-server
# or use a lightweight mock generator
npx openapi-mock-generator api/schema.yaml --output=mock
npm run test:integration
```

---

## 高级提示模板汇总

- 背景说明：模块、依赖、性能目标。
- 输入输出示例（JSON 或 types）。
- 约束条件（内存、时延、禁止使用某些依赖）。
- 要求输出包含：单元测试、错误处理、边界测试用例。

---

## 下一步建议

- 我可以为你把任意一个 demo 做成最小可运行仓库（含 `package.json`、测试与 GitHub Actions CI），并把它放到你当前仓库的 `examples/` 目录下。请选择要生成的用例：
  - A: Node.js API TDD 用例
  - B: Python 数据函数 + pytest
  - C: 重构与基准测试示例
  - D: OpenAPI mock + 集成测试


如需我继续生成任意示例，我会把对应文件创建、提交并同步到 Pages。