# **技术文档：将 markdown-resume 部署到主站子路径 /resume/（GitHub Pages）**



## **1. 目标与约束**





目标：把 leoxouyang/markdown-resume 构建出的静态站点，作为你主站（leoxouyang.github.io）的一部分，发布到子路径 /resume/，最终访问地址为：



* https://leoxouyang.github.io/resume/





约束：GitHub Pages 对“单页应用（SPA）路由”和 “子路径资源前缀”比较敏感，所以必须正确设置 Nuxt 的 app.baseURL，否则 CSS/JS 会 404。Nuxt 3 提供了 app.baseURL 用于设置站点前缀路径。 



------





## **2. 方案选择（推荐这套）**





推荐方案：**把简历站点当作“一个静态子应用”，构建后把产物拷贝进主站仓库的 /resume/ 目录**。



这样你主站仍然按原来的方式部署（可能是 Astro/纯静态/别的框架都行），只是多了一个静态目录 /resume/。



------





## **3. 前置条件**





1. 你已经有主站仓库（通常是）：







* 仓库名：leoxouyang.github.io
* GitHub Pages 发布源：可能是 main 分支的 root，或 gh-pages 分支，或 /docs 目录（以你当前设置为准）







1. 本地环境：







* Node.js（建议 18+）
* pnpm（项目文档用的是 pnpm） 





------





## **4. 必须修改的配置（关键）**







### **4.1 修改 Nuxt baseURL 为 /resume/**





在 markdown-resume 项目里找到 site/nuxt.config.ts（或等价配置文件），把：



* app.baseURL 设置为：'/resume/'





原因：Nuxt 生成的静态资源默认在 /_nuxt/ 下，发布到子路径时需要让它变成 /resume/_nuxt/...，否则浏览器会去根路径找资源直接 404。app.baseURL 就是专门做这个的。 



示例：

```
// site/nuxt.config.ts
export default defineNuxtConfig({
  app: {
    baseURL: '/resume/',
  },
})
```



### **4.2 建议同步更新站点 URL 元信息（可选但强烈建议）**





如果你的 nuxt.config.ts 里有类似 site.url、og:url 之类的元信息，建议改成真实地址 https://leoxouyang.github.io/resume/，避免分享预览/站点元信息指向错误。



------





## **5. 构建与产物目录**





项目 README 的构建步骤是：



* pnpm install
* pnpm build:pkg
* pnpm build 





Nuxt 3 静态生成的产物通常在：



* site/.output/public/





你最终要把这个目录的内容，完整拷贝到主站仓库的：



* <主站仓库根>/resume/





最终主站仓库结构会类似：

```
leoxouyang.github.io/
  index.html
  assets/...
  resume/
    index.html
    _nuxt/...
    ...
```



------





## **6. 部署方式 A：手动部署（最直观，适合先跑通）**







### **6.1 在 markdown-resume 仓库构建**



```
git clone https://github.com/leoxouyang/markdown-resume.git
cd markdown-resume
pnpm install
pnpm build:pkg
pnpm build
```

确认产物存在：

```
ls site/.output/public
```



### **6.2 拷贝产物到主站仓库的 /resume/**





假设你的主站仓库在同级目录：

```
# 在 markdown-resume 仓库根目录执行
rm -rf ../leoxouyang.github.io/resume
mkdir -p ../leoxouyang.github.io/resume
cp -R site/.output/public/* ../leoxouyang.github.io/resume/
```



### **6.3 提交并推送主站仓库**



```
cd ../leoxouyang.github.io
git add resume
git commit -m "Deploy resume site to /resume/"
git push
```

等待 GitHub Pages 发布完成后访问：



* https://leoxouyang.github.io/resume/





------





## **7. 部署方式 B：GitHub Actions 自动化（推荐长期使用）**





思路：在主站仓库里写一个 workflow，每次你更新简历仓库（或手动触发）就自动拉取 markdown-resume、构建，然后把产物写进主站仓库的 /resume/ 目录并推送。



注意：如果你的主站本身也用 Actions 构建并发布 pages，需要把两者合并到同一个发布流程里（最终输出一个包含主站 + resume 子目录的静态产物）。





### **7.1 简单版（主站是“直接发布 main 分支内容”）**





在 leoxouyang.github.io 仓库里添加 .github/workflows/update-resume.yml：

```
name: Update Resume

on:
  workflow_dispatch: {}

jobs:
  build-and-commit:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout main site repo
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Enable pnpm
        run: corepack enable

      - name: Clone resume repo
        run: |
          rm -rf /tmp/markdown-resume
          git clone https://github.com/leoxouyang/markdown-resume.git /tmp/markdown-resume

      - name: Build resume
        working-directory: /tmp/markdown-resume
        run: |
          pnpm install
          pnpm build:pkg
          pnpm build

      - name: Copy to /resume
        run: |
          rm -rf resume
          mkdir -p resume
          cp -R /tmp/markdown-resume/site/.output/public/* resume/

      - name: Commit changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add resume
          git commit -m "chore: update resume site" || echo "No changes"
          git push
```

你可以先用 workflow_dispatch 手动跑通，再考虑加定时或监听另一个仓库的触发（repo dispatch）。



------





## **8. GitHub Pages 常见坑与排错**







### **8.1 资源 404（最常见）**





症状：打开 /resume/ 页面只有 HTML，样式丢失、控制台显示 _nuxt/*.js 404。



原因：app.baseURL 没设对（必须是 /resume/），或构建时没生效。Nuxt 的 app.baseURL 就是用来解决子路径部署的。 





### **8.2 GitHub Pages/Jekyll 吃掉** 

### **_nuxt**

###  **目录**





GitHub Pages 默认会跑 Jekyll，它会忽略以 _ 开头的目录（比如 _nuxt），导致你部署上去资源“消失”。Nuxt 3 并不总是自动生成 .nojekyll，社区里这点被反复踩坑。 



解决：



* 在 resume/ 目录下放一个空文件 .nojekyll
* 或者在拷贝产物后追加：



```
touch resume/.nojekyll
```

（如果你用 Actions，就在 “Copy to /resume” 后加一步 touch。）





### **8.3 SPA 路由刷新 404**





如果你简历站点是纯静态多页面，通常没事；如果是 SPA 且依赖 history 路由，刷新子路由可能 404。常见解决是用 404.html 回退到 index.html（GitHub Pages 的老套路）。是否需要取决于你的项目路由模式。



------





## **9. 验收清单**





1. https://leoxouyang.github.io/resume/ 能打开
2. 控制台无 _nuxt 相关 404
3. 样式正常加载（UnoCSS/Vite 构建的 CSS 正常）
4. 字体选择（如果启用 Google Fonts Key）工作正常（需要 .env，项目 README 有说明） 
5. 刷新页面不会 404（如有子路由）





