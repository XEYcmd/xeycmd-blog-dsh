# 番剧推荐 & 音乐播放器

## 番剧推荐页

地址：`/anime/`（英文 `/en/anime/`），导航栏「番剧」进入。

### 数据来源

199 部作品来自作品库 JSON，生成到 `src/data/anime.ts`，分三步：

```bash
node scripts/build-anime.mjs    # 从作品库 JSON 生成基础列表（名字 + 标签）
node scripts/apply-baike.mjs    # 写入百度百科精确词条路径（baikePath）
node scripts/apply-fixes27.mjs  # 人工修正 27 条 + 按百科官方名改名
```

**不要直接改 `src/data/anime.ts`**，它会被重新生成覆盖。

作品库源文件按这个顺序查找：

1. 环境变量 `ANIME_SOURCE` 指定的路径
2. `content/anime-source.json`（默认，已被 `.gitignore` 忽略，不入库）

找不到会直接报错退出，不会生成空数据。

### 修改作品列表

改作品库 JSON（`content/anime-source.json`）后重新跑上面三步。想临时换源：

```bash
ANIME_SOURCE=/path/to/works.json node scripts/build-anime.mjs
```

### 百科链接

每部作品只有一个链接：**百科** → 百度百科。

优先用 `baikePath` 精确路径（形如 `/item/冰菓/5650507`，带数字 ID，避免落到消歧义页）；
没有 `baikePath` 的条目回退到站内搜索页。

想手工指定某部的条目，改 `scripts/apply-fixes27.mjs` 里的 `FIXES` 对象（按序号）
或 `RENAMES` 对象（按原名改官方名），然后重跑后两步。

名字对不上的（如 `love live` → `LoveLive!`），在 `build-anime.mjs` 的 `ALIAS` 里加一条。

### 页面功能

- 搜索框（输入即时过滤，按 `/` 快速聚焦）
- 标签筛选（34 种标签，点选切换）
- 搜索 + 标签可组合
- 无结果时显示提示
- 手机版：标签横向滚动、条目单列

---

## 音乐播放器

全局悬浮按钮，右下角，所有页面都有。

### 加歌

1. 把音频文件放进 `public/music/`
2. 编辑 `src/data/music.ts`：

```ts
export const tracks: Track[] = [
  { title: '曲名', artist: '歌手', src: '/music/xxx.mp3' },
  { title: '另一首', artist: '歌手', src: '/music/yyy.mp3', cover: '/music/cover.jpg' },
];
```

3. `npm run build`

**歌单为空时播放器按钮仍然显示**（右下角），点开会提示如何添加歌曲。这样功能可见、可发现。

### 支持的字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `title` | ✅ | 曲名 |
| `src` | ✅ | 站内路径 `/music/x.mp3` 或完整外链 |
| `artist` | ❌ | 歌手 |
| `cover` | ❌ | 封面图，站内路径或外链 |

格式：mp3 / ogg / m4a / wav 等浏览器能播的都行。

### 功能

- 播放 / 暂停 / 上一首 / 下一首
- 拖动进度条
- 音量调节 + 静音
- 三种播放模式：列表循环 → 单曲循环 → 随机
- 媒体键支持（耳机线控、键盘媒体键）
- 记住上次的曲目、进度、音量和模式（localStorage）
- 面板打开时按空格可播放/暂停

### 配置项

`src/data/music.ts` 里还有两个开关：

```ts
export const DEFAULT_VOLUME = 0.7;   // 默认音量
export const REMEMBER_STATE = true;  // 是否记忆播放状态
```

### 关于版权

播放器本身不提供任何音频，只播放你放进 `public/music/` 的文件。
请确保你有权分享这些音频——公开站点上传播受版权保护的音乐有法律风险。
