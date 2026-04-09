---
name: jiufu-fishing
description: Manage the 九洑环湖路亚基地 website (jiufu.evan-minho.com). Use when the user wants to update the fishing club site, modify content, upload new images, fix layout, or deploy changes to the jiufu-fishing repo.
---

# 九洑环湖路亚基地 网站管理指南

## 站点信息

| 项目 | 值 |
|------|-----|
| 公开地址 | `https://jiufu.evan-minho.com` |
| GitHub 仓库 | `https://github.com/evan-lei/jiufu-fishing` |
| 本地工作目录 | `/Users/user/Documents/jiufu-fishing` |
| GitHub 用户名 | `evan-lei` |

**部署方式**：`git push` 到 GitHub main 分支后，GitHub Pages 自动部署，约 30 秒生效。

> ⚠️ 密钥信息（COS SecretId/SecretKey、GitHub Token）保存在本地 `~/.cursor/skills/jiufu-fishing/SKILL.md`，不随此文件同步到 GitHub。

---

## 腾讯云 COS（图片/素材存储）

| 字段 | 值 |
|------|----|
| Bucket | `jiufu-1256297898` |
| 地域 | `ap-beijing` |
| SecretId | 见本地 SKILL |
| SecretKey | 见本地 SKILL |
| 公开 Base URL | `https://jiufu-1256297898.cos.ap-beijing.myqcloud.com/` |

### 已上传素材结构
```
/
├── logo.webp
├── 2.jpg               ← 地图截图
├── 3.jpg               ← 导航截图
├── 4.jpg               ← 钓场正门
├── 钓场环境/           ← IMG_1722-1746.JPG（7张）
├── 钓友鱼获/           ← IMG_1724-1747.JPG（9张）
├── 咖啡厅/             ← IMG_1729-1741.JPG（5张）
└── 露营环境/           ← IMG_1726-1744.JPG（4张）
```

### 上传新图片
```python
import warnings; warnings.filterwarnings('ignore')
from qcloud_cos import CosConfig, CosS3Client

config = CosConfig(Region='ap-beijing',
    SecretId='<见本地SKILL>',
    SecretKey='<见本地SKILL>')
client = CosS3Client(config)

with open('local_file.jpg', 'rb') as fp:
    client.put_object(Bucket='jiufu-1256297898', Body=fp,
                      Key='目录/文件名.jpg', ACL='public-read')
```

---

## 推送到 GitHub

```bash
cd /Users/user/Documents/jiufu-fishing
git add .
git commit -m "描述改动"
git remote set-url origin https://evan-lei:<TOKEN见本地SKILL>@github.com/evan-lei/jiufu-fishing.git
git push origin main
git remote set-url origin https://github.com/evan-lei/jiufu-fishing.git
```

如 token 失效，在此重新生成：`https://github.com/settings/tokens/new`（勾选 `repo` 权限）。

---

## DNS 配置（已完成，仅供参考）

- 域名 `evan-minho.com` 由 **Cloudflare** 管理
- 已添加 CNAME 记录：`jiufu` → `evan-lei.github.io`（DNS only，灰色云朵）
- 仓库根目录有 `CNAME` 文件内容为 `jiufu.evan-minho.com`

### HTTPS 证书重新签发（如证书失效）

```bash
# 1. 删除 Pages
curl -X DELETE https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN见本地SKILL>"

# 2. 重建 Pages
curl -X POST https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN见本地SKILL>" \
  -H "Content-Type: application/json" \
  -d '{"source": {"branch": "main", "path": "/"}}'

# 3. 等 15 秒检查证书状态
sleep 15 && curl https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN见本地SKILL>" | python3 -c "
import sys,json; d=json.load(sys.stdin)
print('cert:', (d.get('https_certificate') or {}).get('state'))
"
```

---

## 页面结构（index.html）

| 模块 | 说明 |
|------|------|
| Hero | 全屏背景图、logo、slogan、开业公告、CTA 按钮 |
| Stats Bar | 5000斤 / 5种 / 2区 / 11h 四项数据 |
| 钓场分区 | 大湖区（全天180/下午120）+ 雷强区（180）+ 露营区 |
| 投放鱼种 | 6种鱼卡片，含重量 |
| 图片画廊 | 4 个 tab（钓场环境/钓友鱼获/露营风光/九洑小饮），支持 lightbox |
| 配套设施 | 咖啡厅 + 露营 + 亲子 |
| 位置导航 | 上1下2图片布局，一键复制「九洑环湖路亚」，支持 lightbox |
| Footer | logo + slogan + 营业时间 |

## 本地素材备份

原始素材在：`/Users/user/Desktop/九洑/source/`
