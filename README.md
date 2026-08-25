# 家中日用品清单（PWA）

用手机拍照管理家中日用品的轻量应用：拍照入库、三级分类、关键词检索、存货扣减、到期提醒、导出/导入备份，可安装到主屏、离线可用。

- 形态：单页 PWA（纯前端，无需服务器）
- 存储：物品数据存于**手机浏览器本地（IndexedDB）**，不上传任何云端
- 部署：GitHub Pages（HTTPS），供手机真机调摄像头、安装到主屏

---

## 1. 本地预览（电脑测试用）

> 说明：在电脑浏览器预览时**摄像头不可用**（桌面无后置摄像头），仅用于检查界面与逻辑；真机拍照请在手机上打开部署后的网址。

在本文件夹下起一个本地静态服务（localhost 是安全上下文，可验证 Service Worker / 安装）：

```bash
cd 日用品清单_v3_部署包
python3 -m http.server 8080
```

然后浏览器打开 `http://localhost:8080`。

---

## 2. 部署到手机（GitHub Pages，一次性）

> 前置：需要一个 GitHub 账号。GitHub Pages 对个人仓库**免费**，但免费账号要求仓库为**公开（public）**才能开启 Pages。

1. 在 github.com 新建一个**空仓库**（如 `household`），不要勾选自动生成 README。
2. 把本文件夹（`日用品清单_v3_部署包/`）里的全部文件推上去（见第 3 节命令）。
3. 仓库页 → **Settings → Pages → Build and deployment → Source** 选
   **Deploy from a branch** → Branch 选 `main`、目录 `/ (root)` → **Save**。
4. 约 1 分钟后访问：`https://<你的用户名>.github.io/<仓库名>/`
   （本例为 `https://jessstarw.github.io/household/`）
5. 手机浏览器打开该地址 → 允许摄像头 → 浏览器菜单「添加到主屏幕」完成安装。

---

## 3. 版本更新后，如何推送到 GitHub（核心流程）

应用采用**「代码与数据完全解耦」**设计：你的物品数据始终在手机浏览器本地，
与网页代码无关。因此**升级功能 = 只换网页文件**，历史物品自动保留、无需迁移。

每次需要新功能 / 修 bug 时：

1. **你告诉我需求**（如「加低库存提醒」）。
2. **我（AI）在工作区做出新版本**，按留痕规则先快照旧版到 `历史版本/`，
   再把新版文件更新进本部署包文件夹（含 `index.html` / `sw.js` / `manifest.webmanifest` /
   `icon-*.png` / `.nojekyll` / `README.md`）。
   - 关键：我会把 `sw.js` 里的缓存版本号 +1，确保手机刷新能拉到新代码。
3. **你只需在电脑上执行下面三条命令**（在本文件夹内）：

```bash
cd 日用品清单_v3_部署包
git add -A
git commit -m "升级到 v4：分类搜索+删改+数量统计"
git push -u origin main
```

> 首次推送需 GitHub 身份验证：用户名用你的 GitHub 账号；**密码处粘贴
> Personal Access Token（不是账号密码）**，令牌需勾选 `repo` 权限。
> 生成地址：github.com → Settings → Developer settings → Personal access tokens。

4. **手机打开 App，刷新一下即可**（极少数情况多刷一次或重开 App，
   因 Service Worker 需接管新缓存）。物品数据原样保留。

### 数据连续性说明
- 一直用**同一个部署网址**打开 → 数据天然连续，升级零操作。
- 换网址 / 换手机 / 清缓存 → 用 App 内「设置 → 导出 JSON」备份，
  再到新环境「导入 JSON」恢复（schema 向前兼容，旧备份缺字段不报错）。
- 物品数据**不会**进入 GitHub 仓库，仅前端代码上线。

---

## 4. 常见问题

| 问题 | 原因 / 解决 |
|---|---|
| Pages 提示「Upgrade or make this repository public」 | 免费账号需把仓库设为**公开**。代码无隐私风险（数据在本地）。设置 → General → Danger Zone → Change visibility → Make public |
| 推送报 `Authentication failed` / `Password authentication is not supported` | GitHub 已停用密码登录 git，需用 **Personal Access Token** 代替密码 |
| 手机打开后还是旧版 | 刷新；若仍旧，重开 App 一次（Service Worker 接管）。确认已 `git push` 成功、Pages 已重新部署（约 1 分钟） |
| 电脑预览摄像头用不了 | 正常，桌面无后置摄像头；真机拍照请在手机打开部署网址 |
| 想真正「没打开也弹到期通知」 | 当前为 PWA 轻档（打开即提醒）；系统级推送需 Web Push + 后端，可后续升级 |

---

## 5. 当前版本功能（v4）

- 新增物品：调用摄像头拍照（自动压缩）、选分类、名称、数量+单位、存放位置、到期时间
- **分类选择增加搜索**：输入即过滤现有分类；搜不到时一键「新建大类」
- **分类栏支持删改**：每个大类/中类/小类均可重命名、删除；删除分类不删物品
- **分类数量统计**：分类栏实时显示每个类下有多少件物品
- 清单：卡片展示、关键词检索（名称/分类/位置）、临期横幅（到期前 30 天）
- 存货：用掉 / 补货 / 编辑 / 删除；数量为 0 自动标「已用完」
- 设置：导出 / 导入 JSON 备份、安装到主屏

分类锚定公开标准（京东居家/厨具、日用百货百科）。
