# WorldMonitor 部署指南 (Vercel + Upstash)

## 📋 前置准备

### 1. 注册账户
- **Vercel**: https://vercel.com (免费)
- **Upstash Redis**: https://upstash.com (免费 10K commands/day)
- **可选 API Keys**: 见下方列表

### 2. Fork 仓库
已完成: https://github.com/edwardqiu1976/worldmonitor

---

## 🚀 部署步骤

### Step 1: 推送 CORS 修改

```bash
cd ~/worldmonitor-deploy
git add api/_cors.js
git commit -m "feat: update CORS for self-hosted deployment"
git push origin main
```

### Step 2: 创建 Upstash Redis

1. 登录 https://upstash.com
2. 创建 Redis 数据库:
   - Name: `worldmonitor`
   - Region: 选择最近的 (如 `ap-northeast-1` 东京)
3. 复制连接信息:
   - `UPSTASH_REDIS_REST_URL`
   - `UPSTASH_REDIS_REST_TOKEN`

### Step 3: 在 Vercel 创建项目

1. 登录 https://vercel.com
2. 点击 "Add New..." → "Project"
3. Import Git Repository: 选择 `edwardqiu1976/worldmonitor`
4. 配置:
   - Framework Preset: **Vite** (自动检测)
   - Root Directory: `./`
   - Build Command: `npm run build` (默认)
   - Output Directory: `dist` (默认)

### Step 4: 配置环境变量

在 Vercel 项目 Settings → Environment Variables 中添加:

#### 必需
```
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-token
```

#### 可选 (免费)
```
# AI 服务 (用于智能摘要)
GROQ_API_KEY=xxx          # https://console.groq.com (免费 14.4K req/天)
OPENROUTER_API_KEY=xxx    # https://openrouter.ai (免费 50 req/天)

# 市场数据
FINNHUB_API_KEY=xxx       # https://finnhub.io
FRED_API_KEY=xxx          # https://fred.stlouisfed.org
EIA_API_KEY=xxx           # https://www.eia.gov/opendata/

# 冲突数据
ACLED_ACCESS_TOKEN=xxx    # https://acleddata.com (研究免费)

# 卫星图像
NASA_FIRMS_API_KEY=xxx    # https://firms.modaps.eosdis.nasa.gov
```

### Step 5: 部署

点击 "Deploy" 按钮，等待构建完成 (~3-5 分钟)

---

## ✅ 验证部署

部署成功后访问:
- 主页: `https://worldmonitor-xxx.vercel.app`
- Health Check: `https://worldmonitor-xxx.vercel.app/api/health`

预期 Health 输出:
```json
{"status": "ok", "checks": {...}}
```

---

## 🔄 后续配置

### 自定义域名 (可选)

1. Vercel 项目 Settings → Domains
2. 添加你的域名 (如 `monitor.yourdomain.com`)
3. 更新 `api/_cors.js` 添加域名 pattern:
   ```js
   /^https:\/\/monitor\.yourdomain\.com$/,
   ```

### Seed Scripts (可选)

如果需要预填充数据，可以手动运行 seed scripts:
```bash
# 需要在本地执行，连接到 Upstash Redis
export UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
export UPSTASH_REDIS_REST_TOKEN=xxx
node scripts/seed-earthquakes.mjs
node scripts/seed-military-flights.mjs
# ... 其他 seed scripts
```

---

## 🔧 已修改的文件

| 文件 | 修改内容 |
|------|----------|
| `api/_cors.js` | 添加 Vercel preview 域名 pattern |

---

## 📊 数据源状态

| 状态 | 数据源 |
|------|--------|
| 🟢 无需 key | 地震、天气、难民、加密货币、网络威胁、气候异常 |
| 🟢 免费注册 | GROQ、FRED、EIA、NASA FIRMS、Finnhub、ACLED |
| 🔴 付费 | Cloudflare Radar (网络中断监控) |