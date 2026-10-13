# 官方数据源：SDE 与 ESI

写 EVE 中文攻略时，**物品名、模块名、机制术语的官方译名只能从官方数据源取得**。社区译名（繁中、第三方 Wiki、旧版本、玩家俗称）互相冲突，照抄会写出客户端里搜不到的名字。

EVE 官方提供两个数据源，正好互补：

| 数据源 | 形态 | 内容 | 适用 |
|--------|------|------|------|
| **SDE**（Static Data Export）| 离线压缩包 | 全量静态数据，含 name/description **多语言字典** | 批量核对、离线查名、看官方描述 |
| **ESI**（EVE Swagger Interface）| 在线 API | 单条查询，支持 `language` 参数 | 快速查几个名字 |

> ⚠️ **关键限制（先看这条，避免白忙）**：两者都**只覆盖"物品/类型"的名称与描述**。
> **UI 字符串**（战斗日志的命中质量描述、总览列名、按钮文字等）**不在 SDE 里**，也不在 ESI 里——那些在客户端本地化文件中，无法通过公开数据源核对。遇到这类术语，只能在文档中保留英文并标注"中文以游戏内为准"。

---

## 一、ESI：快速查官方译名

基址 `https://esi.evetech.net/latest/`，无需认证。

支持的语言（由 SDE `translationLanguages.yaml` 给出）：
`en` / `fr` / `de` / `ja` / `ru` / `zh` / `ko` / `es`

> `zh` 即**简体中文**。传非法值会返回 400 `'zz' is not valid enum value`。

### 步骤 1：名称 → typeID

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  -d '["Spike L","Javelin L","Void L","Null L"]' \
  "https://esi.evetech.net/latest/universe/ids/?language=en"
```

返回：

```json
{"inventory_types":[{"id":12807,"name":"Spike L"},{"id":12803,"name":"Javelin L"},
                    {"id":12791,"name":"Void L"},{"id":12787,"name":"Null L"}]}
```

> 名称要**精确**（含尺寸后缀与空格）：`Spike L` 而不是 `Spike`。
> 注意返回里还可能有 `characters` 字段——玩家角色名与物品名可能重名，别取错。

### 步骤 2：typeID → 官方本地化名称

```bash
curl -s "https://esi.evetech.net/latest/universe/types/12807/?language=zh"
```

返回中被删减后的关键字段：`name`（本地化名）、`description`（本地化描述）。

### 批量查询脚本（推荐）

```bash
python3 - << 'PYEOF'
import json, subprocess, time

NAMES = ["Void S","Null S","Spike S","Javelin S","Barrage S","Hail S"]

def curl(url, method="GET", data=None):
    cmd = ["curl","-s","--max-time","25"]
    if method == "POST":
        cmd += ["-X","POST","-H","Content-Type: application/json","-d", json.dumps(data)]
    cmd.append(url)
    return json.loads(subprocess.run(cmd, capture_output=True, text=True).stdout)

ids = curl("https://esi.evetech.net/latest/universe/ids/?language=en", "POST", NAMES)
tmap = {t["name"]: t["id"] for t in ids["inventory_types"]}

print(f"{'英文名':<16} {'typeID':<9} 官方中文名")
for en in NAMES:
    tid = tmap.get(en)
    if not tid:
        print(f"{en:<16} ❌ 未找到"); continue
    zh = curl(f"https://esi.evetech.net/latest/universe/types/{tid}/?language=zh")
    print(f"{en:<16} {tid:<9} {zh.get('name','?')}")
    time.sleep(0.05)   # 温和节流
PYEOF
```

> 内网/受限网络下给 curl 加 `-x <proxy>`。ESI 有错误率限制，批量查询请加小延时。

---

## 二、SDE：批量核对与看官方描述

### 下载

```bash
# 官方入口（会 302 到带版本号的 URL，如 eve-online-static-data-3607705-yaml.zip）
curl -L -o sde-yaml.zip \
  "https://developers.eveonline.com/static-data/eve-online-static-data-latest-yaml.zip"
```

| 项目 | 实测值（2026-10）|
|------|------------------|
| 体积 | 约 **97 MB** |
| 格式 | 102 个 `.yaml`，**扁平结构**（无子目录）|
| 下载耗时 | 约 16 s（带宽充足时）|

其他格式：把 `yaml` 换成 `jsonl` 即可（`sqlite` 版本已下线，返回 404）。

### 关键文件

| 文件 | 内容 |
|------|------|
| `types.yaml` | **53,288 个 typeID**，`name` 与 `description` 都是**多语言字典** |
| `translationLanguages.yaml` | 支持语言列表（ru/fr/en/zh/de/ko/ja/es）|

### `types.yaml` 的结构

```yaml
12618:
  basePrice: 150000.0
  description:
    de: 'Die Spitzmunition wurde…'
    en: 'The spike munition package is designed…'
    zh: '“钉刺”型弹药套件可对超远距离目标造成巨大破坏……最佳射程增加80%。跟踪速度降低75%。'
  name:
    de: Spike S
    en: Spike S
    zh: 钉刺 S
  portionSize: 1
  published: true
```

### 按 typeID 取中文名（流式，不必加载整个文件）

```bash
unzip -p sde-yaml.zip types.yaml | python3 - << 'PYEOF'
import sys, re
TARGETS = {12618:"Spike S", 12620:"Javelin S", 12612:"Void S", 12614:"Null S"}
cur=None; in_name=False; res={}
for line in sys.stdin:
    m = re.match(r'^(\d+):\s*$', line)
    if m:
        cur = int(m.group(1)); in_name=False; continue
    if cur not in TARGETS: continue
    if re.match(r'^  name:\s*$', line):
        in_name=True; continue
    if in_name:
        mm = re.match(r"^\s+zh:\s*(.+?)\s*$", line)
        if mm:
            res[cur] = mm.group(1).strip().strip("'\""); in_name=False
        elif re.match(r'^  \S', line) and not line.startswith('    '):
            in_name=False
for tid, label in TARGETS.items():
    print(f"{label:<14} {tid:<8} {res.get(tid,'❌ 未取到')}")
PYEOF
```

---

## 三、反查术语：用官方描述挖出正确中文译名

**这是最有价值的技巧。** 官方物品描述里会直接写出机制术语，例如「钉刺 S」的中文描述：

> 最佳射程增加80%。跟踪速度降低75%。……

于是这些术语一次确认：

| 概念 | 官方中文 | 常见错误写法 |
|------|----------|--------------|
| Optimal Range | **最佳射程** | ~~最优射程~~ |
| Accuracy Falloff | **失准范围** | ~~失准~~（作属性名时应带"范围"）|
| Tracking Speed | **跟踪速度** | ~~追踪速度~~ |
| Signature Radius | **信号半径** | — |
| Turret | **炮台** | ~~炮塔~~ |

### 查哪些物品能覆盖哪些术语

| 想确认的术语 | 去查这些物品的中文描述 |
|--------------|------------------------|
| 最佳射程 / 失准范围 / 跟踪速度 | T2 弹药：`Spike S`、`Null S`、`Barrage S`、`Conflagration S` |
| 跟踪速度 | 技能：`Motion Prediction`、`Trajectory Analysis`、`Sharpshooter` |
| 失准范围 | 技能：`Trajectory Analysis` |
| 信号半径 | 改装件：`Small Core Defense Field Extender I` |
| 炮台 | 技能：`Gunnery`、`Small Hybrid Turret`、`Small Energy Turret` |

```bash
# 一步到位：取中文描述并高亮术语
tid=12618   # Spike S
curl -s "https://esi.evetech.net/latest/universe/types/$tid/?language=zh" \
| python3 -c "import json,sys,re; d=json.load(sys.stdin); \
print(d['name']); print(re.sub(r'<[^>]+>',' ',d.get('description','')))"
```

---

## 四、本次已核实的译名（可直接引用）

弹药 / 武器 / 模块（ESI 与 SDE 两路交叉验证一致）：

| 英文 | 官方中文 | 英文 | 官方中文 |
|------|----------|------|----------|
| Void | 涅槃 | Null | 虚空 |
| Spike | **钉刺** | Javelin | 标枪 |
| Barrage | **雷暴** | Hail | 冰雹 |
| Quake | **地震** | Tremor | **战栗** |
| Scorch / Conflagration | 灼烧 / 爆燃 | Aurora / Gleam | 极光 / 微光 |
| Antimatter Charge | 反物质轨道弹 | Iron Charge | 铁质轨道弹 |
| Multifrequency | 多频晶体 | Radio | 射频晶体 |
| Standard / Gamma / Xray | 标准晶体 / 伽玛晶体 / X射线晶体 | EMP / Fusion | 电磁脉冲弹 / 聚变弹 |
| Magnetic Field Stabilizer | **磁性力场稳定器** | Gyrostabilizer | **回转稳定器** |
| Heat Sink | 散热槽 | Stasis Webifier | **停滞缠绕光束** |
| Tracking Computer | **索敌计算机** | Tracking Enhancer | **索敌增强器** |
| Target Painter | **目标标记装置** | Entropic Disintegrator | **熵能分解者** |
| Small Blaster / Railgun Specialization | 小型疾速炮 / 磁轨炮专业研究 | | |

> 注意几个容易错的：**钉刺**（非"尖峰"）、**涅槃=Void / 虚空=Null**（易被互换）、**熵能分解者**（非"分解器"）、**索敌**系列（避免与"跟踪速度"混淆——模块名用"索敌"，属性名用"跟踪"）。

---

## 五、限制与盲区

| 想核对的内容 | 能否核对 | 说明 |
|--------------|----------|------|
| 物品/模块/弹药/舰船的中文名 | ✅ | ESI `language=zh` 或 SDE |
| 物品的中文描述（含机制数值） | ✅ | 同上，**这是挖术语的最佳入口** |
| dogma 属性名 | ❌ 仅英文 | ESI `/dogma/attributes/{id}/` 忽略 `language`，恒返回英文 |
| UI 字符串（命中质量、总览列名、日志文案） | ❌ | 不在 SDE 中；只能在文档里保留英文并标注"以游戏内为准" |
| 技能名 | ✅ | 技能也是 type，如 `Motion Prediction` → 航迹预算学 |

### 无法访问时的替代方案（按优先级）

1. **ESI 直连**（通常可用，无需认证）
2. **SDE 下载**（约 97 MB，一次性）
3. 第三方 SDE 镜像 —— 注意多为旧版 SDE（`trnTranslations` 表结构），**可能不反映当前版本**
4. 官方帮助中心 API：`https://support.eveonline.com/api/v2/help_center/zh-cn/categories.json`（仅文章分类，无物品数据）
5. 中文 Wiki（灰机等）常被 Cloudflare 拦截，浏览器可开、脚本抓取通常失败

---

## 相关页面

- `术语表.md` — 通用术语与使用注意
- `资料来源.md` — 来源优先级总览
- `写作规范.md` — 铁律"术语中英并置"的落地方法
