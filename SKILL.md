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

> ⚠️ 不要用 COS 托管 HTML 页面（大陆节点未备案会强制下载），HTML 统一走 GitHub Pages。

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

```bash
export PATH="$PATH:$HOME/Library/Python/3.9/bin"
coscmd config -a <SecretId> -s <SecretKey> -b jiufu-1256297898 -r ap-beijing

# 批量上传目录
coscmd -b jiufu-1256297898 -r ap-beijing upload -rs <本地目录>/ /
```

或用 Python SDK：

```python
import warnings; warnings.filterwarnings('ignore')
from qcloud_cos import CosConfig, CosS3Client

config = CosConfig(Region='ap-beijing', SecretId='<见本地SKILL>', SecretKey='<见本地SKILL>')
client = CosS3Client(config)
with open('local_file.jpg', 'rb') as fp:
    client.put_object(Bucket='jiufu-1256297898', Body=fp, Key='目录/文件名.jpg', ACL='public-read')
```

### 上传前必须压缩（节省流量费）

取原始 vs 压缩版中**更小的**上传。

```bash
# JPG/JPEG：最大1920px，质量82
sips -Z 1920 -s formatOptions 82 input.jpg --out output.jpg
# PNG：最大1920px
sips -Z 1920 input.png --out output.png
# 视频（720p）
ffmpeg -i input.mp4 -c:v libx264 -crf 26 -preset fast \
  -vf scale=-2:720 -c:a aac -b:a 96k -movflags +faststart output.mp4
```

### 上传前检查尺寸，规划 UI 布局

```bash
for f in *.jpg *.jpeg *.png *.JPG *.JPEG *.PNG; do
  [ -f "$f" ] || continue
  w=$(sips -g pixelWidth "$f" | awk '/pixelWidth/{print $2}')
  h=$(sips -g pixelHeight "$f" | awk '/pixelHeight/{print $2}')
  orient=$([ "$w" -gt "$h" ] && echo "横图" || echo "竖图")
  echo "$orient  ${w}x${h}  $f"
done
```

布局原则：
- **全横图**：`grid-template-columns: repeat(auto-fill, minmax(240px,1fr))`
- **全竖图**：单列或并排 2 列
- **混排**：竖图 `grid-row: 1 / span N`，横图堆叠另一列

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

如 token 失效，重新生成：`https://github.com/settings/tokens/new`（勾选 `repo` 权限）。

---

## DNS 配置（已完成）

- 域名 `evan-minho.com` 由 **Cloudflare** 管理
- CNAME 记录：`jiufu` → `evan-lei.github.io`（DNS only，灰色云朵）
- 仓库根目录 `CNAME` 文件内容：`jiufu.evan-minho.com`

### HTTPS 证书重新签发（证书失效时）

```bash
curl -X DELETE https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN>"
sleep 3
curl -X POST https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN>" -H "Content-Type: application/json" \
  -d '{"source": {"branch": "main", "path": "/"}}'
sleep 15
curl https://api.github.com/repos/evan-lei/jiufu-fishing/pages \
  -H "Authorization: token <TOKEN>" | python3 -c "
import sys,json; d=json.load(sys.stdin)
print('cert:', (d.get('https_certificate') or {}).get('state'))"
```

---

## 页面结构（index.html）

| 模块 | 说明 |
|------|------|
| Hero | 全屏背景图、logo、slogan、开业公告、CTA 按钮 |
| Stats Bar | 5000斤 / 5种 / 2区 / 11h |
| 钓场分区 | 大湖（全天180/下午120）+ 雷强区（180）+ 露营区 |
| 投放鱼种 | 6种鱼卡片，含重量 |
| 图片画廊 | 4 tab（钓场环境/钓友鱼获/露营风光/九洑小饮），支持 lightbox |
| 配套设施 | 咖啡厅 + 露营 + 亲子 |
| 位置导航 | 上1下2图片布局，一键复制「九洑环湖路亚」，支持 lightbox |
| Footer | logo + slogan + 营业时间 |

---

## 移动端响应式规范

> 断点：`@media (max-width: 768px)`

### 通用横向滑动（h-scroll）

多列卡片在手机上改为横向滑动，避免纵向串联。在需要横滑的 grid 容器加 `.h-scroll` class：

```css
.h-scroll {
  display: flex !important;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  gap: 16px;
  padding-bottom: 16px;
  /* 突破 section 两侧 padding，让卡片贴边 */
  margin-left: -5vw; margin-right: -5vw;
  padding-left: 5vw; padding-right: 5vw;
  scrollbar-width: none;
}
.h-scroll::-webkit-scrollbar { display: none; }
```

卡片子元素设置固定宽度 + snap：
```css
/* 钓场分区：82vw（右侧微露提示可滑动） */
.zones-grid .zone-card { flex: 0 0 82vw; min-width: 0; scroll-snap-align: start; }
/* 配套设施：78vw */
.facilities-grid .facility-card { flex: 0 0 78vw; min-width: 0; scroll-snap-align: start; }
```

圆点指示器 HTML（放在 grid 后面）：
```html
<div class="scroll-dots" id="xxxDots">
  <span class="active"></span><span></span><span></span>
</div>
```

圆点联动 JS（通用函数）：
```js
function bindScrollDots(gridSel, dotsId) {
  const grid = document.querySelector(gridSel);
  const dots = document.querySelectorAll('#' + dotsId + ' span');
  if (!grid || !dots.length) return;
  grid.addEventListener('scroll', () => {
    const cardW = (grid.querySelector(':scope > *') || grid).offsetWidth;
    const idx = Math.min(Math.round(grid.scrollLeft / cardW), dots.length - 1);
    dots.forEach((d, i) => d.classList.toggle('active', i === idx));
  }, { passive: true });
}
bindScrollDots('.zones-grid', 'zonesDots');
bindScrollDots('.facilities-grid', 'facilDots');
```

### 各模块移动端处理方式

| 模块 | 桌面 | 手机 |
|---|---|---|
| Stats Bar | `auto-fit minmax(160px)` | 强制 `repeat(4,1fr)` + 缩小字号/内边距 |
| 钓场分区 | 3列 grid | `.h-scroll` 横向滑动，每张 82vw |
| 鱼种图鉴 | `auto-fit minmax(140px)` | 固定 `repeat(3,1fr)`，紧凑 3×2 |
| Gallery Tabs | flex wrap | `flex:1` 等宽铺满一行，不换行 |
| 图片画廊 | `auto-fill minmax(240px)` | `repeat(2,1fr)` 2列 |
| 配套设施 | 3列 grid | `.h-scroll` 横向滑动，每张 78vw |
| 位置导航 | 左右两列 | `grid-template-columns:1fr`，上下堆叠 |

---

## 图片/视频交互规范

### Lightbox 规范

```js
// 触摸滑动
lb.addEventListener('touchstart', e => { _tx = e.touches[0].clientX; _ty = e.touches[0].clientY; }, { passive: true });
lb.addEventListener('touchend', e => {
  const dx = e.changedTouches[0].clientX - _tx, dy = e.changedTouches[0].clientY - _ty;
  if (Math.abs(dx) > 50 && Math.abs(dx) > Math.abs(dy) * 1.5) lbNav(dx < 0 ? 1 : -1);
}, { passive: true });

// 键盘导航
document.addEventListener('keydown', e => {
  if (!lb.classList.contains('open')) return;
  if (e.key === 'Escape') closeLb();
  if (e.key === 'ArrowLeft') lbNav(-1);
  if (e.key === 'ArrowRight') lbNav(1);
});
```

### 视频预览规范（如未来添加视频）

- 禁止 `autoplay`，改为点击播放（COS 按下行流量计费，autoplay 白烧钱）
- 使用 `preload="metadata"` 只加载首帧
- 播放按钮用**白底深色图标**（视频首帧偏暗，黑色按钮会隐身）

```html
<div class="pthumb">
  <video src="https://jiufu-1256297898.cos.ap-beijing.myqcloud.com/xxx.mp4"
         muted playsinline preload="metadata"></video>
  <div class="pthumb-play"></div>
</div>
```

---

## 工作流：更新网站内容

1. 修改 `/Users/user/Documents/jiufu-fishing/index.html`
2. 如有新图片：压缩 → 上传 COS → 更新 HTML 中的 URL
3. 提交推送（见上方推送命令）
4. 30 秒后刷新 `https://jiufu.evan-minho.com` 验证（**强制刷新：Cmd+Shift+R**）

---

## 换电脑时恢复 SKILL

1. Clone 仓库：`git clone https://github.com/evan-lei/jiufu-fishing.git`
2. 复制 SKILL 到 Cursor：
   ```bash
   mkdir -p ~/.cursor/skills/jiufu-fishing
   cp SKILL.md ~/.cursor/skills/jiufu-fishing/SKILL.md
   ```
3. 将占位符替换为真实密钥：
   - GitHub Token：`https://github.com/settings/tokens`
   - 腾讯云密钥：`https://console.cloud.tencent.com/cam/capi`
4. 重启 Cursor 即可生效

---

## 注意事项

- 单文件严禁超过 25MB（GitHub Pages 限制），素材必须放 COS
- 图片/视频不要提交进 git（在 `.gitignore` 中排除）
- 本地原始素材备份：`/Users/user/Desktop/九洑/source/`
- 更新此 SKILL 后，记得脱敏后同步到 GitHub 仓库的 `SKILL.md`
