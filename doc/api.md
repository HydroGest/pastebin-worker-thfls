# API 参考

## GET `/`

返回首页。

## **GET** `/<name>[.<ext>]` 或 `/<name>/<filename>[.<ext>]`

获取名为 `<name>` 的粘贴内容。默认情况下，它将返回粘贴的原始内容。

`Content-Type` 头信息会被设置为 `text/plain;charset=UTF-8`。如果指定了 `<ext>`，服务工作线程会根据 `<ext>` 推断 MIME 类型并更改 `Content-Type`。此方法接受以下查询字符串参数：

`Content-Disposition` 头信息默认设置为 `inline`。但可以通过 `?a` 查询字符串进行覆盖。如果粘贴内容上传时带有文件名，或者在给定的请求 URL 中设置了 `<filename>`，`Content-Disposition` 会追加 `filename*` 来指示文件名（如果 `<ext>` 存在则包含 `<ext>`）。

- `?a=`: 可选。如果存在该参数，将 `Content-Disposition` 设置为 `attachment`。
- `?lang=<lang>`: 可选。返回一个由 prism.js 提供语法高亮的网页。
- `?mime=<mime>`: 可选。指定 MIME 类型，忽略 `<ext>` 的影响。如果指定了 `lang`，则此参数无效（这种情况下 MIME 类型始终为 `text/html`）。

示例: `GET /abcd?lang=js`，`GET /abcd?mime=application/json`。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `404`: 未找到指定名称的粘贴内容。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```shell
$ curl https://shz.al/i-p-
https://web.archive.org/web/20210328091143/https://mp.weixin.qq.com/s/5phCQP7i-JpSvzPEMGk56Q

$ curl https://shz.al/~panty.jpg | feh -

$ firefox 'https://shz.al/kf7z?lang=nix'

$ curl 'https://shz.al/~panty.jpg?mime=image/png' -w '%{content_type}' -o /dev/null -sS
image/png;charset=UTF-8
```

## GET `/<name>:<passwd>`

返回用于编辑名为 `<name>` 且密码为 `<passwd>` 的粘贴内容的网页。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `404`: 未找到指定名称的粘贴内容。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

## GET `/u/<name>`

重定向到名为 `<name>` 的粘贴内容中记录的 URL。

如果发生错误，服务工作线程将返回不同于 `302` 的状态码：
- `404`: 未找到指定名称的粘贴内容。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```shell
$ firefox https://shz.al/u/i-p-

$ curl -L https://shz.al/u/i-p-
```

## GET `/a/<name>`

返回从名为 `<name>` 的粘贴内容中存储的 Markdown 文件转换而来的 HTML。Markdown 转换遵循 GitHub Flavored Markdown (GFM) 规范，由 [remark-gfm](https://github.com/remarkjs/remark-gfm) 提供支持。

语法高亮由 [prims.js](https://prismjs.com/) 提供支持。LaTeX 数学公式由 [MathJax](https://www.mathjax.org) 提供支持。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `404`: 未找到指定名称的粘贴内容。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```md
# 标题 1
这是 `test.md` 的内容

<script>
alert("脚本应被移除")
</script>

## 标题 2

| abc | defghi |
:-: | -----------:
bar | baz

**粗体**，`等宽字体`，*斜体*，~~删除线~~，[链接](https://github.com)

- A
 - A1
 - A2
- B

![短裤](https://shz.al/~panty.jpg)

1. 第一项
2. 第二项

> 引用

$$
\int_{-\infty}^{\infty} e^{-x^2} = \sqrt{\pi}
$$

```

```shell
$ curl -Fc=@test.md -Fn=test-md https://shz.al

$ firefox https://shz.al/a/~test-md
```

## **POST** `/`

上传你的粘贴内容。它接受表单数据中的参数：
- `c`: 必需。你的粘贴内容，可以是文本或二进制数据。其大小不应超过 10 MB。在获取粘贴内容时，其 `Content-Disposition` 中的 `filename` 会显示出来。
- `e`: 可选。粘贴内容的**过期**时间。在此时间段过后，粘贴内容将被永久删除。它应该是一个整数或浮点数，并可选择添加单位（默认单位为秒）。支持的单位有：`s`（秒）、`m`（分钟）、`h`（小时）、`d`（天）、`M`（月）。例如，`360.24` 表示 360.25 秒；`25d` 表示 25 天。由于 Cloudflare KV 存储的限制，该值不应小于 60 秒。
- `s`: 可选。用于修改和删除粘贴内容的**密码**。如果未指定，服务工作线程将生成一个随机字符串作为密码。
- `n`: 可选。你的粘贴内容的自定义**名称**。如果未指定，服务工作线程将生成一个随机字符串（默认 4 个字符）作为名称。在获取自定义名称的粘贴内容时，你需要在名称前加上 `~`。名称至少为 3 个字符，由字母、数字以及 `+_-[]*$=@,;/` 中的字符组成。
- `p`: 可选。**私密模式**标志。如果指定为任何值，粘贴内容的名称将长达 24 个字符。如果使用了 `n`，则此参数无效。

`POST` 方法默认返回一个 JSON 字符串，如果没有发生错误，示例如下：
```json
{
    "url": "https://shz.al/abcd",
    "admin": "https://shz.al/abcd:w2eHqyZGc@CQzWLN=BiJiQxZ",
    "expire": 100,
    "isPrivate": false
}
```

字段说明：
- `url`: 字符串。用于获取粘贴内容的 URL。使用自定义名称时，格式类似于 `https//shz.al/~myname`。
- `suggestUrl`: 字符串或 null。可能携带文件名或 URL 重定向的 URL。
- `admin`: 字符串。用于更新和删除粘贴内容的 URL，即 `url` 后面加上 `~` 和密码。
- `expire`: 字符串或 null。过期秒数。
- `isPrivate`: 布尔值。粘贴内容是否处于私密模式。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `400`: 你的请求格式不正确。
- `409`: 名称已被使用。
- `413`: 内容太大。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```shell
$ curl -Fc="kawaii" -Fe=300 -Fn=hitagi https://shz.al  # 上传一些文本
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false,
  "expire": 300
}

$ curl -Fc=@panty.jpg -Fn=panty -Fs=12345678 https://shz.al   # 上传一个文件
{
  "url": "https://shz.al/~panty",
  "admin": "https://shz.al/~panty:12345678",
  "isPrivate": false
}

# 因为 `curl` 会将某些字符作为字段分隔符，如果字段包含分号或逗号，则这些字段应该用双引号引起来
$ curl -Fc=@panty.jpg -Fn='"hi/hello;g,ood"' -Fs=12345678 https://shz.al
{
  "url": "https://shz.al/~hi/hello;g,ood",
  "admin": "https://shz.al/~hi/hello;g,ood:QJhMKh5WR6z36QRAAn5Q5GZh",
  "isPrivate": false
}
```

## **PUT** `/<name>:<passwd>`

更新名为 `<name>` 且密码为 `<passwd>` 的粘贴内容。它接受表单数据中的参数：
- `c`: 必需。与 `POST` 方法相同。
- `e`: 可选。与 `POST` 方法相同。注意，此时删除时间将重新计算。
- `s`: 可选。与 `POST` 方法相同。

`PUT` 方法的返回结果与 `POST` 方法相同。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `400`: 你的请求格式不正确。
- `403`: 你的密码不正确。
- `404`: 未找到指定名称的粘贴内容。
- `413`: 内容太大。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```shell
$ curl -X PUT -Fc="kawaii~" -Fe=500 https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false,
  "expire": 500
}

$ curl -X PUT -Fc="kawaii~" https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false
}
```

## DELETE `/<name>:<passwd>`

删除名为 `<name>` 且密码为 `<passwd>` 的粘贴内容。全局同步删除操作可能需要几秒钟。

如果发生错误，服务工作线程将返回不同于 `200` 的状态码：
- `403`: 你的密码不正确。
- `404`: 未找到指定名称的粘贴内容。
- `500`: 发生意外异常。你可以向作者报告此问题以进行修复。

使用示例：
```shell
$ curl -X DELETE https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
粘贴内容将在几秒钟内被删除

$ curl https://shz.al/~hitagi
未找到
```
