# leoxouyang.github.io

Zhenghao Lu（陆政豪）的求职用个人网站，基于 [Astro](https://astro.build)，push 到 `main` 后由 GitHub Actions 自动部署到 GitHub Pages。

## 页面

| 路径 | 文件 |
| --- | --- |
| `/` | `src/pages/index.astro` |
| `/portfolio` | `src/pages/portfolio.astro` |
| `/cv` | `src/pages/cv.astro` |
| 404 | `src/pages/404.astro` |

- 共享信息（姓名、邮箱、LinkedIn）：`src/site.ts`
- 页面骨架、SEO / OG 标签：`src/layouts/Layout.astro`
- 顶部导航：`src/components/Header.astro`
- 新增页面后记得加进 `src/pages/sitemap.xml.ts`

## /resume/（自用简历工具）

`https://leoxouyang.github.io/resume/` 不从主页进入，是 fork 来的 [markdown-resume](https://github.com/leoxouyang/markdown-resume) 简历制作工具。
部署时由 `.github/workflows/deploy.yml` 拉取该仓库构建后放进 `public/resume/`，详见 `technical_document.md`。
已在 `robots.txt` 中禁止搜索引擎收录。

## 本地开发

```sh
npm install
npm run dev      # http://localhost:4321
npm run build    # 输出到 dist/
```
