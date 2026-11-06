# PRTS Terminal

PRTS 风格终端界面，基于 Vue 3 + Vite。包含开屏神经链接认证动画、四屏滚动主页，以及角色提示词编辑器（分块标签、自定义指令、导出到剪贴板）。

## 本地开发

```bash
npm install
npm run dev      # http://127.0.0.1:5173/
npm run build    # 产物输出到 dist/
```

Node.js 建议 22.x（Vite 8 要求 20.19+ / 22.12+）。

## 目录

```text
src/
├── components/
│   ├── SplashScreen.vue   开屏认证动画
│   ├── HomeView.vue       四屏主页
│   └── EditorView.vue     提示词编辑器
├── App.vue                视图路由
├── main.js                入口
└── style.css              全局变量与 reset
```

## 部署

推送到 `main` 分支后，`.github/workflows/deploy.yml` 会自动执行 `npm ci` → `npm run build`，并把 `dist/` 发布到 GitHub Pages。

首次部署需要在仓库 **Settings → Pages** 将 Build and deployment 的 Source 设为 **GitHub Actions**。
