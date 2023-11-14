- https://pnpm.io/
- https://docs.npmjs.com/cli/commands
- https://npmmirror.com/

---

#### 修改 node_module 包方法

##### 1. 直接修改

通过 `pnpm store status` 查看状态（仅当前应用下有效果）

通过 `pnpm install --force` 去生新捕获修改后的原文件。

##### 2. pnpm patch

通过 `pnpm patch` 得到临时目录，修改后，通过 `pnpm patch-commit <path>` 进行打补丁的方式。

##### 3. pnpm link

下载源包去 `pnpm link` 同 npm。

#### 修改国内源

同 npm，修改 `.npmrc` 或者直接 config 命令实现，如修改 cnpm 的源：

```
registry=https://registry.npmmirror.com
```
