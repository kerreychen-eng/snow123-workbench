# Snow123 投前助手 - 部署与使用说明

## 你拿到的东西

1. **`Snow123_内部工作台.html`** - 工作台主文件(已加扩展导入功能)
2. **`snow123-extension/`** - Chrome 扩展整个文件夹

## 一、部署工作台到 GitHub Pages

### Step 1:创建 GitHub 仓库

1. 登录 https://github.com
2. 右上角 **+** → **New repository**
3. 仓库名建议 `snow123-workbench`
4. **必须勾选 Public**(Private 用不了免费 Pages)
5. 不要勾任何 README/.gitignore
6. 点 **Create repository**

### Step 2:上传工作台 HTML

最简单的方式:

1. 仓库页面点 **uploading an existing file** 链接
2. 把 `Snow123_内部工作台.html` 拖进去
3. **关键**:把文件名改成 `index.html`(否则访问不方便)
   - 直接在 Github 网页里上传后,无法重命名 → 改成本地先把 `Snow123_内部工作台.html` 复制一份命名为 `index.html` 再上传
4. 点 **Commit changes**

### Step 3:开启 Pages

1. 仓库 → **Settings**(右上角)
2. 左侧 → **Pages**
3. Source:选 **Deploy from a branch**
4. Branch:选 **main**(或 master)+ **/(root)**
5. 点 **Save**
6. 等 1-2 分钟,刷新页面会看到 **"Your site is live at https://你的用户名.github.io/snow123-workbench/"**

### Step 4:打开测试

访问那个 URL,工作台应该正常加载。**记住这个URL**,等下扩展要用。

> 如果你不希望工作台公开可访问:仓库改 Private,但要订阅 GitHub Pro($4/月) 才能用 Pages on private repo。或者保留 Public 但URL不外传(URL本身就是不公开的,搜不到)。

## 二、安装 Chrome 扩展

### Step 1:下载扩展文件夹

把 `snow123-extension/` 整个文件夹下载到电脑上(任意位置,例如桌面)。

### Step 2:在 Chrome 加载

1. Chrome 打开 `chrome://extensions/`
2. 右上角打开 **开发者模式**
3. 点 **加载已解压的扩展程序**
4. 选择 `snow123-extension/` 文件夹
5. 扩展出现在列表里,**钉到工具栏**(扩展图标 → 图钉按钮)

### Step 3:配置工作台 URL

1. 点工具栏的 **S** 图标
2. 最下方 **工作台URL** 输入框,填入你的 GitHub Pages URL
   - 例如:`https://你的用户名.github.io/snow123-workbench/`
3. URL 会自动保存

## 三、使用流程

### 一次完整投放调研(约 1 分钟):

```
1. 打开亚马逊产品页(等评分/评论加载完)
   → 点扩展 S 图标
   → 点【抓取当前亚马逊页】
   → 勾选用哪个搜索关键词
   ✓ 看到 ASIN/原价/评分/评论/类目 已抓到

2. 在扩展里点【① 打开 dealdeal.net 搜索】
   → 新标签打开 dealdeal 搜索页(关键词已带)
   → 等结果出来
   → 回扩展点【② 抓取当前 dealdeal 页价格】
   ✓ 看到价格区间已抓到

3. 在扩展里点【① 打开 Facebook 搜索】
   → 新标签打开 FB 搜索页
   → 滚动几下让结果加载
   → 回扩展点【② 抓取当前 FB 页价格】
   ✓ 看到价格区间已抓到

4. 点【🚀 打开工作台并导入】
   → 工作台打开,自动跳到投前建议页
   → 弹窗提示"已导入:链接/ASIN/原价/..."
   → 填一下折扣价(这个必须自己填) + 选目标 + 点生成
```

## 四、常见问题

**Q: 抓取亚马逊页失败?**
- 确认页面是 `amazon.com/dp/B0XXX` 格式的产品详情页
- 等页面完全加载(评分星显示出来)再抓
- 评分/评论选择器在亚马逊改版后可能失效,告诉我可以修

**Q: dealdeal 抓到 0 条价格?**
- 确认在 `dealdeal.net/deal/search.php` 这个页面
- 搜索"3 weeks"过滤再抓,会有更多数据

**Q: FB 抓不到价格?**
- FB搜索结果懒加载,**滚动几下**让更多帖子出来再抓
- FB必须登录才能看搜索结果

**Q: 折扣价为什么不自动抓?**
- 你的折扣价是策划的(报价单上的目标价),不是亚马逊当前页价格 - 必须人工填

**Q: 扩展自动加载新版本怎么做?**
- 直接用新版替换 snow123-extension 文件夹,Chrome 扩展页点扩展上的"刷新"图标
- 或者用 chrome.runtime.reload() 强制重载

**Q: 工作台改了怎么更新到 Pages?**
- 上传新的 index.html 替换旧的(GitHub 仓库 → 点 index.html → 编辑铅笔图标 → 删除内容粘贴新的)
- 或者用 GitHub Desktop 客户端推送(更专业)

## 五、扩展文件清单

```
snow123-extension/
├── manifest.json       # 扩展配置
├── popup.html         # 点图标弹出的窗口
├── popup.js           # 弹窗逻辑
├── background.js      # 后台(目前空)
├── scrape_amazon.js   # 亚马逊页抓取脚本
├── scrape_deal.js     # dealdeal 抓取脚本
├── scrape_fb.js       # FB 抓取脚本
└── icons/             # 图标
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

## 六、版本说明

**v0.1.0 (本版本)** - 初版
- 亚马逊页抓取(ASIN/原价/评分/评论/类目/FBA-FBM/关键词候选)
- dealdeal.net 价格抓取
- FB 搜索结果价格抓取(自动剔除疑似原价)
- 通过URL hash把数据传给工作台,无服务器

**接下来可能的迭代**:
- 扩展弹窗里直接编辑数据
- 自动识别折扣价(目前必须手填)
- 支持 SellerSprite / 卖家精灵的产品数据接口
- 团队多人同步(目前每人各自抓)
