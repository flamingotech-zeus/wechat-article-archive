---
name: wechat-article-archive
description: 抓取微信公众号推文的完整图文内容（正文全文 + 防盗链图片），并归档/提炼到本地知识库。当用户给出一条 mp.weixin.qq.com 链接并要求"阅读这篇推文""学习一下""告诉我学到什么""录入知识库"时使用。覆盖 WebFetch 取正文 + agent-browser 提取图片真实地址 + curl 带 referer 下载 + 逐张读图 的完整流程，以及本次踩过的坑。
agent_created: true
---

# 微信公众号推文图文抓取与归档

用户常给出 `https://mp.weixin.qq.com/s/xxxx` 链接，要求「阅读推文 + 图片 + 文字」，然后提炼并录入知识库。微信图片是防盗链，纯 WebFetch 拿不到图，需浏览器 + referer 下载。

## 一、标准流程

### 1. 先拿正文全文（最省事）
用 `WebFetch` 直接请求链接，prompt 要求「逐字完整提取标题/发布账号/发布时间/正文全文/小标题/图片URL/数据案例人名公司名」。
- WebFetch 通常能拿到完整文字与结构（微信文章正文是静态 HTML）。
- 但 WebFetch **拿不到图片的真实 URL**（它只解析文本），必须走浏览器。

### 2. 用 agent-browser 提取图片真实地址
```bash
AB="/c/Users/<user>/.workbuddy/binaries/node/versions/<ver>/agent-browser"   # 用 `which agent-browser` 确认路径
"$AB" open "<url>"
"$AB" wait --load load
"$AB" eval "JSON.stringify([...document.querySelectorAll('img')].map(i=>i.getAttribute('data-src')||i.src||'').filter(s=>s&&s.indexOf('mmbiz')>-1))"
```
- 微信图片懒加载，真实地址在 `data-src`，`src` 只是占位 → **必须优先取 data-src**。
- 返回一串 `https://mmbiz.qpic.cn/...` 地址（正文图带 `#imgIndex=N`，末尾二维码图无）。
- 结束后 `"$AB" close`（整个任务结束才关，中途不要关）。

### 3. 带 referer 下载图片
```bash
curl -s -e "https://mp.weixin.qq.com/" -o "img_$i.png" "<mmbiz_url>"
```
- **必须带 `-e referer`**，否则微信返回 403/空文件。
- 下载后 `stat -c%s` 检查字节数非 0。

### 4. 逐张读图
用 `Read` 工具读每张 png（Read 支持图片视觉），一次 6 张左右分批读，避免上下文过载。

## 二、坑与注意（血泪）

- **首次 `agent-browser open` 会卡住无输出**：因为 Chromium 未安装。先跑 `"$AB" install`（约 196MB，联网下载），装完再 open。别以为是命令错了反复重试。
- `agent-browser open` 可能被前台超时自动转入后台；`agent-browser install` 建议直接 `run_in_background=true` 跑，用 TaskOutput 等完成。
- 微信图片 URL 携带签名，**过期后失效**，需及时下载；URL 里 `#imgIndex=N` 是前端 fragment，curl 会自动忽略，不影响。
- 微信文章页无登录态一般也能渲染图文；若遇验证页，用 `agent-browser snapshot` 确认。

## 三、归档到本地知识库（本项目规范）

知识库 = `D:\GEO工作台\03-知识库本地镜像\`（本地镜像）+ ima 云端。
- **入库铁律：禁止图片，纯文字**。配图只留本地 `_配图\<日期>-<主题>\`，附 `_配图说明.md`（列出每张图内容），不进 ima。
- 命名：`火烈鸟GEO-两位序号-标题.md`；制度 00-09 / 事件 10-49 / 精选 50-79 / **方法论 80-99**（行业干货文章归此类，序号递增）。
- 新增后**必须同批更新 `火烈鸟GEO-00-总目录.md`**（补清单 + 追加维护记录），保持索引一致。
- 文档转 Word 便于查看：用 `D:\GEO工作台\05-模板与配置\md2docx.py` 的 `convert(md_path, docx_path)`，Python 用 `C:\Users\book\.workbuddy\binaries\python\envs\default\Scripts\python.exe`。

## 四、提炼口径

- 长文（1万字+）拆多篇时，按"知识块"拆（如生态/方法/归因/统计与ROI/实操问答），每篇开头标来源（标题+作者+日期），正文用表格压缩「事件→关键词→要点」。
- 纯知识沉淀 + 一句"对本项目的应用要点"，便于复用。
- 严格遵守用户红线：公司名写全称"温州火烈鸟网络科技有限公司"；不杜撰数据，只用原文数字。
