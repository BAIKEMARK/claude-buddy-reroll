# Claude Code 宠物刷取工具 (Buddy Reroll)

自定义 Claude Code `/buddy` 宠物的物种、稀有度、眼睛、帽子和属性。基于社区逆向分析。

> **选择困难？** 打开 [图鉴页面](https://github.com/BAIKEMARK/claude-buddy-reroll/blob/main/index.html) 浏览所有宠物。

## 安装

```bash
# 一键安装（推荐）
npx skills add BAIKEMARK/claude-buddy-reroll -g
```

安装后，在 Claude Code 里说"刷宠物"、"换宠物"、"看看有什么宠物"等即可触发。

也可以手动安装：将仓库根目录下的 `SKILL.md` + `references/` + `scripts/` 复制到 `~/.claude/skills/buddy-reroll/`。

## 关键：根据安装方式选择运行时

Claude Code 的宠物由 `hash(userID + SALT)` 确定性生成，但哈希函数**取决于安装方式**：

| 安装方式 | 运行时 | 哈希函数 | 运行命令 |
|---------|--------|---------|---------|
| `npm install` | Node.js | **FNV-1a** | `node buddy-reroll.js` |
| 官方 native | Bun | **Bun.hash** | `bun buddy-reroll.js` |

**用错运行时搜出来的 UID 写进去宠物会完全不对！**

### 确认安装方式

```bash
cat ~/.claude.json | grep installMethod
```

- `"global"` → npm → 用 `node`
- `"native"` → 官方 → 用 `bun`

## 使用方式

### 通过 Skill（安装后）

直接在 Claude Code 对话中用自然语言：

- "我想刷一只传奇猫，皇冠，星星眼"
- "帮我换只龙，要闪光的"
- "看看当前是什么宠物"
- "刷宠物" → 会展示所有可选组合让你选

### 通过脚本（手动运行）

```bash
# 查看当前宠物
node buddy-reroll.js --check <你的userID>

# 搜索指定宠物
node buddy-reroll.js --species cat --rarity legendary --eye '✦' --hat crown

# native 用户用 bun
bun buddy-reroll.js --species duck --rarity legendary --shiny
```

## 可选内容

| 参数 | 说明 |
|------|------|
| `--species <名称>` | 物种：duck(鸭子), cat(猫), dragon(龙), octopus(章鱼), owl(猫头鹰), penguin(企鹅), turtle(乌龟), snail(蜗牛), ghost(幽灵), axolotl(美西螈), capybara(水豚), cactus(仙人掌), robot(机器人), rabbit(兔子), mushroom(蘑菇), chonk(胖猫) |
| `--rarity <等级>` | 最低稀有度：common(普通), uncommon(非凡), rare(稀有), epic(史诗), legendary(传奇) |
| `--eye <样式>` | 眼睛：·(默认) ✦(星星) ×(晕眩) ◉(大眼) @(机械) °(惊讶) |
| `--hat <帽子>` | 帽子：none(无), crown(皇冠), tophat(高礼帽), propeller(螺旋桨), halo(光环), wizard(巫师帽), beanie(毛线帽), tinyduck(小鸭子) |
| `--shiny` | 闪光版（1% 概率，搜索更久） |
| `--min-stats <值>` | 要求所有属性 ≥ 指定值（默认 90） |
| `--max <次数>` | 最大搜索次数（默认 5000 万） |
| `--count <数量>` | 搜索结果数量（默认 3） |
| `--check <uid>` | 查看指定 uid 对应的宠物 |

## 修改原理

只改 `~/.claude.json` 一个文件：

1. 将搜索到的 uid 写入 `userID` 或 `oauthAccount.accountUuid`
2. 删除或自定义 `companion` 块（名字和性格）
3. 重启 Claude Code，输入 `/buddy` 领养

## 参考

- [图鉴页面（浏览器预览所有宠物）](https://github.com/BAIKEMARK/claude-buddy-reroll/blob/main/index.html)
- [博客文章：Reverse Engineering Claude Code's 2026 April Fools](https://variety.is/posts/claude-code-buddies/)
- [Linux.do 帖子：Claude Code /buddy 宠物系统逆向分析](https://linux.do/t/topic/1871870)
