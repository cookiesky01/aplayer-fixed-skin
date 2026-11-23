# aplayer-fixed-skin

APlayer 左下角固定播放器皮肤：暂停时收成方形封面，悬停展开成玻璃卡片，适配暗色模式与手机窄屏。只改 CSS，不动 APlayer 源码。

![效果](cover.svg)

## 目录

```
aplayer-fixed-skin/
├── aplayer-fixed.css   # 皮肤样式
├── nav-music.js        # 拉歌单 + 创建播放器（fixed + mini）
├── demo.html           # 预览页（APlayer 走 CDN）
├── cover.svg           # 示例封面
└── README.md
```

依赖 [APlayer 1.10.x](https://github.com/DIYgod/APlayer)；歌单走 [Meting](https://github.com/metowolf/Meting) 接口，音频用网易云外链。

## 本地预览

`nav-music.js` 用 `fetch` 拉歌单，必须走 HTTP：

```bash
npx serve .
# 或
python -m http.server 8080
```

打开 `http://localhost:8080/demo.html`，暂停状态左下角是 66px 封面，悬停展开 402px，右上角可以切暗色。

## Hexo + Butterfly 接入

1. 把 `APlayer.min.js` 放到 `source/js/`，`APlayer.min.css` 放到 `source/css/`（本站资源本地化；也可以只用 CDN，把 inject 里的地址换掉）。
2. 把 `aplayer-fixed.css`、`nav-music.js` 分别放到 `source/css/`、`source/js/`。
3. `_config.butterfly.yml` 的 `inject` 加上：

```yml
inject:
  head:
    - <link rel="stylesheet" href="/css/APlayer.min.css">
    - <link rel="stylesheet" href="/css/aplayer-fixed.css">
  bottom:
    - <div id="nav-music" class="no-destroy"></div>
    - <script defer src="/js/APlayer.min.js"></script>
    - <script defer src="/js/nav-music.js"></script>
```

4. 改完升一次 `?v=` 版本号；开了 Service Worker 的还要升 `sw.js` 里的 `VERSION`。

## 换歌单与封面

`nav-music.js` 顶部：

```js
var PLAYLIST_ID = "2331639478";            // 网易云歌单 ID
var APIS = ["https://api.i-meto.com/meting/api"]; // Meting 接口
var FALLBACK_COVER = "/img/HG.JPG";         // 封面缺失时的兜底图
```

- 歌单结果缓存在 `localStorage`，6 小时后刷新；接口挂了会先用缓存。
- 音频默认用 `https://music.163.com/song/media/outer/url?id=歌曲ID.mp3`，失败时回退接口直链。
- 想换 QQ 音乐等平台，更换 Meting 接口与 `server` 参数即可（见下表）。

## `aplayerInject` 写法的参数

如果你用的是 hexo-tag-aplayer 的 `aplayerInject` 标记写法：

```html
<div id="aplayer-oSEOhviA" class="aplayer aplayer-tag-marker meting-tag-marker"
     data-id="4895239160" data-server="netease" data-type="playlist"
     data-mode="random" data-autoplay="false" data-listmaxheight="340px"
     data-preload="auto" data-theme="#e3f2f5" data-volume="0.4" mutex="true"></div>
```

| 参数 | 说明 | 常用值 |
| --- | --- | --- |
| `id` | 容器 ID，随便起，不要重复 | `aplayer-oSEOhviA` |
| `class` | 固定类名，插件靠它识别标记 | `aplayer aplayer-tag-marker meting-tag-marker` |
| `data-id` | 歌单 / 歌曲 ID | `4895239160` |
| `data-server` | 平台：`netease` 网易云、`tencent` QQ 音乐、`kugou` 酷狗、`baidu` 百度 | `netease` |
| `data-type` | 资源类型：`playlist` 歌单、`song` 单曲、`album` 专辑、`artist` 歌手 | `playlist` |
| `data-mode` | 播放模式：`random` 随机、`order` 顺序、`single` 单曲循环、`circulation` 列表循环 | `random` |
| `data-autoplay` | 是否自动播放（浏览器通常拦截） | `false` |
| `data-listmaxheight` | 列表最大高度 | `340px` |
| `data-preload` | 预加载：`none` / `metadata` / `auto` | `auto` |
| `data-theme` | 进度条、高亮色 | `#e3f2f5` |
| `data-volume` | 初始音量 `0–1` | `0.4` |
| `mutex` | 多播放器互斥，同时只响一个 | `true` |

关于 QQ 音乐：`data-server="tencent"` 是支持的，但**能否拉到数据取决于所用的 Meting 接口**。公共接口里 `api.i-meto.com` 对 `tencent` 常返回空数组，需要换支持 QQ 的实例或自建 Meting API，ID 也要用 QQ 音乐的歌曲/歌单 ID。

## 皮肤可调参数

| 项目 | 值 | 位置 |
| --- | --- | --- |
| 距左下角 | `18px`（手机 `12px`） | `.aplayer.aplayer-fixed` / 媒体查询 |
| 收起宽度 | `66px` | `.aplayer-narrow .aplayer-body` |
| 展开宽度 | `402px`（手机 `calc(100vw - 24px)`） | `:not(.aplayer-narrow)` / 媒体查询 |
| 圆角 | `18px`（封面 `15px`） | `.aplayer-body` / `.aplayer-pic` |
| 玻璃背景 | `rgba(255,255,255,.92)` + `blur(16px) saturate(160%)` | `.aplayer-body` |
| 主题渐变 | `#49b1f5 → #39c5bb` | 播放按钮、进度条、列表高亮 |
| 暗色背景 | `rgba(20,24,32,.94)` | `[data-theme="dark"]` 段 |

换色时全局替换 `73, 177, 245`（主色）与 `57, 197, 187`（辅色）。

## 已知限制

- 只针对 `fixed` 模式；页面内 `list` 模式的播放器不受影响。
- 改展开宽度要同步改三处：`.aplayer-narrow`、`:not(.aplayer-narrow)`、`@media (max-width: 768px)`。
- 基于 APlayer 1.10.x 的类名（`.aplayer-body`、`.aplayer-pic`、`.aplayer-list` 等），大版本升级前先对照结构。
- 网易云外链有版权/地区限制，个别歌曲会 302 到空内容。

## License

MIT。播放器内核 [APlayer](https://github.com/DIYgod/APlayer) 与 [Meting](https://github.com/metowolf/Meting) 遵循各自许可。
