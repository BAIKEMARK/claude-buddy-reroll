---
name: buddy-reroll
description: "This skill should be used when the user asks to 'reroll buddy', 'change buddy', '刷宠物', '换宠物', 'reset buddy', 'reroll /buddy', '想要一只新的宠物', '看看有什么宠物', or mentions Claude Code buddy/companion customization. Helps users search for a specific buddy combination and apply it to their config."
---

# Claude Code 宠物刷取

帮用户刷取自定义的 Claude Code `/buddy` 宠物组合。

## 原理

宠物由 `hash(userID + SALT)` 确定性生成。SALT = `friend-2026-401`。哈希函数取决于安装方式：

| 安装方式 | 运行时 | 哈希函数 | 运行命令 |
|---------|--------|---------|---------|
| `npm install` | Node.js | FNV-1a | `node scripts/buddy-reroll.js` |
| 官方 native | Bun | Bun.hash | `bun scripts/buddy-reroll.js` |

**用错运行时结果完全不对！**

## Step 0: 检查版本

`/buddy` 功能需要 Claude Code **v2.1.88** 及以上版本。

运行 `claude --version` 检查当前版本。如果低于 v2.1.88，提示用户先更新：

- npm 用户：`npm update -g @anthropic-ai/claude-code`
- native 用户：`claude update`

版本不够则停止后续流程，等用户更新完成后再继续。

## Step 1: 判断安装方式

读 `~/.claude.json` 的 `installMethod` 字段：
- `"global"` → npm → 用 `node`
- `"native"` → 官方 → 用 `bun`

## Step 2: 查看当前宠物

读 `~/.claude.json`，宠物种子字段优先级：
- `oauthAccount.accountUuid` 存在 → 用这个
- 否则 → 用 `userID`

```bash
node scripts/buddy-reroll.js --check <值>
```

展示当前宠物的物种、稀有度、眼睛、帽子、闪光、属性和名字。

## Step 3: 选择宠物配置

**核心原则：一次展示全部选项，让用户直接用自然语言告诉想要什么组合。不要用 AskUserQuestion 选择类别（选项数限制无法展示全部）。**

### 3.1 展示完整选项清单

输出完整的宠物选项参考，一次性全部展示，引导用户去图鉴页面看预览。格式示例：

```
🐾 宠物配方大全

👉 不确定选什么？打开图鉴页面看看每个宠物的样子：
   https://baikemark.github.io/claude-buddy-reroll/

【稀有度】
  普通 ★ (60%) | 非凡 ★★ (25%) | 稀有 ★★★ (10%) | 史诗 ★★★★ (4%) | 传奇 ★★★★★ (1%)

【物种（18种）】
  鸭子  鹅  史莱姆  猫  龙  章鱼  水豚  机器人
  猫头鹰  企鹅  乌龟  蜗牛  幽灵  美西螈  仙人掌  兔子  蘑菇  胖猫

【眼睛】
  默认·  星星✦  晕眩×  大眼◉  机械@  惊讶°

【帽子】（普通稀有度固定无帽子）
  无  皇冠  高礼帽  螺旋桨  光环  巫师帽  毛线帽  小鸭子

【闪光】1% 概率，搜索时间更长，名字会标记为"格外特别"

告诉我你想要的组合，比如"传奇猫 皇冠 星星眼 闪光"。
想看某个物种的 ASCII 样子，也可以直接说。
```

**注意：必须把所有选项一次性全部展示，不要分页、不要分轮、不要用 AskUserQuestion 选。**

### 3.2 用户选择

用户直接用自然语言回答想要的组合（比如"传奇猫 皇冠 星星眼"、"我想要一只闪光的龙"）。如果用户对某个物种感兴趣想看长什么样，读取 `references/sprites.md` 渲染对应 ASCII 精灵。

如果用户没说稀有度，默认不限制。如果没说眼睛/帽子，默认不限制（随机）。普通稀有度自动跳过帽子。

### 3.3 闪光和属性

用 AskUserQuestion 一次性弹出闪光和属性两个问题（左右切换）：

- **闪光**：不要 / 要闪光（1% 概率，搜索更久）
- **属性要求**：不限（默认≥90） / 高属性（≥95） / 极限（=100）

如果用户在 3.2 中已经说了要闪光或指定了属性，跳过此步。

### 3.4 汇总确认

根据用户选择汇总配置，展示后开始搜索。

## Step 4: 搜索目标宠物

把中文选项转为英文参数：

**中文→英文映射：**

物种：鸭子=duck, 鹅=goose, 史莱姆=blob, 猫=cat, 龙=dragon, 章鱼=octopus, 猫头鹰=owl, 企鹅=penguin, 乌龟=turtle, 蜗牛=snail, 幽灵=ghost, 美西螈=axolotl, 水豚=capybara, 仙人掌=cactus, 机器人=robot, 兔子=rabbit, 蘑菇=mushroom, 胖猫=chonk

眼睛：默认=·, 星星=✦, 晕眩=×, 大眼=◉, 机械=@, 惊讶=°

帽子：无=none, 皇冠=crown, 高礼帽=tophat, 螺旋桨=propeller, 光环=halo, 巫师帽=wizard, 毛线帽=beanie, 小鸭子=tinyduck

稀有度：普通=common, 非凡=uncommon, 稀有=rare, 史诗=epic, 传奇=legendary

```bash
node scripts/buddy-reroll.js --species cat --rarity legendary --eye '✦' --hat crown
```

其他参数：`--shiny`（闪光）、`--min-stats 95`（高属性）、`--min-stats 100`（极限属性）

搜索结果展示后，用 AskUserQuestion 让用户选择要应用哪一个 uid（通常 3 个结果，刚好 ≤4 个选项）。

## Step 5: 处理名字和性格

**重要：不要直接删除 companion 块！先读取当前 companion 的 name 和 personality，然后询问用户。**

读 `~/.claude.json` 的 `companion` 块，获取当前名字（如有）。

使用 AskUserQuestion 询问：

```
你的宠物当前名字是"咪呜"。
你想怎么处理名字和性格？
  1. 保留原名（不改名字和性格）
  2. 自定义名字（支持中文和英文）
  3. 让 Claude Code 重新生成（基于新宠物的稀有度、物种、属性自动生成）
```

**选项说明：**

A. **保留原名** — 只替换 uid，不动 companion 块。如果 companion 块不存在则保留为空。

B. **自定义名字** — 用户输入自定义名字和性格（支持中文和英文）。更新 companion 块：
```json
"companion": {
  "name": "用户输入的名字",
  "personality": "用户输入的性格描述",
  "hatchedAt": 1775015300069
}
```
如果用户只输入了名字没输入性格，让 Claude Code 自动生成性格。如果用户提供了性格描述就用用户的。

C. **让 Claude Code 重新生成** — 删除整个 companion 块，重启后 Claude Code 会调用 LLM 根据新宠物的稀有度、物种、属性和 156 词灵感库自动生成名字和性格。

## Step 6: 写入配置

只改 **`~/.claude.json`** 一个文件。

**改哪个字段：**
- `oauthAccount.accountUuid` 存在 → 改这个
- 否则 → 改 `userID`

完整流程：读配置 → 确定种子字段 → 替换为搜索结果 → 根据用户选择更新/保留/删除 companion → 用 AskUserQuestion 确认写入（确认 / 取消） → 提示重启并 `/buddy`

## 属性说明

调试(DEBUGGING)、耐心(PATIENCE)、混乱(CHAOS)、智慧(WISDOM)、毒舌(SNARK)。
每个宠物有 1 个最高属性和 1 个最低属性。属性影响 LLM 生成的名字和性格。

## 注意

- 脚本路径：`scripts/buddy-reroll.js`（相对于此 skill）
- 闪光(shiny)目前无视觉差异，但在名字生成时会标记为"格外特别"
- 改 userID/accountUuid 不影响 API key、对话历史、本地配置
