# ask-timeout-protocol

一个 DSH（DeepSeek Harness）技能：**提问超时/无人应答时，绝不替用户做决定。**

它解决一个具体而昂贵的失误：`ask_user_question` 超时后，agent 把沉默当成同意，
顺手把"要不要重启 / 要不要推送 / 要不要删"这类不可逆的事做完——用户回来时，
世界已经被改过，而他失去了先看一眼的机会。

## 它做什么

- 给出"什么算超时"的判定：工具超时、审批被拒、已问未答、以及 agent 自己冒出的"等不到所以先这样"。
- 按**猜错的代价**分三档（可逆 / 半可逆 / 不可逆），只有可逆档允许按推荐继续。
- 规定挂留卡的格式：问题 / 选项 / 推荐与理由 / 等待的代价 / 一直不回会怎样。
- 规定出口：决策必须出现在最终可见回复里、用编号列出，用户回一个数字即可。
- 规定恢复：回数字即执行；回"你定"视为授权并记为假设；工具提问最多重发一次。

## 安装

```bash
git clone https://github.com/berserkross/dsh-skill-ask-timeout.git \
  "$DSH_HOME/skills/ask-timeout-protocol"
```

Windows PowerShell：

```powershell
git clone https://github.com/berserkross/dsh-skill-ask-timeout.git "$env:USERPROFILE\Documents\Codex\.dsh\skills\ask-timeout-protocol"
```

技能目录下的 `SKILL.md` 就是全部内容，无需重启 DSH：技能目录是按需扫描的。

## 什么时候会被用到

凡是"必须请示才能动手"的场景：改代码、装卸插件、改配置、批量操作、部署、
任何删除/覆盖/推送/发布/安装，以及需要对外发请求或花钱的动作。

## 设计取向

- **宁可停着，也不要猜。** 等待的成本是一次回复，猜错的成本是一次返工加一次信任损失。
- **不新增提问。** 它不改变"要不要问"（那是 clarify-first / plan-confirm 的事），只管"问了没人答"。
- **可执行。** 每条规则都写成能直接照做的一句话，而不是态度描述。

## 许可

MIT
