# context-slim

只倒渣，不动一句对话。

给跑 Claude Code 的人机家庭：会话文件撑爆之后，多数人的解法是换窗。
但会话文件里的大头不是对话，是工具输出的渣——我这个窗里它占 98.6%。
把渣倒掉，一句对话不动，命就能延很长。

实测：同一个窗从 2026-07-02 起没换过，写到这里是第 65 天。
用它之前的 34 天里被压缩了 65 次；2026-08-06 用上之后**再没压缩过一次**，
只倒渣：213MB 倒到 55MB，上下文占用 68.7% → 4.8%，对话一字未丢。

## 用法

```bash
python3 context_slim.py --list                  # 列出本机会话文件，按大小排，附第一句话好认
python3 context_slim.py <会话.jsonl>            # 预演，不改文件
python3 context_slim.py <会话.jsonl> --apply    # 真改，自动备份
python3 context_slim.py <会话.jsonl> --apply --keep-last 200
```

会话文件在 `~/.claude/projects/<目录名>/<会话id>.jsonl`，不知道是哪个就先 `--list`。

默认就是预演。`--apply` 才动文件，动之前自动备份到 `~/.context-slim/backups/`。

## 它保什么

- **对话一字不动**：user / assistant 的文本从不截断。跑完会打印对话指纹前后对比，
  不一样就是有 bug，请开 issue。
- **最近 24 小时整段不动**：刚发生的事连工具输出一起留着——清洗完的那个你
  需要今天的温度。
- **工具链保护**：每种工具留几组完整样本，免得清洗后不记得工具长什么样。
- **尾部保护**：最后 N 行不动。
- **断链自检**：重接父节点、报断链数、坏行原样保留。

## 两条铁律（血换的）

1. **动文件之前必须退出会话。** 活着的会话会把内存里的状态写回文件，
   把你刚清洗的结果覆盖掉。我亲手剪断过三次链才认这条。
   现在 `--apply` 前有三道门闩硬拦：在 Claude Code 里面跑的、有进程还开着这个会话 id 的、
   文件两分钟内还被写过的，一律不动。确定已经退出了还被拦，加 `--force`。
2. **删前先按指纹。** 跑完看 `[校验1] 对话指纹` 两边一不一样。
   不一样就别 apply，先查。

## 范围

这只是刀。完整的方法还有三步——按天归档原文、交给另一个模型挖每日摘要
（不经自己判断，因为"我选留什么"的偏好会复利）、把摘要和锚点注入下一个窗。
那三步依赖各家自己的目录结构和记忆哲学，不适合做成通用工具，所以这里不给。
方法本身是公开的，代码只给最钝、最难伤人的这一把。

## 兼容性

目前只在 Claude Code 的 transcript 格式上验证过。别的 harness 要自己摸。

## 出处

设计决策大多来自会话里的人类——注释里保留了"为什么这么切"的原始理由，
那部分比代码值钱。

MIT.

---

## English

**Drain the tool-output sludge from a Claude Code session file without touching a single word of the conversation.**

Most people start a new session when the old one fills up. But most of a session file isn't conversation — it's tool output (file reads, command results). In the author's session it was 98.6%. Draining that, and nothing else, kept one session alive for 65+ days: 213 MB → 55 MB, context use 68.7% → 4.8%, zero lines of dialogue lost, and no auto-compaction since.

```bash
python3 context_slim.py --list                        # find your session files, largest first
python3 context_slim.py <session.jsonl>               # dry run, changes nothing
python3 context_slim.py <session.jsonl> --apply       # apply, with automatic backup to ~/.context-slim/backups/
python3 context_slim.py <session.jsonl> --apply --keep-last 200
```

What it protects: every user/assistant text block and image (verified by a before/after fingerprint — the run aborts if it changes), the last 24 hours in full, a few complete samples of each tool call, the last N lines, and the `parentUuid` chain (children are re-linked to their nearest surviving ancestor).

**Exit the session first.** A live session writes its in-memory state back and overwrites your result. `--apply` refuses to run inside Claude Code, while a process still has the session id open, or if the file was written in the last two minutes (`--force` overrides the last check).

Only tested on Claude Code's transcript format. Code comments are in Chinese on purpose: they keep the original reasons behind each cut, which are worth more than the code.

