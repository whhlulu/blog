> 作者：lulu
> [原文](https://github.com/whhlulu/blog)

# 用工具思路来规范化 git commit message

### 为什么要规范 Git Commit Message
- 发生问题快速识别问题代码
- commit和代码建立联系，和相关prd、bug予以关联。

### 如何写出规范化的 Git Commit Message
选用当前业界内应用比较广泛的 [Angular Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)

```js
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

1. `type`：commit的类型：`feat` `fix` `refactor` `docs` `style` `test` `chore`

2. `scope`：commit 影响的范围, 比如: route, component, utils, build...

3. `subject`：commit 的概述

    - 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
    - 第一个字母小写(可以使用中文)
    - 结尾没有符号

4. `body`：commit 具体修改内容, 可以分为多行

5. `footer`：一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接

### 项目使用参考步骤
#### 1.Commitizen: 替代你的 git commit
[commitizen/cz-cli](https://github.com/commitizen/cz-cli), 我们需要借助它提供的 git cz 命令替代我们的 git commit 命令, 帮助我们生成符合规范的 commit message.

除此之外, 我们还需要为 commitizen 指定一个 Adapter 比如: [cz-conventional-changelog](https://github.com/commitizen/cz-conventional-changelog) (一个符合 Angular团队规范的 preset). 使得 commitizen 按照我们指定的规范帮助我们生成 commit message.

```js
yarn add --dev commitizen cz-conventional-changelog
```

package.json中配置:

```js
"script": {
    ...,
    "commit": "git-cz",
},
 "config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  }
```

如果全局安装过 commitizen, 那么在对应的项目中执行 git cz or npm run commit 都可以

#### 2.Commitlint: 校验你的 message
[commitlint](https://github.com/marionebl/commitlint): 可以帮助我们 lint commit messages,校验的配置推荐 [@commitlint/config-conventional]() (符合 Angular团队规范).
话不多说直接按照：
```js
yarn add --dev @commitlint/config-conventional @commitlint/cli
```

同时需要在项目目录下创建配置文件 .commitlintrc.js, 写入:
```js
module.exports = {
  extends: [
    '@commitlint/config-conventional'
  ],
  rules: {
  }
};
```

#### 3.结合 Husky
```js
yarn add --dev husky
```

package.json 中添加:
```js
"husky": {
    "hooks": {
      ...,
      "commit-msg": "commitlint -e $GIT_PARAMS"
    }
  },
```



### 参考资料

[优雅的提交你的 Git Commit Message](https://juejin.im/post/5afc5242f265da0b7f44bee4)

[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
