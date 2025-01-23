作为一名 Pixiv 爱好者，能够及时获取关注画师的最新作品是一件令人愉悦的事情。然而，每天手动检查更新既费时又费力，有没有更高效的方法呢？本文将介绍如何通过 Telegram 自动订阅 Pixiv 新图，让每一张作品都能被及时推送到你的设备上。

## 方法概述

我们将利用 RSSHub 提供的 Pixiv 订阅接口，通过 Node.js 开发一个工具，将订阅内容推送到 Telegram 频道中。以下是实现的主要步骤：

1. **RSS 数据采集**：通过 `rss-parser` 获取 Pixiv 的订阅数据。
2. **数据处理**：提取图片链接并生成预览图地址。
3. **消息发送**：通过 Telegram Bot API 将图片推送到频道。

---

## 第一步：RSS 数据采集

为了简化 RSS 数据的处理，我们使用了 `rss-parser` 组件。安装方法如下：

bash
npm i rss-parser --save


使用 `rss-parser` 获取 RSS 数据非常简单，以下是一个示例代码：

javascript
const Parser = require('rss-parser');
const parser = new Parser();

(async () => {
  const feed = await parser.parseURL('https://rsshub.app/pixiv/user/123456');
  console.log(feed.items);
})();


通过解析 RSSHub 生成的订阅数据，我们可以获取到一个对象数组，其中包含了图片的链接、标题等信息。

---

## 第二步：数据处理

Pixiv 的图片链接通常以 `https://pixiv.cat/*ArtworkID*.jpg` 的形式呈现。我们可以通过正则表达式提取这些链接：

javascript
const picIdReg = /https:\/\/pixiv\.cat\/(\d+)-?(\d+)?\.(jpg|png|gif)/gi;
const artworks = [...item.content.matchAll(picIdReg)];


对于单张图片和多张图片的情况，提取的结果如下：

- 单图：`https://pixiv.cat/123456.jpg`
- 多图：`https://pixiv.cat/123456-1.jpg`, `https://pixiv.cat/123456-2.jpg`

为了生成预览图地址，我们可以根据 Pixiv 的图片命名规则，结合 RSS 数据中的 `isoDate` 字段，拼接出完整的预览图链接。

---

## 第三步：消息发送

我们使用 Telegram 的 Bot API 将图片推送到频道。以下是一个示例代码：

javascript
const got = require('got');

const apiBaseUrl = `https://api.telegram.org/bot${confData.bot.token}`;

got.post(`${apiBaseUrl}/sendPhoto`, {
  json: {
    chat_id: confData.bot.chat,
    photo: picItem.preview,
    caption: picItem.text,
    reply_markup: {
      inline_keyboard: [
        [
          { text: '🌏 查看原图', url: picItem.url },
          { text: '⤵ 下载图片', url: picItem.pic }
        ]
      ]
    }
  }
});


通过 `sendPhoto` 接口，我们可以发送图片，并在消息下方添加内联按钮，方便用户查看原图或下载图片。

---

## 优化与部署

为了避免重复推送，我们可以维护一个时间戳，每次只发送最新的内容。同时，使用 `js-yaml` 组件读取配置文件，方便管理 Bot 的参数。

最终代码可以通过 `pm2` 等工具进行部署：

bash
pm2 start bot.js --name phandream


运行后，Telegram 频道中将会自动收到 Pixiv 的最新图片。

---

## 结语

通过以上方法，你可以轻松实现 Pixiv 新图的自动订阅和推送。如果你对代码感兴趣，可以尝试进一步优化或扩展功能。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)