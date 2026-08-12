---
name: "wallpaper-crawler"
description: "从 livewallpaper.giantapp.cn 动态壁纸网站爬取壁纸数据,生成符合规范的 bg.json 文件。当用户提供一个壁纸网站的列表页 URL 或详情页 URL 时调用此技能。"
---

# 壁纸爬虫 (wallpaper-crawler)

## 用途

从 `livewallpaper.giantapp.cn` 网站批量爬取动态壁纸的缩略图和视频直链,生成符合 Fndesk 桌面美化工具规范的 `bg.json` 文件。

## 数据格式规范

输出文件 `bg.json` 必须位于 `deskdata/bg.json`,格式如下:

```json
[
  {
    "ID": 1,
    "p0": "https://.../covers/0.jpg",
    "p1": "https://.../video.mp4"
  }
]
```

- `ID`: 数字序号,从 1 开始递增
- `p0`: 缩略图 URL (来自详情页 `coverSource` 字段)
- `p1`: 视频直链 URL (来自详情页 `wpSource` 字段)
- `p0` 可选,缺失时省略;`p1` 必填,无 `p1` 则跳过该条

## 爬取流程

### 第一步:获取壁纸 ID 列表

用户会提供一个列表页 URL,形如:
`https://livewallpaper.giantapp.cn/wallpapers/1/all/video/download/week`

URL 格式: `https://livewallpaper.giantapp.cn/wallpapers/{页码}/all/video/download/week`

**获取方式:**
1. 用浏览器打开该列表页
2. 右键 → 查看页面源代码 (View Page Source)
3. 在源码中搜索 `window.__NUXT__`,找到其中包含 ID 的部分
4. 提取每个壁纸的 ID (通常是类似 `6a737137b52e7d34e813e173` 这样的字符串)

ID 在 `__NUXT__` 中的位置可能如下:
- 在第一页 HTML 源码中,只展示部分 ID
- 其余 ID 在 JavaScript 注入的数据中 (需滚动或翻页获取)

**注意**: 如果源码中能找到的 ID 不足,需要用户继续提供滚动/翻页后新增的 ID,或换到下一页的 URL。

### 第二步:逐个请求详情页,提取视频和缩略图

对每个 ID,请求详情页:
`https://livewallpaper.giantapp.cn/wallpapers/info/{ID}`

在详情页的 `window.__NUXT__` JS 对象中,需要提取两个字段:
- `wpSource` — 视频直链 URL (必须,缺失则跳过)
- `coverSource` — 缩略图 URL (可选,缺失则省略)

**重要:数据格式解析**

`__NUXT__` 对象使用的是 JS 字面量格式 (key 无引号):
```javascript
// 不是 "wpSource": "...", 而是:
wpSource:"https:\u002Fu002F..."
coverSource:"https:\u002F..."
```

URL 中的 `/` 被编码为 `\u002F`,解析时:
1. 先找到 `wpSource:"` 或 `wpSource`:`"` (两种格式都要匹配)
2. 逐字符读取,遇到 `"` 停止
3. 保留原始字符串中的反斜杠
4. 最后用 `[System.Text.RegularExpressions.Regex]::Unescape()` 统一解码 `\u002F` 等 unicode 转义

### 第三步:生成 bg.json

按 ID 顺序生成 JSON 数组,写入 `deskdata/bg.json`。

## PowerShell 爬虫脚本模板

以下是完整的爬虫脚本,根据用户需求填写 ID 列表和输出路径后直接运行:

```powershell
# ============ 在此填写壁纸 ID 列表 ============
$ids = @(
  '6a737137b52e7d34e813e173',
  '6a73708ab52e7d34e813e165',
  # ... 更多 ID
)
# ============================================

# 解析 __NUXT__ 中 JS 字面量格式的 URL 值
function Parse-Url($html, $key) {
  $quoted = '"' + $key + '":'
  $i = $html.IndexOf($quoted)
  if ($i -ge 0) {
    $tail = $html.Substring($i + $quoted.Length)
    $q = $tail.IndexOf('"')
    if ($q -lt 0) { return '' }
    $tail = $tail.Substring($q + 1)
  } else {
    $plain = $key + ':"'
    $i = $html.IndexOf($plain)
    if ($i -lt 0) { return '' }
    $tail = $html.Substring($i + $plain.Length)
  }
  $sb = [System.Text.StringBuilder]::new()
  for ($j = 0; $j -lt $tail.Length; $j++) {
    $ch = $tail[$j]
    if ($ch -eq '"') { break }
    [void]$sb.Append($ch)
  }
  $raw = $sb.ToString()
  $result = [System.Text.RegularExpressions.Regex]::Unescape($raw)
  return $result
}

# 逐 ID 爬取
$wallpapers = [System.Collections.Generic.List[string]]::new()
$cid = 1

foreach ($wpId in $ids) {
  try {
    $url = 'https://livewallpaper.giantapp.cn/wallpapers/info/' + $wpId
    $html = Invoke-WebRequest -Uri $url -UseBasicParsing | Select-Object -ExpandProperty Content
    $videoUrl = Parse-Url $html 'wpSource'
    $thumbUrl = Parse-Url $html 'coverSource'

    if ($videoUrl -ne '') {
      $lines = [System.Collections.Generic.List[string]]::new()
      $lines.Add('  {')
      $lines.Add('    "ID": ' + $cid + ',')
      if ($thumbUrl -ne '') {
        $lines.Add('    "p0": "' + $thumbUrl + '",')
      }
      $lines.Add('    "p1": "' + $videoUrl + '"')
      $lines.Add('  }')
      $wallpapers.Add($lines -join "`n")
      Write-Host ('OK #' + $cid)
      $cid++
    } else {
      Write-Host ('SKIP ' + $wpId)
    }
  } catch {
    Write-Host ('ERR ' + $wpId)
  }
}

if ($wallpapers.Count -eq 0) {
  Write-Host 'WARN: no wallpapers collected'
}

$sep = ",`n"
$joined = $wallpapers -join $sep
$nl = "`n"
$json = '[' + $nl + $joined + $nl + ']'
$pwd = (Get-Location).Path
$outdir = $pwd + '\deskdata'
if (-not (Test-Path $outdir)) { New-Item -ItemType Directory -Path $outdir -Force | Out-Null }
$outpath = $outdir + '\bg.json'
$json | Out-File -FilePath $outpath -Encoding utf8
Write-Host ('Done. Total: ' + $wallpapers.Count + '  -> ' + $outpath)
```

## 注意事项

### PowerShell 脚本编写规范 (Windows 环境)

1. **禁止使用 `-f` 格式化操作符**: 当 URL 中 `{` `}` 等字符会导致格式化错误。所有字符串拼接必须用 `+`。
2. **禁止在双引号字符串中嵌套 `$(...)` 子表达式**: 会被解析器误读。应将子表达式先赋值给变量,再用 `+` 拼接。
3. **避免硬编码中文路径**: 使用 `(Get-Location).Path` + `Join-Path` 或 `+` 拼接构建路径。
4. **`Invoke-WebRequest` 必须加 `-UseBasicParsing`**: 避免依赖 IE 引擎。

### 网络/超时问题

- 个别 ID 可能因网站数据已删除而无 `wpSource`,会显示 `SKIP`
- 如果请求失败会显示 `ERR`,通常重试即可
- 网站返回 `contentLength` 为 0 说明该壁纸的视频已被删除

### URL 编码

- `__NUXT__` 中的 URL 使用 JS 字面量转义,`/` 写作 `\u002F`
- 解析时**不要**在逐字符循环中消耗反斜杠字符
- 使用 `[System.Text.RegularExpressions.Regex]::Unescape()` 做最终解码

## 输出示例

```json
[
  {
    "ID": 1,
    "p0": "https://pvzh.giantapp.cn/upload/xxx/wallpapers/abc/covers/0.jpg",
    "p1": "https://pvzh.giantapp.cn/upload/xxx/wallpapers/abc/1234567890_Filename.mp4"
  },
  {
    "ID": 2,
    "p0": "https://pvzh.giantapp.cn/upload/xxx/wallpapers/def/covers/0.jpg",
    "p1": "https://pvzh.giantapp.cn/upload/xxx/wallpapers/def/1234567891_Video.mp4"
  }
]
```

## 使用场景

- 用户在 Fndesk 桌面美化工具项目中需要批量导入动态壁纸
- 用户提供一个壁纸网站列表页 URL,要求生成对应的 `bg.json`
- 用户需要补充/更新壁纸库数据