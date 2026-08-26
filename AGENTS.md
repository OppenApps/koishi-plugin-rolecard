# koishi-plugin-rolecard 项目常驻指令

> 本文件是本仓库的常驻开发约定，适用于本仓库内所有会话与任务。全程使用简体中文（回复、注释、提交说明、changeset 说明）。

## 项目是什么

**数据驱动的角色台词引擎**：让经典文学角色（别里科夫、格里高尔、孔乙己、阿Q、哈姆雷特、堂吉诃德）在 QQ 群里"活"过来。核心设计是**引擎与角色卡数据完全解耦**：

- `RolecardEngine`（core.ts）是通用台词引擎，不含任何角色专属逻辑；角色卡是 `assets/<id>/` 下的纯数据资源（`rolecard.json` 清单 + `words.json` 台词库 + `trigger-words.json` 触发词 + 可选插图），**新增角色只需添加数据目录，不改引擎代码**。
- 引擎行为：按优先级匹配关键词触发、概率随机触发、分频道冷却（白名单豁免）、按概率附插图。
- 配置模型：全局 `cooldownWhitelist` + 多群聊 `channels[]`，每群每角色卡独立参数（冷却 / 标签开关 / 随机概率 / 插图），通过 `Schema.intersect` 条件联动。
- 注意：「加卡不改码」指**引擎**；新角色卡要在 `types.ts` / `config.ts` / `index.ts` 各加约 10 行注册开关与专属配置（参看已有六个角色的写法）。

模块依赖方向（不可逆）：`types.ts` ← `config.ts` / `loader.ts` / `core.ts` ← `index.ts`。角色卡只含数据不含逻辑，契约以 `types.ts` 为准。

## 工作流与门禁

```bash
yarn check     # biome check . + tsc --noEmit（提交前必跑）
yarn fix       # biome check --write . + tsc（自动修复后再验类型）
yarn build     # tsdown 构建，产物 lib/（CJS .cjs + d.ts）
yarn changeset # 写 changeset
yarn release   # changeset version && yarn build && npm publish --access public
```

- 工具链：TS7（`@typescript/native` = npm:typescript@7.0.2）、tsdown、biome ^2.5.9。依赖已从工作区根提升，**禁止在本目录运行任何 install / add / remove**。
- 门禁全绿（check 0 error、build 成功）才算完成；逐模块改动，每完成一块跑一次 `yarn check`。

## 代码风格（biome + tsconfig 双重强制）

- 4 空格缩进、行宽 100、LF、**双引号**、尾逗号 all；biome 自动整理导入（organizeImports）。
- 严格模式全家桶：`noUncheckedIndexedAccess` / `exactOptionalPropertyTypes` / `verbatimModuleSyntax` / `noUnusedLocals` / `erasableSyntaxOnly` 均为 error。
- 类型导入一律 `import type`；无默认导出（Koishi 插件入口 `apply` 及 tsdown 配置除外）；`noFloatingPromises` 为 error——异步调用必须 `await` / `void` / 显式 `.catch`。

## Changesets 工作流

每次完成一批**用户可见**改动（feat / fix / 新角色卡）必须随改动一起写 changeset，不要攒到发版前（攒必漏，漏写则 `changeset version` 无 pending 条目直接退出、无法发包）。写法：`yarn changeset` 交互式，或手写 `.changeset/<名字>.md`：

```md
---
"koishi-plugin-rolecard": patch
---

fix: ……（简体中文说明）
```

- bump 类型：`patch` = 修复 / 小改动 / 新增角色卡；`minor` = 向后兼容的功能新增；`major` = 破坏性变更。
- **不需要 changeset**：纯 chore（格式化、依赖升级、文档、tsbuildinfo）。
- 发版：`yarn release`（自动升版本 + 构建 + `npm publish --access public`）；发不出包先检查 `.changeset/` 是否有 pending 条目。
- `package.json` 中的 `"workspaces": ["."]` 是**必需的 changesets 根标记**（见下方已知坑），勿删。

## 已知坑（历史经验，别再踩）

- **NodeNext 下相对导入必须带 `.js` 扩展名**（`from "./types.js"`），不带会被 tsc 拒绝且报出一串误导性的 implicit any。
- **`erasableSyntaxOnly` 禁止构造器参数属性**（`constructor(private readonly x: T)`）：需显式声明字段并在构造体内赋值。
- **源码 ESM、产物 CJS**：`package.json` 为 `type: module`，但 Koishi loader 用 `require()` 加载插件，tsdown 固定输出 `.cjs`。源码里的 `__dirname` 依赖 CJS 产物语义，若改产物为 ESM 必须换 `import.meta.url` + `fileURLToPath`。
- **koishi 的 d.ts 不能被 dts 打包**：其生态类型用 CJS dts 语法（`export =`）或 namespace 成员 re-export，tsdown 配置里已用 `deps.dts.neverBundle` 保持外部引用，勿删。
- `assets/` 角色卡目录下所有图片会被自动扫描为插图（按扩展名映射 MIME），非插图图片勿放。
- **changesets 根解析**：本项目是 koishi-dev yarn workspace 的成员，但 changesets 依赖的 @manypkg 会向上找到 koishi-dev 根、找不到本项目的 `.changeset/`。`package.json` 的 `"workspaces": ["."]` 就是阻止这个向上查找的标记（对 yarn/npm 均无副作用，已实测 publish dry-run 正常），勿删勿改。
- biome 实际版本（^2.5.9，当前 2.5.10）高于 `$schema` 声明的 2.5.9 时会输出一条版本提示（info 级，非 error），无害，勿"修复"。

## git 提交流程

1. 先跑 `yarn check`（必要时先 `yarn fix`），全部通过再提交。
2. `git add -A` 后提交，提交信息用简体中文，格式参考现有历史（`feat:` / `fix:` / `chore:`）。
3. 提交到 `main` 分支；若不在主分支先切回。
4. 提交完成后向用户简要说明改动内容与提交哈希。
