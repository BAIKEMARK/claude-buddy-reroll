# Claude Code Buddy Reroll

刷 Claude Code `/buddy` 宠物的脚本，基于社区逆向分析。

## 关键：根据安装方式选择运行时

Claude Code 的宠物由 `hash(userID + SALT)` 确定性生成，但哈希函数**取决于安装方式**：

| 安装方式 | 运行时 | 哈希函数 | 运行命令 |
|---------|--------|---------|---------|
| `npm install -g @anthropic-ai/claude-code` | Node.js | **FNV-1a** | `node buddy-reroll.js` |
| 官方 native 安装 | Bun | **Bun.hash** | `bun buddy-reroll.js` |

**用错了运行时，搜出来的 UID 写进去宠物会完全不对！**

### 如何确认自己的安装方式

```bash
# 查看安装方式
cat ~/.claude.json | grep installMethod
```

- `"installMethod": "global"` → npm 安装 → 用 `node`
- `"installMethod": "native"` → 官方安装 → 用 `bun`

## 使用方法

### 查看当前宠物

```bash
# npm 用户
node buddy-reroll.js --check <你的userID或accountUuid>

# native 用户
bun buddy-reroll.js --check <你的userID或accountUuid>
```

userID 位置：
- npm 用户：`~/.claude.json` 中的 `userID` 字段
- OAuth 登录用户：`~/.claude.json` 中的 `oauthAccount.accountUuid` 字段（如果手动设置了的话）

### 搜索指定宠物

```bash
# npm 用户
node buddy-reroll.js --species cat --rarity legendary --eye '✦' --hat crown

# native 用户
bun buddy-reroll.js --species cat --rarity legendary --eye '✦' --hat crown
```

### 写入配置

1. 编辑 `~/.claude.json`
2. 将 `userID` 或 `oauthAccount.accountUuid` 替换为搜到的 uid
3. 删除 `companion` 字段（让系统重新生成名字和性格）
4. 重启 Claude Code，输入 `/buddy` 领养

## 全部选项

```
--species <name>       物种: duck, goose, blob, cat, dragon, octopus, owl,
                       penguin, turtle, snail, ghost, axolotl, capybara,
                       cactus, robot, rabbit, mushroom, chonk
--rarity <name>        最低稀有度: common, uncommon, rare, epic, legendary
--eye <char>           眼睛样式: · ✦ × ◉ @ °
--hat <name>           帽子: none, crown, tophat, propeller, halo, wizard,
                       beanie, tinyduck
--shiny                要求闪光版（1% 概率，搜索时间会更长）
--min-stats [value]    要求所有属性 >= 指定值（默认 90）
--max <number>         最大迭代次数（默认 5000万）
--count <number>       搜索结果数量（默认 3）
--check <uid>          查看指定 uid 对应的宠物
```

## 示例

```bash
# 刷传奇鸭子（native 用户）
bun buddy-reroll.js --species duck --rarity legendary --shiny

# 刷高属性龙（npm 用户）
node buddy-reroll.js --species dragon --min-stats 80

# 随机看看当前是什么宠物
node buddy-reroll.js --check 0bdbf7581277af73e4bcbc281099a77700feaaa99e8ade02cd1c9e2816507c4c
```

## 参考

- [博客文章：Reverse Engineering Claude Code's 2026 April Fools](https://variety.is/posts/claude-code-buddies/)
- [Linux.do 帖子：Claude Code /buddy 宠物系统逆向分析](https://linux.do/t/topic/1871870)
- SALT: `friend-2026-401`（来源：Claude Code v2.1.89 源码泄露）
