# Pastebin-worker

这是一个可以部署在Cloudflare Workers上的代码分享平台（pastebin）。你可以在 [paste.thfls.club](https://paste.thfls.club) 上体验。 

**理念**：轻松部署，友好的命令行界面使用体验，功能丰富。 

**特点**：
1. 分享代码时，网址短至4个字符。
2. 自定义代码分享链接。
4. 随意**更新**和**删除**你的分享内容。
5. 设定一段时间后**过期**分享内容。
6. 由PrismJS提供支持的**语法高亮**功能。
7. 将**Markdown**文件显示为HTML格式。
8. 可用作网址缩短工具。
9. 自定义返回的媒体类型（mimetype）。

## 使用方法
1. 你可以直接在网站上（如 [paste.thfls.club](https://paste.thfls.club) ）发布、更新、删除你的分享内容。 
2. 它还提供了便捷的HTTP API。详情请查看 [API参考](doc/api.md) 。你可以通过命令行（使用`curl`或类似工具）轻松调用API。 
3. [pb](/scripts) 是一个bash脚本，方便在命令行中使用。

## 限制
1. 如果部署在Cloudflare Worker的免费套餐上，该服务每天最多允许100,000次读取、1000次写入和1000次删除操作。 
2. 由于Cloudflare KV存储的大小限制，每个分享内容的大小限制在25MB以内。 

## 部署
如果你在Cloudflare上托管你的域名，你可以自由地在自己的域名上部署这个代码分享平台。 
1. 安装`node`和`yarn`。
2. 在Cloudflare Workers仪表板上创建一个KV命名空间，记住它的ID。
3. 克隆这个仓库并进入目录。
4. 修改`wrangler.toml`中的条目。其中的注释会告诉你如何操作。
5. 登录Cloudflare并按以下步骤进行部署：
```console
$ yarn install
$ yarn wrangler login
$ yarn deploy
```
6. 尽情使用吧！

## 认证
如果你想要一个私人部署（只有你可以上传分享内容，但任何人都可以读取），在你的`wrangler.toml`中添加以下条目：
```toml
[vars.BASIC_AUTH]
user1 = "passwd1"
user2 = "passwd2"
```
现在，每次对POST请求的访问，以及对静态页面的每次访问，都需要使用上述列出的用户名 - 密码对进行HTTP基本认证。例如：
```console
$ curl example-pb.com
需要HTTP基本认证

$ curl -Fc=@/path/to/file example-pb.com
需要HTTP基本认证

$ curl -u admin1:wrong-passwd -Fc=@/path/to/file example-pb.com
错误401：基本认证密码错误

$ curl -u admin1:this-is-passwd-1 -Fc=@/path/to/file example-pb.com
{
  "url": "https://example-pb.com/YCDX",
  "suggestUrl": null,
  "admin": "https://example-pb.com/YCDX:Sij23HwbMjeZwKznY3K5trG8",
  "isPrivate": false
}
```

## 管理
删除一个分享内容：
```console
$ yarn delete-paste <分享内容名称>
```
列出分享内容：
```console
$ yarn -s wrangler kv:key list --binding PB > kv_list.json
```

## 开发
运行本地模拟器：
```console
$ yarn dev
```
运行测试：
```console
$ yarn test
```
运行测试并生成覆盖率报告：
```console
$ yarn coverage
```

---
特别感谢原作者，本仓库由天河外国语学校计算机社团Fork并汉化。
