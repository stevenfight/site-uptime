# site-uptime

`www.suweizhao.com` 的可用性检查。每 15 分钟探一次，失败自动开 issue，恢复后自动关闭。

**这个仓库不存放站点代码**，站点源码在本地 `个人主页/` 目录，通过 `deploy/deploy.ps1` 发布到服务器。这里只有一份 GitHub Actions 配置。

## 它做什么

每 15 分钟跑一次，逐个请求下面 7 个地址，每个失败重试 3 次（间隔 8 秒），避免网络抖动误报：

```
/  /case-rag.html  /case-agent.html  /case-lora.html
/case-quiz.html  /case-booking.html  /sitemap.xml
```

- **全部 200**：什么都不做，结束。
- **任一不是 200**：开一个标题为 `[uptime] 站点不可用` 的 issue，正文列出哪个地址返回了什么码。GitHub 会把 issue 发到账号邮箱。
- **还在故障中**：不会重复开 issue，只在原 issue 下追加评论。
- **恢复了**：自动评论并关闭 issue。

## 为什么有个 `heartbeat` 提交

GitHub 有一条规则：**公共仓库的定时任务，在仓库连续 60 天没有提交活动后会被自动停用**，而且不会通知你。这个仓库除了监控脚本没有别的改动，很容易两个月后悄悄失效。

所以 workflow 最后一步会检查 `.heartbeat` 文件里的时间戳，超过 30 天就更新一次并提交。这样仓库始终有活动，定时任务不会被停。正常情况大约每月一次提交，不是每次都提交。

## 手动跑一次

仓库页面 → Actions → 左侧 `site-uptime` → 右侧 `Run workflow`。

也可以在本地验证判断逻辑：

```bash
for p in / /case-booking.html; do
  curl -s -o /dev/null -w "%{http_code}  $p\n" "https://www.suweizhao.com$p"
done
```

## 改监控范围

编辑 `.github/workflows/uptime.yml` 里 `for path in ...` 那一行。加了新页面就补一个路径。

## 它查不出来的东西

- 页面内容错了但返回 200（比如样式丢了）—— 这个只能靠人看。
- 某个地区访问不了但 GitHub 的机器能访问 —— 单一探测点，代表不了全国。
- 慢但没挂（比如 8 秒才响应）—— 现在只判状态码，不设耗时阈值。
