# Changesets

本项目使用 [Changesets](https://github.com/changesets/changesets) 管理版本与发布。

每次完成一批用户可见改动（新功能 / 修复 / 新角色卡）时，**随改动一起**在 `.changeset/` 下写一个 changeset：运行 `yarn changeset` 交互式生成，或直接手写 `.changeset/<名字>.md`（模板见下）。纯 chore（格式化、依赖升级、文档）不需要 changeset。

发版时运行 `yarn release`（= `changeset version && yarn build && npm publish --access public`）：消费所有 pending changeset，更新 `package.json` 版本号后构建并发布。

## changeset 模板

```md
---
"koishi-plugin-rolecard": patch
---

fix: 修复某场景下角色卡不触发的问题（简体中文说明）
```

bump 类型：`patch` = 修复 / 小改动 / 新增角色卡；`minor` = 新配置项等向后兼容的功能新增；`major` = 破坏性变更。
