# 部署说明 · AI 产业框架地图（GitHub Pages）

> 目标：把单文件地图发布到互联网，并让每周 `ai-stack-weekly-pulse` 自动更新后**自动重新部署**，使线上版本保持"活地图"。

---

## 1. 一次性发布（手动，约 10 分钟）

1. 新建 GitHub 仓库（公开或私有皆可，Pages 公开仓库免费）。
2. 把以下文件放进仓库根目录：
   - `AI产业框架地图.html` → **复制/重命名为 `index.html`**（GitHub Pages 默认在根目录找 `index.html`，且中文文件名做 URL 不友好）。
   - `og.png`（社交分享图，必须和 index.html 同级，供 `og:image` 引用）。
   - 可选：把 `SPEC`、`CHANGELOG`、`backups/` 也提交，作为工程档案。
3. Settings → Pages → Source 选 `Deploy from a branch`，分支 `main`、目录 `/ (root)`，保存。
4. 等 1–2 分钟，得到 URL：`https://<用户名>.github.io/<仓库名>/`。
   - 如绑定自定义域名：Pages 里填 Custom domain，并勾选 **Enforce HTTPS**。
5. **回填域名**：把 `index.html` 里 4 处 `REPLACE-WITH-YOUR-DOMAIN` 换成上面的正式 URL（`og:url` / `canonical` / `og:image` / `twitter:image`）。社交预览图必须用绝对地址。
6. 用 [opengraph.xyz](https://www.opengraph.xyz/) 等工具验证分享卡片显示正常。

> 注意：源文件仍以 `AI产业框架地图.html` 为工作主文件（SPEC/任务都引用它）；`index.html` 是它的发布副本。发布时同步两者即可，见 §2。

---

## 2. 自动更新 → 自动重新部署（活地图的命门）

现状：`ai-stack-weekly-pulse` 每周改的是**本地 `AI产业框架地图.html`**。一旦发布，线上 `index.html` 不会变。要让"每周自动更新"对读者成真，每周任务跑完后必须把改动推到仓库。

**做法（在每周任务末尾追加，见 SPEC §6 新增的第 7–8 步）：**

```bash
# 0) 前提：仓库已 clone 到任务可访问的目录，且配置了可推送的凭据
#    （Personal Access Token / Deploy Key / gh auth），git user.name/email 已设
cd <仓库本地路径>

# 1) 把更新后的工作文件同步为发布文件
cp "AI产业框架地图.html" index.html

# 2) 提交并推送 —— GitHub Pages 会自动重建（约 1–2 分钟）
git add -A
git commit -m "pulse: $(date +%F) 第 N 批自动更新"
git push origin main
```

**鉴权选项（任选其一）：**
- **PAT**：仓库 remote 用 `https://<token>@github.com/<user>/<repo>.git`，token 存在任务的密钥环境里，勿写进文件。
- **Deploy Key**：生成 SSH key，公钥加到仓库 Deploy keys（勾选写权限），remote 用 SSH。
- **gh CLI**：`gh auth login` 后用 `gh` 推送。

**可选更稳的方案**：仓库加一个 GitHub Actions（push 即部署到 Pages），任务只需 push，部署由 Action 负责——便于加"部署前自动跑 §9 验证"这一关（HTMLParser、PULSE 标记、状态 ID、链接体检），验证不过就拒绝发布。

---

## 3. 发布前一次性检查清单

- [ ] 4 处 `REPLACE-WITH-YOUR-DOMAIN` 已回填正式域名
- [ ] `og.png` 与 `index.html` 同级、可访问
- [ ] 真机/多浏览器（Chrome/Safari/Firefox + 手机）实际渲染：明暗两模式、4 个断点无回退、无 console error
- [ ] 链接体检：第 3 批及全站外链可达
- [ ] 估值/份额等关键数字最后一次核对（或标注口径与时间）
- [ ] 免责声明在位（footer：非投资建议）
- [ ] HTTPS 已强制

---

## 4. 维护节奏

- 每周一：`ai-stack-weekly-pulse` 自动采集 → 严过 §0.5 闸 → 更新 pulse + 状态 ID → 自动 commit/push（§2）。
- 每季度：校准 12 层正文数字、玩家名单。
- 每半年：§03 维度卡、结构性力维度复盘。
- 每次大改：跑 SPEC §9 验证清单 + 备份旧版到 `backups/`。
