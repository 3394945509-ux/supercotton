# SuperCotton · 超级棉花 — 网页介绍

这是 SuperCotton 1.2.1 的对外介绍网页。视觉从原来的深色科幻版式改为 **奶油纸张 + 衬线标题 + 产品目录卡**：把十个功能当成十件“屏幕工具”陈列，而不是把工具堆在一起。

`SuperCotton.html` 是单文件主页面，所有页面结构、CSS 和交互都在这一个文件里；`assets/` 放真实软件截图、棉织物照片和图标。`index.html` 是 Cloudflare Pages / 静态站入口，会自动跳到主页面。

## 文件结构

```text
SuperCotton-Web/
├── SuperCotton.html      # 主页面：首屏 + 10 个叠放板块 + 演示弹窗 + 主题切换
├── index.html            # 静态站入口，自动打开主页面
├── assets/               # 页面全部图片，WebP 压缩
│   ├── record_ui_clean.webp
│   ├── annotator_ui.webp
│   ├── editor_ui_clean.webp
│   ├── pin_card.webp
│   ├── floating_card.webp
│   ├── delete_dialog.webp
│   ├── share_dialog.webp
│   ├── preview_hover.webp
│   ├── logo.webp
│   ├── cotton_still.webp      # 起点板块的棉枝照片
│   └── cotton_texture.webp    # 首屏的棉布纹理
└── README.md
```

## 页面内容

- **首屏**：奶油纸底、三行 `Noto Serif SC` 标题、土黄色收束句；右侧是一张 544×492 的产品目录卡，内含真实录屏界面、`SC—01 / RECORDING OBJECT` 编号、版本角标与下载入口。
- **10 个滚动叠放板块**：起点、录屏、截屏标注、钉图、剪辑、素材库、主题、可靠性、关于作者、下载。每个板块独立编号、独立说明、独立演示入口。
- **五套演示弹窗**：录屏、截屏、钉图、剪辑、素材库。弹窗改成奶油纸 + 液态玻璃深色遮罩，保留 `backdrop-filter` 模糊。
- **五套主题实时切换**：极光青、深海蓝、石墨灰、日出橙、森林绿。色板按钮带编号和名称，点击即时改强调色；390px 下五块色板一行排布。
- **钉图示例**：页面内可拖动、滚轮缩放、双击复位；演示不是真实桌面 Pin 窗口。
- **安装包入口**：首屏、顶栏、下载板块共 4 个 `data-download` 入口；部署后统一使用 `RELEASE_URL`。
- **安装包 SHA256 一键复制**：复制失败时自动选中哈希文本，提示手动复制。
- **移动端**：单列布局，390px 宽无横向溢出；面板取消 sticky，演示弹窗内截图与文字上下排列。

## 本地预览

在 `SuperCotton-Web` 目录执行：

```powershell
python -m http.server 8765
```

然后打开：

```text
http://127.0.0.1:8765/SuperCotton.html
```

本地预览时，下载按钮会回退到网页目录上一层的 `SuperCotton-Setup-1.2.1.exe`，所以请把安装包放在 `SuperCotton-Web` 的父目录再测试下载。

## 安装包托管（必须做）

安装包大小为 **202,211,670 B（约 202 MB）**，SHA256：

```text
DCE38D38254CD53790234EFF6562348695C2BF8B5C738B9609B66D7C9DD8A1D4
```

不要把 exe 提交进 GitHub 普通仓库，也不要放进 Cloudflare Pages 的站点目录：

- GitHub 普通仓库对单文件有大小限制，体验也不好；
- Cloudflare Pages 单文件上限为 25 MiB，放不下 202 MB 的安装包。

推荐放在 **GitHub Releases**（单资源上限远大于 202 MB），网页只保留下载跳转；也可以用 **Cloudflare R2**。

### 方案 A：GitHub Releases（推荐）

1. 在 GitHub 新建仓库（例如 `SuperCotton`），只提交网页文件：

   ```powershell
   git init
   git add index.html SuperCotton.html assets README.md
   git commit -m "Add SuperCotton 1.2.1 product site"
   git branch -M main
   git remote add origin https://github.com/<你的 GitHub 用户名>/SuperCotton.git
   git push -u origin main
   ```

2. 打开仓库的 **Releases → Draft a new release**，创建标签 `v1.2.1`，上传 `SuperCotton-Setup-1.2.1.exe` 作为 release asset 并发布。
3. 复制安装包资源的下载地址，格式类似：

   ```text
   https://github.com/<你的 GitHub 用户名>/SuperCotton/releases/download/v1.2.1/SuperCotton-Setup-1.2.1.exe
   ```

4. 打开 `SuperCotton.html`，找到文件底部脚本里的：

   ```javascript
   const RELEASE_URL = "";
   ```

   把空字符串替换成上一步的地址。保存后重新部署。
5. 页面里所有 `data-download` 的入口会自动改用这个地址，并去掉本地 `download` 属性，直接用浏览器打开 release 资源。

如果不配置 `RELEASE_URL`，部署后的下载按钮会显示一条配置提示，而不是跳到 404；本地预览不受影响。

### 方案 B：Cloudflare R2

1. 在 Cloudflare R2 建 bucket，上传安装包。
2. 给对象配置公开访问域名，复制公开 URL。
3. 把 `RELEASE_URL` 改成该 R2 公开 URL。
4. 注意 CORS 与 Content-Disposition；如果希望浏览器直接下载，建议在 R2 侧设置 `Content-Disposition: attachment`，或保留页面里的跳转方式。

## Cloudflare Pages 部署

1. 把上面的 GitHub 仓库连接到 Cloudflare Pages。
2. Framework preset 选 **None**。
3. Build command 留空。
4. Build output directory 填 **/**（仓库根目录）。
5. 部署后访问 Cloudflare 给出的域名；根路径会由 `index.html` 自动转到 `SuperCotton.html`。
6. 在 Cloudflare 的 **Environment variables** 里不需要放安装包；安装包仍在 GitHub Releases / R2。
7. 如果使用 Cloudflare Pages 的“直接上传”，只上传 `index.html`、`SuperCotton.html`、`assets/`、`README.md`，不要上传 exe。

## 设计参考与素材

- 版式参考：`https://www.stylekit.top/zh/templates/ecommerce-product` 的电商产品目录语言——奶油纸、细横线、大留白、衬线标题、产品编号、黑色胶囊按钮、浅色产品框。
- 棉织物照片来自 Pexels 的棉花 / 棉布搜索结果，压缩为 `cotton_still.webp` 与 `cotton_texture.webp`。Pexels License 允许免费使用且通常不要求署名；具体摄影师未逐一确认，公开页未强制署名。若需要严谨署名，请在发布前补上摄影师与图片页链接。
- 软件界面截图来自 SuperCotton 本机真实运行画面，统一转成 WebP；页面不展示本机目录、盘符或测试路径。

## 验收记录

在 Edge / Chromium、1440×900 与 390×844 下，用真实滚动 + 指针点击路径复测：

- [x] 首屏产品卡完整，`Noto Serif SC` 生效，无横向溢出。
- [x] 10 个板块全部进入 active 状态，均为 1240×738；控制台无 error、页面无异常。
- [x] 录屏、截屏、钉图、剪辑、素材库 5 个演示弹窗全部打开；关闭按钮与 Esc 退出。
- [x] 五套主题切换后 `--accent` 与主题名同步：`#2f6b5c / #46688b / #676966 / #b96e3f / #506f54`。
- [x] 钉图示例真实指针拖动、滚轮缩放、双击复位均有 `transform` 变化。
- [x] 4 个 `data-download` 指向 `../SuperCotton-Setup-1.2.1.exe`。
- [x] 390px 下 `scrollWidth === innerWidth === 390`，五块主题色板一行排布。
- [x] SHA256 复制通过：真实指针点击后 `navigator.clipboard.readText()` 返回完整哈希；若浏览器权限被拒，页面会自动选中哈希文本提示手动复制。
- [ ] 真实部署后的 `RELEASE_URL` 尚未配置，下载入口目前只完成本地回退与配置提示逻辑。

## 已知边界

- 部署后必须先完成安装包托管并填写 `RELEASE_URL`，否则下载入口只会显示配置提示。
- 网页不包含账号、支付、评论、统计后台；它是静态介绍页。
- 安装包签名 / 杀毒误报取决于 Windows 与安全软件环境，网页侧无法替代代码签名。
- PPT 是独立交付物，不在这个网页目录内。

## 版本

- 产品版本：SuperCotton 1.2.1
- 站点内容：2026
- 运行时：Windows 10 / 11 · 64-bit
- 安装包：单文件安装器，免 Python，内置 ffmpeg / Qt / OpenCV
