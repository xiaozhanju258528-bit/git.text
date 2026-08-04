# Python 学习计划与方法

> 针对：有 Dart/Flutter 编程基础、带团队、务实风格的学习者
> 预计周期：10-14 周达到独立产出水平

---

## 写在前面：你有编程基础，别从零开始学

你写过 Dart，懂类型系统、异步、面向对象。Python 对你来说不是"学一门新语言"，而是"换一种表达方式"。以下计划跳过了"什么是变量""什么是循环"这种废话，直奔 Python 的特性和工程实践。

**核心原则：写代码 > 看教程。每个阶段必须有产出物，不做练习题。**

---

## 阶段一：基础语法（1-2 周）

### 目标
能用 Python 写出等价于你 Dart 代码的东西。

### 重点知识

| 知识点 | 和 Dart 的区别 | 学习优先级 |
|--------|---------------|-----------|
| 缩进语法 | Dart 用花括号，Python 用缩进 | 高 |
| 动态类型 + 类型提示 | Dart 是强类型，Python 运行时动态但支持静态提示 | 高 |
| 列表/字典/集合 | 对应 List/Map/Set，但语法更简洁 | 高 |
| 字符串处理 | f-string 是 Python 的杀手锏 | 高 |
| 函数定义 | def、默认参数、*args/**kwargs | 中 |
| 文件 IO | with open() 上下文管理 | 中 |
| 模块导入 | import 机制和 Dart 完全不同 | 中 |

### 产出物
写一个 CLI 小工具——比如批量重命名文件、整理下载目录。用到文件 IO、字符串处理、命令行参数。

### 资源
- **官方教程**：https://docs.python.org/zh-cn/3/tutorial/ （过一遍，跳过你已懂的）
- **速查**：https://learnxinyminutes.com/docs/python3/ （有编程基础看这个最快）

---

## 阶段二：进阶特性（2-3 周）

### 目标
理解 Python 的"Pythonic"写法，写出地道的代码而不是"用 Python 语法写的 Dart 代码"。

### 重点知识

**1. 装饰器（decorator）**
Python 最强大的特性之一。本质是高阶函数，类似 Dart 的注解但更灵活。
```python
def log_time(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time() - start:.2f}s")
        return result
    return wrapper

@log_time
def process_data():
    ...
```

**2. 生成器与迭代器**
惰性求值，处理大数据集时不用一次性加载。Dart 有 `yield` 但 Python 用得更广泛。
```python
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()
```

**3. 上下文管理器（with 语句）**
资源管理的标准姿势，__enter__/__exit__ 协议。

**4. 推导式**
列表/字典/集合推导式，Python 的标志性写法。
```python
# Python
squares = {x: x**2 for x in range(10) if x % 2 == 0}

# 等价 Dart 代码要写好几行
```

**5. 闭包与作用域**
LEGB 规则（Local → Enclosing → Global → Built-in）。

### 产出物
用装饰器写一个简单的 API 请求缓存工具，或者用生成器实现一个日志流式解析器。

### 资源
- ** Fluent Python **（流畅的 Python）第 1 版前几章——这本书是 Python 进阶圣经
- **Real Python** 网站的进阶教程：https://realpython.com/

---

## 阶段三：工程化（2-3 周）

### 目标
从"脚本能跑"升级到"项目能维护"。这一步是区分业余和专业的分水岭。

### 重点知识

**1. 虚拟环境管理**
```bash
# 标准库方案
python -m venv .venv
source .venv/bin/activate

# 更现代的方案（推荐）
uv venv          # 速度比 venv 快 10-100 倍
uv pip install
```

**2. 包管理与依赖锁定**
- `pyproject.toml` 是现代 Python 项目的标准配置
- 用 `uv` 或 `poetry` 管理依赖，别裸用 pip + requirements.txt

**3. 类型提示（Type Hinting）**
你是强类型语言的过来人，这个应该很有共鸣：
```python
from typing import Optional

def find_user(user_id: str) -> Optional[dict]:
    ...
```
配合 `mypy` 或 `pyright` 做静态检查，接近 Dart 的类型安全体验。

**4. 测试**
```python
# pytest 是事实标准
def test_add():
    assert add(1, 2) == 3
    assert add(-1, 1) == 0
```

**5. 日志与异常**
- 用 `logging` 模块，别用 `print` 调试
- 异常处理：EAFP（Easier to Ask Forgiveness than Permission）风格

**6. 项目结构**
```
my_project/
├── pyproject.toml
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── core.py
│       └── utils.py
├── tests/
│   ├── test_core.py
│   └── test_utils.py
└── README.md
```

### 产出物
把阶段一、二写的小工具重构为标准项目结构，加类型提示和测试。

### 资源
- **uv 官方文档**：https://docs.astral.sh/uv/ （现代 Python 包管理器，强烈推荐）
- **pyproject.toml 指南**：https://packaging.python.org/en/latest/guides/
- **pytest 文档**：https://docs.pytest.org/

---

## 阶段四：实战项目（4-6 周）

### 目标
做真东西。选 1-2 个项目完整做完，从设计到部署。

### 项目建议（按你的背景排序）

**项目 A：团队工具类（推荐首选）**
- 自动生成周报/站会纪要的 CLI 工具
- 代码质量检查脚本（扫描指定目录，输出报告）
- API 接口文档生成器
- 这些能直接用在团队工作中，有实际反馈

**项目 B：Web API 服务**
- 用 FastAPI 写一个 RESTful API
- 对接数据库（PostgreSQL + SQLAlchemy）
- 加上认证、日志、错误处理
- 部署到服务器
- 你做过移动端，理解 API 设计，这个上手快

**项目 C：数据处理**
- 用 Pandas 处理 CSV/Excel 数据
- 做一个数据清洗 + 报表生成的工具
- 如果团队有数据统计需求，这个很实用

### 关键原则
- 用 Git 管理版本，每个功能一个分支
- 写 README，说明怎么装、怎么跑
- 写测试，至少覆盖核心逻辑
- 用 type hint，跑 mypy 检查

---

## 阶段五：方向深耕（持续）

### 目标
选定一个方向深入，其余了解即可。

### 方向选择

| 方向 | 适合场景 | 核心技术栈 | 学习曲线 |
|------|---------|-----------|---------|
| Web 后端 | API 服务、全栈开发 | FastAPI / Django + SQLAlchemy + PostgreSQL | 平缓 |
| 数据分析 | 报表、数据清洗、可视化 | Pandas + NumPy + Matplotlib | 中等 |
| AI/LLM | AI 应用开发 | LangChain / OpenAI SDK + 向量数据库 | 陡峭 |
| 自动化/运维 | 脚本工具、CI/CD | asyncio + subprocess + APScheduler | 平缓 |

### 我的建议
基于你的背景（技术团队负责人 + Flutter 开发），**Web 后端（FastAPI）是最佳切入点**：
1. 你懂 API 设计，FastAPI 的路由写法和 Dart 的路由很像
2. 能快速给团队搭工具，有实际产出
3. Python 在后端是主流，学了不亏
4. 之后想扩展到 AI 方向，FastAPI + LLM SDK 是标配组合

---

## 学习方法

### 1. 别看视频，写代码
视频教程效率太低。有编程基础的人，看文档 + 写代码比看视频快 3 倍。

### 2. 读优秀的开源代码
找一两个你感兴趣的小型 Python 项目（1000 行以内），读源码。看别人怎么组织代码、怎么处理异常、怎么写测试。

推荐：
- https://github.com/tiangolo/fastapi （看 fastapi/cli/ 目录，代码量小且质量高）
- https://github.com/httpie/cli （CLI 工具的最佳实践）

### 3. 每天写，别攒着
每天 30 分钟写代码 > 周末突击 5 小时。编程是肌肉记忆。

### 4. 用 AI 辅助但不依赖
- 让 AI 解释你不理解的代码片段
- 让 AI review 你写的代码，问"这段代码怎么写更 Pythonic"
- **不要**让 AI 直接帮你写完整个功能——你学不到东西

### 5. 遇到问题先查官方文档
Python 官方文档质量非常高，比大部分中文教程准确。养成查文档的习惯。

---

## 开发环境推荐

```
编辑器：VSCode + Python 扩展 + Pylance
包管理：uv（替代 pip + venv，快得多）
格式化：ruff format（替代 black，更快）
Lint：ruff check（替代 flake8 + pylint）
类型检查：pyright（Pylance 自带）
测试：pytest
```

---

## 速查：Dart → Python 对照

| 概念 | Dart | Python |
|------|------|--------|
| 变量声明 | `var x = 1;` | `x = 1` |
| 类型声明 | `String name = 'a';` | `name: str = 'a'` |
| 空安全 | `String? name;` | `name: str \| None` |
| 异步 | `Future<T>` / `await` | `asyncio` / `await` |
| 列表 | `List<int>` | `list[int]` |
| 字典 | `Map<String, int>` | `dict[str, int]` |
| 枚举 | `enum Color { red }` | `class Color(Enum): red = 1` |
| 类构造 | `ClassName(this.x)` | `def __init__(self, x): self.x = x` |
| 主函数 | `void main() {}` | `if __name__ == '__main__':` |
| 包导入 | `import 'package:foo/foo.dart';` | `import foo` 或 `from foo import bar` |

---

## 常见坑（Dart 开发者转 Python）

1. **缩进是语法**：混用 tab 和空格会报错，统一用 4 个空格
2. **没有 `new` 关键字**：直接 `MyClass()` 就行
3. **self 不是 this**：实例方法第一个参数必须是 self，调用时不用传
4. **没有接口关键字**：用抽象类或 Protocol 实现
5. **私有变量靠约定**：`_name` 表示私有，不是强制的
6. **GIL 限制**：Python 多线程不能真正并行 CPU 密集任务，用多进程替代
7. **== 和 is**：== 比较值，is 比较身份（类似 Dart 的 identical）

---

## 检查清单：每个阶段结束自测

**阶段一完成后能回答：**
- [ ] f-string 怎么用？格式化浮点数保留两位小数？
- [ ] list 和 tuple 的区别？什么时候用哪个？
- [ ] with open() 和直接 open() 的区别？
- [ ] *args 和 **kwargs 分别是什么？

**阶段二完成后能回答：**
- [ ] 写一个计时装饰器
- [ ] 生成器和列表推导式的区别？什么时候用生成器？
- [ ] 上下文管理器的 __enter__ 和 __exit__ 怎么工作？
- [ ] 列表推导式和生成器表达式的语法区别？

**阶段三完成后能回答：**
- [ ] pyproject.toml 里怎么声明依赖？
- [ ] 类型提示怎么写 Optional 和 Union？
- [ ] pytest 怎么跑？fixture 是什么？
- [ ] logging 模块的基本用法？

**阶段四完成后：**
- [ ] 有一个完整的、带测试的、有 README 的项目
- [ ] 项目用 Git 管理，有清晰的 commit 历史
- [ ] 能给别人解释这个项目怎么跑起来

---

## 总结

5 个阶段，10-14 周，核心就一句话：**别看教程，写代码。** 每个阶段有产出物，用真项目练手。你是带团队的人，学完之后能把 Python 用到团队工具链里——这才是学一门语言的意义。
