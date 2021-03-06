---
slug: 20210623
title: Audit with caution
author: Yao
author_title: senior developer
author_url: https://github.com/caperso
author_image_url: https://avatars.githubusercontent.com/u/34877623?s=400&u=8da3f1b8199cdbd5591ea229149fa663f2011065&v=4
tags: [npm, audit]
---

`npm audit` 提供基于 SEMVER 约定的版本控制.

但是遇到一个缺陷无法通过 `npm audit` 自动 fix 是特别常见的事情

可能情况有如下:

1. 目标 lib 可能不遵循 SEMVER 约定
2. 缺陷警报是最近发生的, 当下并未及时修复
3. 不再当前大版本内继续维护
4. lib 不再维护
<!--truncate-->

通常的处理手段:

1. 有可用版本时:根据建议的 manual fix:
   - `npm install XX@xx.yy.zz`
   - 使用 `npm audit --force`
2. 无版本可用时:
   - fork repo, 在仓库内进行修复该 lib 的缺陷依赖项
   - 变通: 修改`package-lock.json`中依赖的有缺陷的依赖项的版本

对上述手段,个人看法

- 不建议使用 --force, 强制全部升级容易带来更大的混乱;
- fork repo 进行自我维护也是可行的, 但需要对 lib 更深入研究,工作量和未来维护可能就难以估计了;
- 变通方法也是取巧的一种, 比较莽撞.

进行升级

升级这些依赖, 个人遵循如下规则:

1. 查看各个依赖 readme.md, 了解依赖的功能
2. 查看项目引入点, 引入数量, 常用的模块
3. 查看各版本安全性问题 工具: https://snyk.io
4. 评估,进行保守/流行版本选择
5. 查看依赖升级跨度, 其中的 breaking changes
6. 逐条核实可能变更, 最小变更代码
7. 完成后进行可能的检查(test, 启动项目, story, type 检查等);
8. 若带来 break, 再次查找新的合适版本

例如升级 react-markdown, 参考 PR:

https://bitbucket.org/xivart/%7B3b434a2f-3aec-4049-8c2a-e3b49bdaa464%7D/pull-requests/71

`"react-markdown": "^4.0.8" => "^5.0.3",`

1. 首先了解到这是个在 react 中使用 markdown 的 lib;
2. 查询版本 npmjs.com,找到仓库地址: github.com/remarkjs/react-markdown 并找到最近符合 SEMVER 的安全升级: 4.3.1
3. 查询该版本安全性: https://snyk.io/advisor/npm-package/react-markdown, 发现并不可靠: https://snyk.io/test/npm/react-markdown/4.3.1
4. 4.3.1 包含缺陷库 trim,依赖路径react-markdown@4.3.1 › remark-parse@5.0.0 › trim@0.0.1
5. 尝试升级到最新版 6.0.2
6. 查询仓库地址下的 CHANGELOG.md
7. 包含 breaking change(4.0.8=>6.0.2)

   - https://github.com/remarkjs/react-markdown/blob/main/changelog.md#breaking

   - https://github.com/remarkjs/react-markdown/blob/main/changelog.md#remove-buggy-html-in-markdown-parser

8. 我们仓库查找相关 import `import ReactMarkdown from 'react-markdown/with-html';`
9. 仅用了这个方法, 是转换输入 string 为 html.
10. 但一个重大变更(见 7.2),直接替换该功能模块为`rehype-raw`.
11. 变更使得当前代码无法使用, 于是退回小版本 5.0.3;
12. 升级, 尽最大可能检查

    - npm audit 检查: npm audit --audit-level=moderate --production

    - 进行 test, 可能会有 snapshot 更新

    - 启动 storybook, 在 react-markdown 相关页面中, 进行简单 markdown 输入测试.

    - 启动项目, 观察相关页面

13. 都通过后, 基本能通过该 lib 的安全升级了

其他

1. 在升级中, 很能体现 ts,高覆盖的 test 和 lib 引入点的组件创建 story 的重要性

2. 尽可能权衡升级版本的控制:

   - 通常最近的流行版本是安全且受推崇的(通过 npmjs.com 可查到版本下载量)

   - 升级至最接近的安全版本是保守的做法, 不过也可能最早受新的缺陷影响

   - 过大的版本跳跃可能带来更多 breaking change,增加风险
     `

3. 升级依赖总是会带来新的不确定成分, 使用 ts, 好的 test、story 是给之后的维护铺路, 如果是空降的老项目,做好能做到的,然后 wish you good luck`

**Update**

最近看了 Dan 的文章, 关于 audit 的吐槽

提及了很多来自 npm audit 的修复是无意义的(对于开发而言),

却造成了诸多困扰

最后提供了若干解决手段, 虽然并不能解决主要矛盾

其中有一个关于现在 cra / vite / nextjs 在做的内联依赖, 整体脱离 node_modules

这个很新颖, 可以了解下

原文暂无翻译

https://overreacted.io/npm-audit-broken-by-design/
