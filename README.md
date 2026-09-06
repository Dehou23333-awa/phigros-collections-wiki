# Phigros 收集品 Wiki（公开仓库）

> 自动维护的 Phigros 全收集品资料库（五语言：简中 / 繁中 / English / 日本語 / 한국어）。
> 数据自官方 APK 逆向提取，包含**正文**在内的全部收集品字段。

- 站点：GitHub Pages（见下方「部署」）
- 数据流水线：全自动 `TapTap 下载 APK → 解包 → 生成 Wiki → 推送 + 部署`（每周一、四 04:00 UTC，或手动触发）
- 当前数据：`wiki/` 目录为五语言源文档；页面顶替标签按章节分类

---

## 仓库结构（安全模型）

| 仓库 | 可见性 | 内容 |
|---|---|---|
| **本仓库（公开）** | Public | 仅 wiki 源文档（`wiki/`）+ 本 CI 配置（仓库名：`phigros-collections-wiki`） |
| **私有仓库**（`Dehou23333-awa/phigros-collections-scripts`） | Private | 解包 / 下载 / 渲染脚本（`tools/`） |

**原理**：`Actions` 在本公开仓库运行；第一步从私有仓库拉取脚本（第二步 `checkout` + `secrets.SCRIPTS_REPO_PAT`），因此逆向脚本永不公开，而 wiki 自动更新可控、可审计。

## 首次设置

1. **准备私有仓库** `phigros-collections-scripts`（Private）：
   - 内容 = `work/scripts/`（01_fetch_apk.py、02_unpack_to_data.py、03_data_to_wiki.py、run_all.py、apk_common.py、requirements.txt、typetree.json）
   - 推送后无需任何 CI。
2. **创建 PAT**（GitHub → Settings → Developer settings → Personal access tokens → Fine-grained）：
   - 权限：仅「Contents: Read」作用于私有仓库即可；工作流用 `actions/checkout` 拉取。
3. **本仓库设置 Secrets**：
   - `Settings → Secrets and variables → Actions → New repository secret`
   - `SCRIPTS_REPO_PAT` = 上面的 PAT
4. **本仓库推送一次**（见下方「首次推送」）。
5. **开启 Pages**：仓库 `Settings → Pages → Source: GitHub Actions`（部署交给 `deploy-pages` action）。

## 首次推送

```bash
# 公开仓库（本目录推入 git 的内容已就绪）
cd phigros-collections-wiki
git remote add origin https://github.com/Dehou23333-awa/phigros-collections-wiki.git
git push -u origin main

# 私有仓库（脚本）
cd phigros-collections-scripts
git remote add origin https://github.com/Dehou23333-awa/phigros-collections-scripts.git
git push -u origin main
```

> 顺序：先建/推私有仓库 → 再推公开仓库（首次 CI 需要能拉到私有仓库）。

## 手动触发更新

- 仓库 `Actions → Update Wiki (auto) → Run workflow`。
- 或修改 `version/.current` 并推送（记录当前数据版本）。

## 本地维护

```bash
# 依赖
pip install -r tools/requirements.txt

# 全流程（TapTap 自动下载最新版）
python -X utf8 tools/run_all.py

# 只重建 wiki（已有数据）
python -X utf8 tools/03_data_to_wiki.py --data-dir 收集品数据 --wiki-dir wiki

# 本地预览
python -m mkdocs serve -f wiki/mkdocs.yml
```

## 数据说明

- 来源：`assets/bin/Data/level22` → `SaturnOSControl`（MonoBehaviour pathID 1124，完整序列化负载）
- 交叉验证：`level0` `GetCollectionControl`（459 条期望 key，0 缺失、0 标题不一致）
- 每个收集品：章节 / 收集时间 / 等级 / 保管单位（5 语言）/ 标题（5 语言）/ 正文（5 语言）/ 附加信息
- 覆盖：简中 459 篇正文；繁中 / 英 / 日 / 韩 按官方译文覆盖；缺失处直接展示简体中文原文

> 非官方项目，与 Pigeon Games 无关。数据仅用于资料整理。