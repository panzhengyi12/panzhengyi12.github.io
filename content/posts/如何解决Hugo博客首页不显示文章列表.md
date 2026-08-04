---
title: Hugo 博客首页不显示文章列表？reimu 主题配置详解
date: 2026-08-04
categories: [博客搭建]
tags: [Hugo, reimu, 配置, GitHub Pages]
---
文章内容...



## 问题现象

在本地部署的 Hugo 博客中，使用 **reimu** 主题时，首页只显示个人简介信息，右上角统计显示“文章 0”，但文章在「标签」和「归档」页面却能正常显示。

## 原因分析

reimu 主题默认将首页设计为个人简介页，需要通过正确的配置来显示文章列表。经过排查，问题主要出在以下几个方面：

1. **配置文件优先级**：Hugo 会优先读取 `config/_default/` 目录下的配置，根目录的 `hugo.toml` 中关于 `[params]` 的设置可能被覆盖。
2. **`mainSections` 指定错误**：主题需要明确指定文章所在的文件夹名称，而默认配置中写的是 `["post"]`，但实际文章存放在 `content/posts/` 中（多了个 `s`），导致匹配不到。
3. **分页参数缺失**：主题的首页文章列表依赖 `paginate.post` 参数，若未设置具体数值，则无法生成列表数据。
4. **无效参数干扰**：在根目录 `hugo.toml` 中错误添加了 `listOnHome = true`，但 reimu 主题并不识别该参数，反而可能造成混淆。

---

## 解决步骤

### 第一步：确认文章存放路径

打开博客根目录下的 `content/` 文件夹，查看您的 `.md` 博文存放在哪个子目录中。

- 常见目录名：`posts`、`post`、`blog` 等。
- 我的实际路径是：`content/posts/`，因此后续配置需要指向 `"posts"`。

### 第二步：修改正确的配置文件

**重要**：不要修改根目录的 `hugo.toml`，而应修改 `config/_default/params.yml`。

用文本编辑器（如 VS Code）打开 `config/_default/params.yml`，找到以下两处并修改：

#### ① 修改 `mainSections`

原内容（约第 29 行）：

```yaml
mainSections: ["post"]
```

修改为（根据您的实际文件夹名，多个可用逗号分隔）：

```yaml
mainSections: ["posts", "blog"]
```

> 这样即使以后文章放在 `blog` 文件夹也能兼容。

#### ② 填写分页参数

原内容（约第 210 行）：

```yaml
paginate:
  archive: # Number of posts per page in archive and taxonomy pages
  post: # Number of posts per page in home and other list pages
```

修改为（填写具体数字，如 10）：

```yaml
paginate:
  archive: 10
  post: 10
```

> `post: 10` 表示首页每页显示 10 篇文章，`archive: 10` 同理。

### 第三步：本地重新生成静态文件

在博客根目录打开终端，执行：

```bash
hugo
```

### 第四步：提交并推送到 GitHub

```bash
git add config/_default/params.yml
git commit -m "fix: 修正 mainSections 和 paginate 配置，使首页显示文章列表"
git push origin main
```

### 第五步：等待部署完成并验证

- 前往 GitHub 仓库的 **Actions** 选项卡，等待构建任务成功（绿色对勾）。
- 访问您的博客地址，按 `Ctrl + F5` 强制刷新浏览器缓存。
- 此时首页应正常显示文章列表。

---

## 配置前后对比（关键差异）

| 配置项 | 修改前 | 修改后 |
|--------|--------|--------|
| `mainSections` | `["post"]` | `["posts", "blog"]` |
| `paginate.post` | 空值 | `10` |
| `paginate.archive` | 空值 | `10` |

---

## 常见问题排查

### Q1：修改后首页仍不显示文章？

- 检查您的博文是否有正确的 **Front-matter**，至少包含 `title` 和 `date`，且 `draft` 不为 `true`。
- 确认 `content/` 下的文件夹名与 `mainSections` 中的名称完全一致（注意大小写）。

### Q2：根目录的 `hugo.toml` 还需要保留什么？

根目录的 `hugo.toml` 只需保留基础配置，如 `baseURL`、`theme`、`languageCode` 等。所有与 `params` 相关的配置都放在 `config/_default/params.yml` 中，避免重复定义。

### Q3：为什么不能用 `listOnHome = true`？

reimu 主题并未实现该参数，它使用 `mainSections` 和 `paginate` 来控制首页列表，因此添加此参数无效且可能干扰解析。

---

## 总结

本次问题源于对 Hugo 多层级配置机制的不熟悉，以及 reimu 主题特有参数的正确使用。通过将 `mainSections` 指向正确的文章目录，并补充分页参数，即可让首页恢复文章列表显示。

如果您遵循以上步骤操作，问题应能顺利解决。若仍有疑问，欢迎在评论区留言交流。
```

您可以直接复制上面的内容，保存为 `如何解决Hugo博客首页不显示文章列表.md`，然后放在 `content/posts/` 目录下，重新生成并推送，它就会成为您博客中的一篇文章。同时，这篇文章本身也记录了解决问题的完整思路，很有价值。😊

---

