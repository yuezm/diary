# Git Hooks、husky、lint-staged

## Git Hooks

Git Hooks 就是 git 在执行过程中的钩子，可以触发自定义脚本。每个项目（含有 repository）下都存在一个.git 文件，每个.git 文件都包含一个 hooks 文件 _$project/.git/hooks_，你可以在此文件夹根据钩子的名称，创建相应的钩子文件，当 git 执行某些操作时，相应的钩子文件就会被执行

## Git Hooks 分类

### 客户端钩子

顾名思义，在客户端进行某些操作时，执行的钩子，常用的有如下

1. **pre-commit**：由 git commit 调用，在 commit 之前执行，检查即将提交的快照，以非 0 退出时，中断此次提交。例如可以用于代码的格式检查、运行测试用例...
2. pre-merge-commit：由 git merge 调用，在 merge 之前执行，以非 0 状态退出时，会中断此次合并
3. prepare-commit-msg：由 git commit 调用，在准备默认的日志信息且**启动编辑前**执行，可以结合模板来使用，动态插入信息
4. commit-msg：由 git commit 或 git merge 调用，以非 0 状态码退出时，会中断此次操作
5. **pre-push**：被 git push 调用，在 git push 前执行，防止进行推送，以非 0 状态码退出时，会中断此次推送 ...

### 服务端钩子

顾名思义，在服务端进行某些操作时，执行的钩子，常用的如下

1. **pre-receive**：响应 git push 后，更新起存储时被调用，以非 0 状态码退出时，表示此次推送不成功。例如可以用来检查此次推送的 message 是否合规
2. update：响应 git push 后，更新起存储时被调用，以非 0 状态码退出时，表示此次推送不成功。它和 pre-receive 的区别是，当一次推送更新多个分支时，pre-receive 只运行一次，而 update 会在每个问题都运行一次
3. **post-receive**：完成 git push 后，所有引用均已更新。例如可以用来自动构建、部署
4. post-update：完成 git push 后，所有引用均已更新。它和 post-receive 区别是，post-update 只能接收到更新，**它无法知道旧值和新值**，但 post-receive 不仅可以知道有哪些文件更新了，还知道旧值和新值

详细的可以参考文档[githooks docs](https://git-scm.com/docs/githooks)

## 代码示例

1. 创建钩子文件

```bash
### 在 $peoject/.git/hooks 文件夹下，创建一个 pre-commit 文件，写入如下代码

# !/bin/sh
echo "<<<<<< I am in pre-commit >>>>>>"
exit 1 # 如果想成功的话，则 exit 0
```

1. 修改为可执行文件

```bash
#### 文件创建好后，执行如下命令
chmod a+x .git/hooks/pre-commit
```

1. 测试

```bash
git add .
git commit -m "测试代码" # 此时可以看到打印出的信息，且无法成功
```

## Husky（v6）

husky 是以 javascript 的代码来管理 Git Hooks，主要是直接修改 .git/hooks 文件不方便，且 shell 也不是前端所熟悉的语言

husky 总共分为 3 个操作

1. install：初始化

  - 在项目根目录创建一个 _.husky_ 文件夹，并复制 _husky.sh_ 进入 _.husky__ 内 。
  - 设置 git 配置，将 _.husky_ 添加为 hooks 路径，`git config core.hooksPath .husky`

2. set、add：添加 Git Hooks。set 和 add 功能都是添加 Git Hooks，但是对于已经添加过的 Git Hooks 则有所不同

  - set 对于已经存在的Git Hooks，直接覆盖
  - add 对弈已经存在的 Git Hooks，会在在后面添加

```bash
npx husky set .husky/pre-commit "npm test" # 这是第一个 Git Hooks

npx husky set .husky/pre-commit "npm test1" # 会直接将 pre-commit 文件内容覆盖掉

npx husky add .husky/pre-commit "npm test2" # 会直接在 pre-commit 末尾处再增加一行代码 npm test2
```

1. uninstall：删除钩子，这里的删除钩子，并非是删除 .husky* 的钩子文件，而是删除 git 的配置，将 .husky 从 hooks 的路径中移除，`git config --unset core.hooksPath`

可以使用 `git config --local --list | grep "core.hookspath"` 来查看你配置的 Git Hooks 路径情况

### 配置实例

ps husky(V6) 有如下两种方式生成 Git Hooks，原来在 package.json 配置 { "hooks": { ... }} 已经不得行了，具体见[Why husky has dropped conventional JS config](https://blog.typicode.com/husky-git-hooks-javascript-config/)

#### 手动添加

```bash
npm install husky --save-dev
npx husky install # 初始化

npx husky add .husky/pre-commit "npm test" # 表示在 pre-commit 执行 npm test
```

#### 脚本添加

```json
{
  "scripts": {
    "prepare": "husky install"
  }
}
```

```bash
npm install husky --save-dev # 由于 npm install 的S生命周期，可以少执行初始化 preinstall --> install --> postinstall --> prepublish --> preprepare --> prepare --> postprepare
npx husky add .husky/pre-commit "npm test"
```

## lint-staged

lint-staged 只是读取**暂存区**的文件，并运行配置好的脚本，避免了对未提交到暂存区的代码造成影响。可以配个 husky 使用，由 husky 触发 lint-staged，再有 lint-staged 执行脚本

lint-staged 由 git 命令获取暂存区的文件 `git diff --staged --diff-filter=ACMR --name-only -z`，整体步骤大致如下

1. 通过命令获取暂存区文件名
2. 将文件拆分后，进行序列化，获取文件的完整路径
3. 获得完整路径后，根据配置的相应执行规则，创建任务，并执行
