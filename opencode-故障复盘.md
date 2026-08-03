# opencode TUI 黑屏故障复盘

**日期**：2026-08-03 | **环境**：WSL (Ubuntu) + CMD

---

## 1. 问题现象

在 WSL 终端中执行 `opencode` 命令后：

- 终端窗口瞬间变黑，清屏后无任何输出
- 无光标、无报错、无响应
- 只能 `Ctrl+C` 强行退出

但之前更新 opencode 后曾经使用正常。

---

## 2. 排查过程

### 第一步：验证程序本身是否能运行

```bash
opencode --help
```

输出正常，程序二进制本身没问题。

### 第二步：检查终端环境

```bash
echo $TERM       # xterm-256color — 正常
echo $COLORTERM  # 空
```

终端类型设置正确。当时初步怀疑是 CMD 的 conhost 不支持 TUI 渲染。

### 第三步：用非 TUI 模式排除终端问题

```bash
opencode run "say hi"
```

输出：

```
Error: Unexpected error
no such column: name
```

**关键发现**：当绕过 TUI 直接跑核心逻辑时，爆出了真正的错误——SQLite 数据库 `no such column: name`。这说明不是终端渲染的问题，而是程序在数据库操作阶段崩溃了。

---

## 3. 根因分析

### 崩溃链

```
opencode
  → TUI 框架接管终端（清屏，进入全屏模式）
    → 程序初始化，读取本地数据库
      → 数据库中某个表缺少 `name` 列
        → 抛出异常，程序崩溃
          → TUI 已清屏，错误信息被吞
            → 用户只看到黑屏
```

### 为什么会出现 schema 不匹配？

opencode 升级到新版本后，数据库表结构发生了变化（增加了 `name` 字段），但**旧数据库文件没有自动迁移**。新代码去查 `name` 列，旧数据库里没有这个列，SQLite 直接报错。

### 为什么之前能用？

刚更新完时，如果 opencode 在你那次使用中成功完成了数据库迁移，或者还没触发到读取那个特定表的路径，就能正常工作。之后某次操作触发了 `name` 列的查询，就崩了。数据库迁移失败是不稳定的——不一定会立刻暴露。

---

## 4. 解决方案

**删除旧数据库，让程序重建**：

```bash
# 备份（以防万一）
cp ~/.local/share/opencode/opencode.db ~/.local/share/opencode/opencode.db.bak

# 删除
rm ~/.local/share/opencode/opencode.db

# 验证
opencode run "say hi"   # 正常输出 →
opencode                # TUI 正常启动
```

**为什么删库可行？** opencode 的本地数据库只存 session 历史、使用统计等**本地缓存数据**，不存 API key、provider 配置等核心配置（那些在别的配置文件里）。删了之后程序启动时检测不到数据库，自动按最新 schema 重建，所以不会丢重要数据。

---

## 5. 经验总结

| 经验 | 说明 |
|------|------|
| **黑屏不等于终端问题** | TUI 应用崩溃时，错误信息会被清屏覆盖。先绕过 TUI 测试核心功能 |
| **用 `--help` / `run` 先排除基础问题** | 非交互模式能直接暴露程序层面的错误，不依赖终端渲染 |
| **`no such column` 就是 schema 版本不一致** | SQLite 数据库升级后如果没迁移，删库重建往往是最快的方案 |
| **TUI 应用排查第一原则** | 优先级：核心逻辑 > 数据库/配置 > 终端渲染。不要上来就怀疑终端 |
| **本地缓存大胆删** | 区分清楚哪些是本地的、可重建的数据（db、cache），哪些是配置（credentials、config），前者删了不心疼 |

---

## 6. 速查卡

以后 opencode 或者其他类似 CLI/TUI 工具出现黑屏/卡死：

```
1. opencode --help        → 确认二进制正常
2. opencode run "test"    → 绕过 TUI，看核心逻辑报什么错
3. 如果报 SQLite 错误    → 删 ~/.local/share/opencode/*.db
4. 如果报其他错误        → 查 ~/.config/opencode/ 配置
5. 以上都正常才查终端    → $TERM、终端模拟器类型
```

**最后才怀疑终端。** 这是这次最大的教训。
