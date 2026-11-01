# elab-pack-market

elab 芯片适配包的**零服务器社区市场索引**。这里不放包内容，只放一份
`index.json`；包本体留在各作者自己的仓库里。

## 使用

```bash
# elab.host.yaml
packs:
  market: https://raw.githubusercontent.com/xkk6663/elab-pack-market/main/index.json

elab pkg search
elab pkg install artery/at32
```

`elab pkg` 读取本索引 → 按 `source` 拉取作者仓库 → 校验 `sha256` → 安装到
`packs/installed/<name>/`。远程索引不可达时自动回退到本仓 `market/index.json`。

## 上架一个新芯片包（全自动，无人工审核）

本索引由 [`.github/workflows/index.yml`](.github/workflows/index.yml) **每小时自动生成**，
不需要 PR、不需要评审、不需要手填 `index.json`：

1. 在你的仓库里按官方模板建立 `pack.yaml` 与 `chips/**/*.yaml`
   （模板见 [elab-flow/packs/official](https://github.com/xkk6663/elab-flow/tree/main/packs/official)）。
2. 仓库根放 `.gitattributes`，内容为 `* -text`（内容哈希按字节计算）。
3. 给仓库加 topic **`elab-pack`**（仓库名含 `elab-pack` 也会被发现）。
4. 等下一轮（≤1 小时）。通过机器校验即自动出现在 `index.json`。

校验规则（schema / `requires` 握手 / 安全审计 / `.gitattributes` / 哈希由 bot 重算 /
同名包取高版本）见
[elab-flow/market/CONTRIBUTING.md](https://github.com/xkk6663/elab-flow/blob/main/market/CONTRIBUTING.md)。
不合格仓库只跳过，不阻断其他包；每轮结果在 Actions 的 `report` 产物里（谁被拒、为什么）。

索引字段说明（机器生成，供排查参考）：

| 字段 | 说明 |
|---|---|
| `name` / `vendor` / `platform` | `vendor/platform`，小写，与 `pack.yaml` 一致 |
| `version` | semver（改包内容必须升版本） |
| `license` | SPDX 标识，与仓库 LICENSE 一致 |
| `status` | 市场条目恒为 `community`（`official` 只给随骨架分发的内置包） |
| `requires` | `kernel_api` 与 `schema_version`，不满足时安装 fail-fast |
| `chips` | 包内芯片引用列表 |
| `source` | `git` + `ref`（最高 semver tag；无 tag 则默认分支 commit SHA） |
| `sha256` | 整包内容哈希，由 bot 用 `tree_hash` 计算，安装时强校验 |

**下架**：删掉仓库的 `elab-pack` topic（或归档仓库），下一轮自动移除。
恶意包可由维护者在 `blocklist.json` 中紧急封禁。

## 安全边界

索引只做分发，不执行任何代码。安装阶段会拒绝包内的脚本、可执行文件、
预编译产物与符号链接；C 源码允许，由用户本机工具链现场编译。

## 内置包

`st/stm32` 随 elab 骨架内置（`packs/official/st/stm32`），由
`elab pkg bootstrap` 直接安装，**不经过本市场**。
