## 概述

### 约定式提交规范是一种基于提交消息的轻量级约定。 它提供了一组用于创建清晰的提交历史的简单规则； 这使得编写基于规范的自动化工具变得更容易。 这个约定与 SemVer 相吻合， 在提交信息中描述新特性、bug 修复和破坏性变更。

提交说明的结构如下所示:

```
<类型>[可选的作用域]: <描述>

[可选的正文]

[可选的脚注]
```
> 摘自 [coventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0/)

其中除了<类型>和<描述>是必填的以外，其他都可选.

---

### 作用:

1. 规范多人协作的提交信息，使之看起来就像一个人写的！提供一个语义化、结构化的提交历史，使得commits的类型一目了然. 尤其是在人数较多的团队，或是由社区维护的一些仓库，这种基于约定，受制于lint和pre-commit等工具的规范就更能体现其价值.
  - [husky](https://github.com/typicode/husky)(`pre` or `post` commit时的检查钩子)
    - [commitlint](https://github.com/conventional-changelog/commitlint#cli)， 顾名思义，是一个lint工具，配合🐶哈士奇，在钩子中触发lint检查.
      - [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional),commitlint需要一定的配置才可使用, 而`config-conventional`正是众多已提供的配置中使用最广泛和适用的一个，他基于`config-angular`做了一些扩展和修改.
2. 根据规范的commit messages，从而提供一个易追溯、易读的版本历史:
  - [standard-version](https://github.com/conventional-changelog/standard-version), 这是一个遵从[semanticVersion](https://semver.org/)版本策略的版本管理和发布工具，并且内部是使用git commits的内容作为元数据，所以规范的commits内容是必要的，于是他依赖于:
3. 基于2，生成规范的CHANGELOG
  - [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog), 简单的说，他提供了standard-version如何来使用git commits元数据的规则，当然，其中也有很多配置，github上都有，这里就不一一列举了.
4. 借助预提交工具, 傻瓜式的生成规范的git commits
  - [@commitlint/prompt-cli](https://github.com/conventional-changelog/commitlint/tree/master/@commitlint/prompt-cli)，commitlint自带的交互式commits生成工具，但是还有更好的选择：
  - [commitizen](https://github.com/commitizen/cz-cli)，交互式提交工具, 让开发者在已有的types中选择，可配置，并且已经有很多成熟的adapters,
    - [cz-conventional-changelog](https://github.com/commitizen/cz-conventional-changelog), 众多adapters中最通用的一个，跟`config-conventional`配套，类似`@commitlint/config-conventional`之于`commitlint`的关系，已经提供了很多可用的配置项，同时也可以根据自己的需要去配置:
    ```
    {
    // ...  default values
        "config": {
            "commitizen": {
                "path": "./node_modules/cz-conventional-changelog",
                "maxHeaderWidth": 100,
                "maxLineWidth": 100,
                "defaultType": "",
                "defaultScope": "",
                "defaultSubject": "",
                "defaultBody": "",
                "defaultIssues": "",
                "types": {
                  ...
                  "feat": {
                    "description": "A new feature",
                    "title": "Features"
                  },
                  ...
                }
            }
        }
    // ...
    }
    ```
