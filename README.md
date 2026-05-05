
# 蓝奏云直链解析 API（Cloudflare Worker 版）
 
[![Platform](https://img.shields.io/badge/platform-Cloudflare%20Workers-orange)](https://workers.cloudflare.com/)

基于 Cloudflare Worker 的蓝奏云分享链接解析服务，可将任意蓝奏云分享链接转换为真实下载直链，支持有密码/无密码分享，并允许自定义下载文件名。

原 PHP 项目：[LanzouAPI](https://github.com/hanximeng/LanzouAPI)  
Worker 版本由 PHP 代码完整转换，保持接口兼容，无需服务器，一键部署即可使用。

---

## 特性

- 🚀 **完全免费**：基于 Cloudflare Workers 免费计划，每日 10 万次请求额度。
- ⚡ **极速响应**：全球边缘节点运行，低延迟。
- 🔒 **密码支持**：自动识别并处理带密码的分享链接。
- 📦 **直链直达**：绕过蓝奏云页面，直接输出下载地址。
- 🖥 **浏览器友好**：支持 CORS 跨域，可直接在前端调用。
- 📄 **自定义文件名**：可替换直链中的文件名后缀。
- 📋 **美观首页**：访问根路径自动展示详细使用文档。

---

## 快速部署

### 前提条件

- 一个 [Cloudflare 账号](https://dash.cloudflare.com/)
- 已开通 Workers 服务（默认免费套餐即可）

### 部署步骤

1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)，进入 **Workers & Pages**。
2. 点击 **创建应用程序** → **创建 Worker**。
3. 在代码编辑器中，**清空默认代码**，然后粘贴本仓库提供的 [`worker.js`](./worker.js) 全部内容。
4. 点击 **保存并部署**，系统会分配一个 `*.workers.dev` 子域名，即为你的 API 地址。
5. （可选）在 Worker 的 **触发器** 中添加自己的域名路由。

部署完成后，访问你的 Worker 域名（例如 `https://lanzou-api.workers.dev/`）即可看到美观的文档首页。

---

## API 文档

### 基础信息

- **基础地址**：`https://<你的 Worker 域名>/`
- **请求方式**：`GET`（也接受 `POST`，参数置于 Query String 中）
- **响应格式**：`application/json`（当 `type=down` 时为 302 重定向）

### 请求参数

| 参数名 | 必填 | 类型   | 说明 |
|--------|------|--------|------|
| `url`  | 是   | string | 蓝奏云分享链接，支持 `lanzouf.com`、`lanzoux.com` 等域名，传入不完整路径亦可，程序自动提取 `.com/` 之后的部分。 |
| `pwd`  | 否   | string | 分享密码/提取码，如果分享有密码则必须提供。 |
| `type` | 否   | string | 当值为 `down` 时，接口直接返回 302 跳转到最终下载地址；其它值或不传则返回 JSON 信息。 |
| `n`    | 否   | string | 自定义下载文件名，仅当最终下载链接中包含 `fn=` 参数时有效（例如原链接为 `...?fn=原文件.zip`，传入 `n=新文件.zip` 后会替换）。 |

### 响应结构

**成功响应（`type`≠`down`）**

```json
{
  "code": 200,
  "msg": "解析成功",
  "name": "示例文件.zip",
  "filesize": "12.3 M",
  "downUrl": "https://down.lanzoug.com/file/..."
}


**在网页中使用**

```javascript
fetch('https://your-worker.workers.dev/?url=https://www.lanzouf.com/xxxx')
  .then(res => res.json())
  .then(data => {
    if (data.code === 200) {
      console.log('文件名：', data.name);
      console.log('大小：', data.filesize);
      console.log('下载地址：', data.downUrl);
    } else {
      console.error('解析失败：', data.msg);
    }
  });
