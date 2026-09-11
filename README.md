# 毛 × 哈利 · 备婚统筹台

一个纯静态的备婚统筹页：进度、清单、预算、婚宴流程都在里面，零依赖、零构建，
浏览器打开就能用，两个人还能通过 Supabase 房间同步同一份数据。

## 仓库里有什么

| 文件 | 作用 |
| --- | --- |
| `index.html` | 页面骨架，约 40 行标记 |
| `style.css` | 全部样式，含深色 / 浅色两套主题变量 |
| `app.js` | 全部逻辑：本地存储、双人同步、导出导入、.ics、列印 |
| `.github/workflows/deploy-pages.yml` | 推送后自动发布到 GitHub Pages |
| `.nojekyll` | 让 Pages 原样发布文件，不走 Jekyll 处理 |
| `robots.txt` | 谢绝搜索引擎收录（页面本身也带了 `noindex`） |

原本这三份内容挤在一个 `index.html` 里，现在拆开了，改样式改逻辑互不干扰。
拆分是纯搬运：把 `style.css` 和 `app.js` 原样塞回 `<style>` / `<script>` 标签，
和拆分前的文件逐字节一致。

## 本地预览

```bash
python3 -m http.server 8080
# 然后打开 http://localhost:8080
```

请务必用 `http://` 打开，不要直接双击 `index.html`：
拆成三个文件之后，`file://` 下浏览器可能拦掉 `style.css` 和 `app.js`。

## 上线（GitHub Pages）

工作流已经配好，第一次上线只要点一步：

**Settings → Pages → Build and deployment → Source** 选 **GitHub Actions**。

之后每次推送到默认分支 `claude/inspiring-pascal-da87o8` 都会自动重新部署，
站点地址是 <https://sqsqsqlll.github.io/marriageplan/>；
也可以在 **Actions → Deploy to GitHub Pages → Run workflow** 手动触发。

> 注意：`github-pages` 环境默认只允许**默认分支**部署，所以工作流盯的是默认分支。
> 以后如果把默认分支改名（例如改成 `main`），记得同步改
> `.github/workflows/deploy-pages.yml` 里 `on.push.branches` 的分支名。

部署前工作流会先体检：三个文件都在且非空、`index.html` 没被截断且确实引用了
另外两个文件、`app.js` 语法能过 —— 任何一项不过就中止部署，不会把坏页面推上线。

## 数据存在哪里

- **本机**：`localStorage`，换设备或清缓存就没了。页面上有「匯出備份」，建议定期导出。
  导出的 `.json` 已经在 `.gitignore` 里，不会误提交进仓库。
- **云端**：Supabase 项目 `qiwyvztluteqrnilajve`，两人共用一个「房间」，约每十秒对一次。
  页面调用两个 RPC，上线前请确认它们在 Supabase 里存在且可用：
  - `plan_save(p_room, p_secret, p_data, p_by)`
  - `plan_load(p_room, p_secret)`

### 关于那把 key

`app.js` 里写着 Supabase 的 **anon（公开）key**，这类 key 本来就是给浏览器用的、
公开可见没问题 —— 真正拦人的是数据库的 RLS 策略和房间密钥。上线前请确认：

- 相关表已开启 RLS，anon 角色不能直接读写表；
- 房间数据只能通过上面两个 RPC、且校验 `p_secret` 之后才能读写；
- 房间密钥别用生日之类好猜的字符串。

## 一点提醒

这个仓库目前是 **public**，任何拿到链接的人都能打开页面。具体内容不在仓库里
（在各自浏览器和 Supabase 里），但如果希望连页面都不外传，把仓库改成 private 即可 ——
注意 private 仓库的 GitHub Pages 需要付费方案才能发布。

## 回滚

```bash
git revert <commit>
git push origin claude/inspiring-pascal-da87o8
```

Pages 会按新的默认分支内容重新部署。
