# jiufu-fishing

九洑环湖路亚基地 · 官方网站

## 访问地址

| 渠道 | 地址 |
|------|------|
| 正式站（自定义域名） | https://jiufu.evan-minho.com |
| GitHub Pages 备用 | https://evan-lei.github.io/jiufu-fishing/ |
| GitHub 仓库 | https://github.com/evan-lei/jiufu-fishing |

## 技术栈

- **前端**：纯 HTML / CSS / JavaScript，无框架依赖
- **托管**：GitHub Pages（main 分支自动部署）
- **域名**：Cloudflare DNS，CNAME 指向 `evan-lei.github.io`
- **素材**：腾讯云 COS `jiufu-1256297898`（ap-beijing）

## 目录结构

```
jiufu-fishing/
├── index.html      ← 网站主页（唯一 HTML 文件）
├── CNAME           ← GitHub Pages 自定义域名：jiufu.evan-minho.com
├── SKILL.md        ← 项目管理指南（脱敏版，含部署/COS/DNS 流程）
└── README.md
```

> 素材（图片/视频）均存放于腾讯云 COS，不提交进 git。

## 页面内容

| 模块 | 内容 |
|------|------|
| Hero | 全屏背景、logo、slogan「一湾九洑水，万里路亚情」 |
| Stats | 5000斤放鱼量 / 5种鱼 / 2钓区 / 11小时营业 |
| 钓场分区 | 大湖区（全天¥180/下午¥120）+ 雷强区（¥180）+ 露营区 |
| 鱼种图鉴 | 翘嘴 / 大口黑鲈 / 青鱼 / 草鱼 / 鳡鱼 / 鳙鱼 |
| 图片画廊 | 4 Tab：钓场环境 / 钓友鱼获 / 露营风光 / 九洑小饮 |
| 配套设施 | 九洑小饮咖啡厅 + 帐篷露营 + 亲子娱乐 |
| 位置导航 | 地图 + 一键复制地址 + lightbox 图片预览 |

## 快速部署

```bash
cd /Users/user/Documents/jiufu-fishing
git add .
git commit -m "update content"
# 推送（token 见本地 ~/.cursor/skills/jiufu-fishing/SKILL.md）
git remote set-url origin https://evan-lei:<TOKEN>@github.com/evan-lei/jiufu-fishing.git
git push origin main
git remote set-url origin https://github.com/evan-lei/jiufu-fishing.git
```

推送后约 30 秒自动生效，强制刷新：**Cmd+Shift+R**

## 管理指南

详见 [`SKILL.md`](./SKILL.md)，包含：COS 上传流程、图片压缩规范、DNS/HTTPS 配置、换电脑恢复步骤。
