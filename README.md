# ask-timeout-protocol

一个 DSH（DeepSeek Harness）技能：**提问超时/无人应答时，绝不替用户做决定。**

它解决一个具体而昂贵的失误：`ask_user_question` 超时后，agent 把沉默当成同意，
顺手把"要不要重启 / 要不要推送 / 要不要删"这类不可逆的事做完——用户回来时，
世界已经被改过，而他失去了先看一眼的机会。

## 它做什么

- 给出"什么算超时"的判定：`timed` 返回 `pending`、稍后到达的 `answer_to_pending_question`、`selected` 为空（用户跳过）、无回答者的 blocking error、明确 deny，以及 agent 自己冒出的"等不到所以先这样"。
- 按**猜错的代价**分三档（可逆 / 半可逆 / 不可逆），只有可逆档允许按推荐继续。
- 规定挂留卡的格式：问题 / 选项 / 推荐与理由 / 等待的代价 / 一直不回会怎样。
- 规定出口：决策必须出现在最终可见回复里、用编号列出，用户回 `2` 或 `Q2-2` 即可。
- 规定恢复：回数字/关键词即执行；回"你定"只授权卡内推荐项；工具提问最多重发一次。

## 安装

DSH 0.2.1-rc.5 没有 `dsh skill add`；技能由 `$DSH_HOME/skills`（默认 `~/.dsh/skills`）或项目 `.dsh/skills` 扫描加载。

POSIX：

```bash
DSH_DIR="${DSH_HOME:-$HOME/.dsh}"
mkdir -p "$DSH_DIR/skills"
git clone --depth 1 https://github.com/berserkross/dsh-skill-ask-timeout.git \
  "$DSH_DIR/skills/ask-timeout-protocol"
```

Windows PowerShell：

```powershell
$dshDir = if ($env:DSH_HOME) { $env:DSH_HOME } else { Join-Path $env:USERPROFILE '.dsh' }
New-Item -ItemType Directory -Force "$dshDir\skills" | Out-Null
git clone --depth 1 https://github.com/berserkross/dsh-skill-ask-timeout.git "$dshDir\skills\ask-timeout-protocol"
```

更新（已是 git 克隆）：

```bash
git -C "${DSH_HOME:-$HOME/.dsh}/skills/ask-timeout-protocol" pull --ff-only
```

卸载：删除 `$DSH_HOME/skills/ask-timeout-protocol`（或项目 `.dsh/skills/ask-timeout-protocol`）。
验证：新会话的技能列表里出现 `ask-timeout-protocol`，或在 DSH 设置的技能页确认。

> 仓库名是 `dsh-skill-ask-timeout`，技能名是 `ask-timeout-protocol`；安装目录建议用后者。

## 什么时候会被用到

在已经发出请示之后：工具超时 / 无人应答 / 审批无响应 / 用户长时间不在，或者你发现自己想说"等不到所以先这样"。
它不决定"要不要问"；要不要问由 `clarify-first` / `plan-confirm` 这类协议决定。

## 设计取向

- **宁可停着，也不要猜。** 等待的成本是一次回复，猜错的成本是一次返工加一次信任损失。
- **不新增决策。** 它不改变"要不要问"，只管"问了没人答"；同一决策最多重问一次。
- **宿主/用户规则优先。** 本协议只约束等待后的行为，不产生新的授权；C 盘写入、安装、删除、推送等要求先批准的规则仍然有效。
- **可执行。** 每条规则都写成能直接照做的一句话，而不是态度描述。

## 许可

MIT
