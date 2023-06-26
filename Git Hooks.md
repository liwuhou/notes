[Git Hooks docs](https://git-scm.com/docs/githooks)
每次使用 `git commit` 等命令时，git 都提供了相应的 hook 让开发者在对应节点去调用副作用。可以用来统一管理团队代码风格，约束统一的提交格式，进而增加代码的可读性，减少代码审查的工作量。

### 使用的工具

- [husky](https://www.npmjs.com/package/husky)：Git Hooks 工具
  - 对 git 执行一些命令，通过对应的 hooks 触发，可以执行自定义的脚本
- [lint-staged](https://www.npmjs.com/package/lint-staged)：检测文件插件
  - 只检测 git add 范围，也就是暂存区中的文件，可以对其进行操作
- [eslint](https://www.npmjs.com/package/eslint)：插件化 Javascript 代码检测工具
  - JS 编码规范，检测并提示错误或警告信息，具有一定自动修复
- [prettier](https://www.npmjs.com/package/prettier)：代码格式化工具
  - 代码风格管理，更好的代码风格效果
- [editorconfig](https://editorconfig.org)：文件代码规范
  - 保持多人开发一致编码样式
- [commitlint](https://www.npmjs.com/package/@commitlint/cli)：代码提交检测
  - 可以用来检测 git commit 内容是否符合定义的规范
- [commitizen](https://www.npmjs.com/package/commitizen)：代码提交内容是否标准化
  - 提示定义输入标准的 git commit 内容

