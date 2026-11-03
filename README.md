# Trend-Gap Radar · 信息差雷达

简体中文 | [English](README.en.md)

对比同一批话题在海外（X、YouTube、Reddit）与国内（小红书、抖音、B站）的每日热度，识别跨区域的信息差：

- 海外 → 国内：海外已有热度、国内尚未跟进的选题
- 国内 → 海外：国内已验证、可迁移至海外的选题

程序输出 Markdown 格式日报，包含各话题在两侧的热度、信息差分数，以及两侧热度最高的 5 条内容与链接。日报可直接接入 [self-media-content-workflow](https://github.com/yanhua1010/self-media-content-workflow) 用于后续文案与视频制作。

> 状态：v0.1（基础框架）。当前通过配置文件中的中英关键词对匹配话题。待实现：基于大模型自动发现与聚类新话题；积累每日历史数据后计算国内滞后天数。

## 原理

1. 读取 `topics.toml` 中的关键词，分别调用 6 个平台的搜索接口。
2. 对单条内容的互动量按平台加权求和（例如小红书 = 点赞 + 收藏 + 2×分享 + 评论），取 `log10`（差 1 分约对应 10 倍互动量差距）。
3. 筛选最近 `window_days` 天内发布的内容，取前 5 条的均值作为该侧热度。
4. 信息差 = 海外热度 − 国内热度；≥ 1 记为「海外热、国内冷」，≤ −1 记为「国内热、海外冷」。

平台数据口径差异：抖音搜索接口不返回播放量，仅计互动数；B站搜索接口仅返回播放量；YouTube 仅返回「5d ago」格式的相对时间。

## 数据来源与成本

数据通过 [AIsa](https://aisa.one) 的 REST API 获取，单个 key 覆盖 6 个平台（国内平台底层数据由 TikHub 提供）。单次调用实测价格：

| 平台 | 接口 | 每次 |
|---|---|---|
| 小红书 | 笔记搜索 | $0.01 |
| 抖音 | 视频搜索 | $0.01 |
| B站 | 综合搜索 | $0.001 |
| X | 高级搜索 | $0.003 |
| YouTube | 搜索 | $0.0029 |
| Reddit | 搜索 | $0.0019 |

单个话题每天约 $0.03，追踪 10 个话题约 $0.3/天。单次运行设有费用硬上限（默认 $0.5），达到即停止请求。

## 快速开始

环境要求：Python 3.11+，无第三方依赖。

1. 在 AIsa 注册并创建 API key：<!-- TODO: 换成 Dub 推荐链接，并保留下面的披露 -->https://aisa.one
   > 披露：此为推荐链接，通过该链接注册并付费，作者会获得佣金，你的价格不变。
2. 设置环境变量：

   ```bash
   export AISA_API_KEY=sk-aisa-...
   ```

3. 编辑 `topics.toml`，先估算费用（免费），再执行抓取：

   ```bash
   python3 -m radar quote
   python3 -m radar run
   ```

日报输出至 `reports/YYYY-MM-DD.md`，原始响应保存至 `data/raw/YYYY-MM-DD/`。使用已保存的原始数据重新生成报告不会产生 API 费用：

```bash
python3 -m radar replay data/raw/2026-11-03
```

常用参数：`--budget 0.2` 指定单次费用上限；`--platforms x xiaohongshu` 仅运行指定平台。

## 定时运行

每天早上 8 点执行一次（macOS / Linux crontab）：

```cron
0 8 * * * cd /path/to/trend-gap-radar && AISA_API_KEY=sk-aisa-... python3 -m radar run
```

## 边界

- 仅读取公开数据，不登录任何平台账号
- 日报仅提供选题方向与来源链接，不复制原内容
- API key 仅从环境变量读取，不写入代码仓库

## 开发

```bash
python3 -m unittest
```

## License

MIT
