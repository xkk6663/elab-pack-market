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

## 提交一个新芯片包

1. 在你的仓库里按官方模板建立 `pack.yaml` 与 `chips/**/*.yaml`
   （模板见 [elab-flow/packs/official](https://github.com/xkk6663/elab-flow/tree/main/packs/official)）。
2. 运行 `elab pkg install ./your-pack --from ./your-pack` 本地自测通过。
3. 计算内容哈希并提交 PR，在本文件 `packs[]` 追加一条：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | ✓ | `vendor/platform`，小写 |
| `vendor` / `platform` | ✓ | 与 `name` 前后缀一致 |
| `version` | ✓ | semver，如 `0.2.0` |
| `license` | ✓ | 必须是可再分发许可证（推荐 MIT/Apache-2.0） |
| `status` | ✓ | `official` / `community` |
| `requires` | ✓ | `kernel_api`（如 `>=1 <2`）与 `schema_version` |
| `chips` | ✓ | 包内芯片引用列表 |
| `source` | ✓ | `git` / `archive`（市场**不接受** `local`） |
| `sha256` | ✓ | 包内容哈希（`elab pkg index` 生成，安装时强校验） |

## 安全边界

索引只做分发，不执行任何代码。安装阶段会拒绝包内的脚本、可执行文件、
预编译产物与符号链接；C 源码允许，由用户本机工具链现场编译。

## 内置包

`st/stm32` 随 elab 骨架内置（`packs/official/st/stm32`），由
`elab pkg bootstrap` 直接安装，**不经过本市场**。
