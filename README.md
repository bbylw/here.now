# here.now - 即时静态网站托管服务

> 免费 · 即时 · 无需账户

here.now 是一个免费的即时静态网站托管服务，让您在几秒钟内部署网站。支持 HTML、CSS、JavaScript、图片、PDF、视频等各种静态文件格式。

📖 [官方文档](https://here.now/docs) | 🚀 [快速开始](#快速开始)

---

## ✨ 核心功能

| 功能 | 描述 |
|------|------|
| ⚡ **即时部署** | 只需几秒钟即可将网站上线，无需等待，无需复杂的配置流程 |
| 🔒 **匿名发布** | 无需注册账户即可发布网站，适合临时分享和快速测试 |
| 🌐 **SSL 自动配置** | 所有网站自动获得 HTTPS 支持，确保内容安全传输 |
| 📁 **多文件支持** | 支持 HTML、CSS、JS、图片、PDF、视频等静态文件 |
| 🔗 **自定义域名** | 支持绑定您自己的域名，打造专业品牌形象 |
| 💻 **命令行工具** | 通过 CLI 工具自动化部署流程，集成到开发工作流 |

---

## 🚀 快速开始

### 方式一：使用 CLI 工具（推荐）

```bash
# 安装 CLI
npx skills add heredotnow/skill --skill here-now -g

# 在项目目录中部署
here-now deploy

# 指定 API 密钥部署
here-now deploy --api-key YOUR_API_KEY
```

### 方式二：使用 API 直接部署

只需三个步骤即可发布网站：

#### 1. 创建站点

```bash
curl -sS https://here.now/api/v1/publish \
  -H "X-HereNow-Client: cursor/direct-api" \
  -H "content-type: application/json" \
  -d '{
    "files": [
      {
        "path": "index.html",
        "size": 1234,
        "contentType": "text/html; charset=utf-8"
      }
    ]
  }'
```

**响应示例：**
```json
{
  "slug": "abc123xyz",
  "siteUrl": "https://abc123xyz.here.now/",
  "upload": {
    "versionId": "01J...",
    "uploads": [
      {
        "path": "index.html",
        "method": "PUT",
        "url": "https://<presigned-url>",
        "headers": { "Content-Type": "text/html; charset=utf-8" }
      }
    ],
    "finalizeUrl": "https://here.now/api/v1/publish/abc123xyz/finalize"
  },
  "claimToken": "claim_token_here",
  "claimUrl": "https://here.now/claim?slug=abc123xyz&token=..."
}
```

> ⚠️ **重要：** 匿名站点务必保存 `claimToken` 和 `claimUrl`，这些信息只返回一次！

#### 2. 上传文件

```bash
curl -X PUT "<presigned-url>" \
  -H "Content-Type: text/html; charset=utf-8" \
  --data-binary @index.html
```

#### 3. 最终化站点

```bash
curl -sS -X POST "https://here.now/api/v1/publish/abc123xyz/finalize" \
  -H "content-type: application/json" \
  -d '{ "versionId": "01J..." }'
```

---

## 📋 完整教程

### 步骤 1：准备工作

#### 选择发布模式

| 模式 | 说明 |
|------|------|
| **匿名模式** | 无需账户，24小时后自动过期，适合临时测试 |
| **认证模式** | 需要 API 密钥，永久存储，适合生产环境 |

#### 获取 API 密钥

**方式一：代理辅助获取**

```bash
# 1. 请求验证码
curl -sS https://here.now/api/auth/agent/request-code \
  -H "content-type: application/json" \
  -d '{"email": "user@example.com"}'

# 2. 验证代码获取 API 密钥
curl -sS https://here.now/api/auth/agent/verify-code \
  -H "content-type: application/json" \
  -d '{"email":"user@example.com","code":"ABCD-2345"}'
```

**方式二：面板获取**

访问 [here.now](https://here.now) 面板，登录后复制 API 密钥。

#### 存储 API 密钥

```bash
mkdir -p ~/.herenow && echo "<API_KEY>" > ~/.herenow/credentials && chmod 600 ~/.herenow/credentials
```

密钥读取优先级（从高到低）：
1. `--api-key` 参数
2. `$HERENOW_API_KEY` 环境变量
3. `~/.herenow/credentials` 文件（推荐）

### 步骤 2：创建站点

cURL 示例：
```bash
curl -X POST "https://here.now/api/v1/publish" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "files": [
      {
        "path": "index.html",
        "size": 1234,
        "contentType": "text/html",
        "hash": "sha256:abc123..."
      }
    ],
    "viewer": {
      "title": "我的网站",
      "description": "这是一个示例网站"
    }
  }'
```

JavaScript 示例：
```javascript
const response = await fetch('https://here.now/api/v1/publish', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_API_KEY'
  },
  body: JSON.stringify({
    files: [
      {
        path: 'index.html',
        size: 1234,
        contentType: 'text/html',
        hash: 'sha256:abc123...'
      }
    ],
    viewer: {
      title: '我的网站',
      description: '这是一个示例网站'
    }
  })
});

const data = await response.json();
console.log(data.siteUrl); // 站点访问地址
```

Python 示例：
```python
import requests
import hashlib

# 计算文件 hash
def file_hash(filepath):
    with open(filepath, 'rb') as f:
        return f"sha256:{hashlib.sha256(f.read()).hexdigest()}"

# 准备文件列表
files = [
    {
        "path": "index.html",
        "size": 1234,
        "contentType": "text/html",
        "hash": file_hash("index.html")
    }
]

# 创建站点
response = requests.post(
    "https://here.now/api/v1/publish",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "files": files,
        "viewer": {
            "title": "我的网站",
            "description": "这是一个示例网站"
        }
    }
)

data = response.json()
print(f"站点地址: {data['siteUrl']}")
```

### 步骤 3：上传文件

cURL 示例（并行上传）：
```bash
# 上传单个文件
curl -X PUT "https://storage.here.now/upload/..." \
  -H "Content-Type: text/html" \
  --data-binary @index.html

# 并行上传多个文件
curl -X PUT "https://storage.here.now/upload/1..." --data-binary @index.html &
curl -X PUT "https://storage.here.now/upload/2..." --data-binary @styles.css &
wait
```

JavaScript 示例：
```javascript
const uploadPromises = uploadUrls.map(async (uploadInfo) => {
  const fileContent = fs.readFileSync(uploadInfo.path);
  
  const response = await fetch(uploadInfo.url, {
    method: 'PUT',
    headers: uploadInfo.headers,
    body: fileContent
  });
  
  if (!response.ok) {
    throw new Error(`上传失败: ${uploadInfo.path}`);
  }
  
  return uploadInfo.path;
});

await Promise.all(uploadPromises);
console.log('所有文件上传成功!');
```

### 步骤 4：最终化站点

```bash
curl -X POST "https://here.now/api/v1/publish/abc123xyz/finalize" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "versionId": "01J..." }'
```

**成功响应：**
```json
{ "status": "published", "url": "https://abc123xyz.here.now" }
```

---

## 📚 API 参考

### 认证方式

| 模式 | 说明 |
|------|------|
| 匿名模式 | 省略 Authorization 头，24小时后自动过期 |
| 认证模式 | `Authorization: Bearer <API_KEY>`，永久存储 |

### 端点列表

| 方法 | 端点 | 描述 |
|------|------|------|
| `POST` | `/api/v1/publish` | 创建新站点 |
| `PUT` | `/api/v1/publish/:slug` | 更新站点（增量部署） |
| `POST` | `/api/v1/publish/:slug/finalize` | 最终化站点 |
| `POST` | `/api/v1/publish/:slug/claim` | 认领匿名站点 |
| `PATCH` | `/api/v1/publish/:slug/metadata` | 更新元数据 |
| `DELETE` | `/api/v1/publish/:slug` | 删除站点 |
| `GET` | `/api/v1/publishes` | 获取站点列表 |
| `POST` | `/api/v1/publish/:slug/duplicate` | 复制站点 |

### 错误代码

| 代码 | 含义 | 说明 |
|------|------|------|
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | API 密钥无效或缺失 |
| 403 | Forbidden | 没有权限执行此操作 |
| 404 | Not Found | 站点不存在 |
| 409 | Conflict | 站点 slug 已存在 |
| 429 | Too Many Requests | 超出速率限制 |

---

## 📊 服务限制对比

| 功能 | 匿名模式 | 认证模式（免费） |
|------|---------|-----------------|
| 最大文件大小 | 250 MB | 5 GB |
| 过期时间 | 24小时 | 永久/自定义 |
| 站点数量 | - | 500个 |
| 存储空间 | - | 10 GB |
| 速率限制 | 5次/小时/IP | 60次/小时 |
| 自定义域名 | ✗ | ✓ |
| 密码保护 | ✗ | ✓ |
| 站点管理 | claimToken | 完整 API |

---

## 🔐 密码保护

为站点添加密码保护，访问者需要输入密码才能查看内容：

```bash
curl -X PATCH "https://here.now/api/v1/publish/abc123xyz/metadata" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "password": "your-password" }'
```

移除密码保护：
```bash
curl -X PATCH "https://here.now/api/v1/publish/abc123xyz/metadata" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "password": null }'
```

---

## 📝 完整示例

### 部署单页应用脚本

```bash
#!/bin/bash
# deploy.sh - 单页应用部署脚本

API_KEY="your_api_key"
DIST_DIR="./dist"

# 1. 准备文件列表
files_json="["
for file in $DIST_DIR/*; do
  filename=$(basename "$file")
  size=$(stat -f%z "$file")
  mime=$(file -b --mime-type "$file")
  files_json+="{\"path\":\"$filename\",\"size\":$size,\"contentType\":\"$mime\"},"
done
files_json="${files_json%,}]"

# 2. 创建站点
response=$(curl -s -X POST "https://here.now/api/v1/publish" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"files\":$files_json}")

site_url=$(echo "$response" | jq -r '.siteUrl')
finalize_url=$(echo "$response" | jq -r '.upload.finalizeUrl')

# 3. 上传文件
echo "$response" | jq -r '.upload.uploads[] | "\(.path)\t\(.url)\t\(.headers | tojson)"' | \
while IFS=$'\t' read -r path url headers; do
  content_type=$(echo "$headers" | jq -r '.["Content-Type"]')
  curl -s -X PUT "$url" -H "Content-Type: $content_type" --data-binary "@$DIST_DIR/$path"
  echo "上传完成: $path"
done

# 4. 最终化
version_id=$(echo "$response" | jq -r '.upload.versionId')
curl -s -X POST "$finalize_url" \
  -H "Authorization: Bearer $API_KEY" \
  -d "{\"versionId\": \"$version_id\"}"

echo "部署成功: $site_url"
```

### GitHub Actions 自动部署

```yaml
# .github/workflows/deploy.yml
name: Deploy to here.now

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Build
        run: |
          npm ci
          npm run build
      
      - name: Deploy to here.now
        run: |
          npx here-now deploy --api-key ${{ secrets.HERENOW_API_KEY }}
```

---

## ❓ 常见问题

**Q: 匿名站点过期后如何恢复？**
> 匿名站点在24小时后会自动删除，无法恢复。如果需要永久存储，请在创建站点后使用 claimToken 将其认领到您的账户。

**Q: 如何绑定自定义域名？**
> 对于子域名，添加 CNAME 记录指向 `fallback.here.now`。对于根域名，添加 ALIAS 记录和 TXT 验证记录。SSL 证书会自动配置。

**Q: 支持哪些文件类型？**
> 支持所有静态文件类型，包括 HTML、CSS、JavaScript、图片（PNG、JPG、SVG、WebP）、字体、PDF、视频等。

**Q: 部署后多久可以访问？**
> 通常情况下，最终化完成后立即可以访问。全球 CDN 传播最多需要 60 秒。

**Q: 如何更新已发布的站点？**
> 使用 PUT `/api/v1/publish/:slug` 端点进行增量部署。提供文件的 SHA-256 hash，系统会自动识别需要更新的文件。

---

## 📄 许可证

本项目基于 here.now 官方文档创建的中文教程网站。

---

<p align="center">
  Made with ❤️ using <a href="https://here.now">here.now</a>
</p>
