# 白兮的 Skills Sync 自动同步仓库

**基于 GitHub Actions 全自动同步第三方 Skill 仓库的聚合工具**

本仓库为**自动化聚合同步仓库**，通过 GitHub Actions 定时任务 \+ 自定义配置文件，全自动拉取、同步、整理全网各类技能仓库，统一聚合至本仓库 `skills/` 目录，实现技能资源统一管理、自动更新、轻量化托管。

## ✨ 项目核心特性

- **三同步模式**：支持 `full` 整仓同步、`dir` 指定目录整包同步、`subdir` 子目录批量平铺同步

- **智能命名规则**：full/dir 模式优先自定义 `name`，无配置自动识别仓库名 / subdir 末段；subdir 模式支持批量目录重命名映射

- **精准过滤规则**：subdir 模式仅同步目录，自动过滤根目录独立文件，杜绝冗余资源

- **自动清理机制**：自动清理过期、已移除配置的旧技能目录，保持仓库整洁

- **定时\+手动触发**：支持每日定时同步，也可手动一键运行更新

- **公私仓兼容**：配置个人 GitHub Token，支持同步公开/授权私有仓库资源

## 📁 仓库目录结构

```Plain Text
.
├── .github/workflows/
│   └── sync-skills.yml      # 核心自动同步工作流
├── skills/                  # 所有同步后的技能资源存放目录
├── repos.config.yml         # 三方仓库同步配置文件（核心配置）
├── .gitignore               # 忽略临时缓存文件
└── README.md                # 项目说明 & 免责声明
```

## ⚙️ 配置文件详解（repos\.config\.yml）

所有需要同步的三方仓库，统一在根目录 `repos.config.yml` 中维护，支持两种同步模式，配置极简、高度自定义。

### 通用配置字段说明

|字段|必填|说明|
|---|---|---|
|url|✅ 是|三方仓库 GitHub 地址（支持带 \.git 后缀/不带后缀）|
|mode|✅ 是|同步模式：`full` 整仓同步 / `dir` 指定目录整包同步 / `subdir` 子目录批量平铺同步|
|name|❌ 否|full/dir 模式生效：自定义同步后的文件夹名称；full 不填自动取原仓库名，dir 不填自动取 subdir 路径最后一段|
|subdir|dir/subdir 模式必填|三方仓库内目标子目录路径：dir 模式下为要整体打包成一个 skill 的目录；subdir 模式下为需要批量平铺的目录|
|names|❌ 否|仅 subdir 模式生效，目录重命名映射字典：`原目录名: 自定义新名称`|
|branch|❌ 否|指定同步分支，不填默认使用仓库默认分支|

### 配置示例

```Plain Text
repos:
  # 1. full 整仓同步 - 自定义文件夹名称
  - url: https://github.com/xxx/awesome-project.git
    mode: full
    name: custom-awesome-skill

  # 2. full 整仓同步 - 自动识别仓库名
  - url: https://github.com/xxx/tools-skill.git
    mode: full

  # 3. dir 指定目录整包同步 - 自定义名称
  # 把三方仓库里某个子目录整体当作一个 skill 拉过来
  - url: https://github.com/xxx/monorepo.git
    mode: dir
    subdir: packages/skill-editor
    name: editor-skill

  # 4. dir 指定目录整包同步 - 自动取 subdir 末段作为名称
  - url: https://github.com/xxx/monorepo.git
    mode: dir
    subdir: tools/skill-formatter

  # 5. subdir 子目录批量平铺 - 沿用原目录名
  - url: https://github.com/xxx/skill-bundle.git
    mode: subdir
    subdir: skills

  # 6. subdir 子目录批量平铺 - 批量重命名目录
  - url: https://github.com/xxx/skill-collection.git
    mode: subdir
    subdir: skills
    names:
      old-skill-1: new-skill-01
      old-skill-2: new-skill-02
```

### 核心同步规则

- **full 模式**：完整克隆三方仓库根目录（剔除 \.git 缓存），优先使用自定义 `name`，无配置则自动提取仓库名，整体放到 `skills/<name>/`

- **dir 模式**：把三方仓库里 `subdir` 指定的子目录**整体作为一个 skill** 拉取，复制到 `skills/<name>/`；`name` 不填时自动取 `subdir` 路径最后一段；适合 monorepo 仓库里只同步其中某个工具目录的场景

- **subdir 模式**：仅同步指定子目录下的**文件夹**，自动过滤所有独立文件；支持批量重命名，平铺合并至本仓库 `skills/` 目录

- **自动清理**：每次同步后，自动删除 skills 目录下未在配置文件中声明的旧资源，保证目录纯净

## 🚀 部署使用教程

### 1\. 仓库初始化

1. 新建空 GitHub 仓库，克隆本项目核心文件（`.github/workflows/`、`repos.config.yml`、本 README）

2. 创建空 `skills/` 目录，用于存放同步资源

### 2\. 开启工作流权限

仓库 `Settings > Actions > General`，开启 **Read and write permissions**，允许工作流提交、推送代码变更。

### 3\. 触发同步

- 自动触发：每日 UTC 17:00（北京时间次日凌晨1点）自动同步更新

- 手动触发：进入仓库 Actions，选择 `Sync third-party skills`，点击 Run workflow 即可一键更新

## ⚠️ 免责声明（重要）

**请在使用本仓库前仔细阅读以下免责条款，使用即代表完全同意本声明！**

1. **资源来源说明**：本仓库所有 `skills/` 目录内的资源，均为**第三方公开开源仓库自动同步聚合所得**。所有代码、脚本、资源的版权、著作权均归原作者及原仓库所有，本仓库仅做自动化整理、聚合、托管展示，**不拥有任何同步资源的版权**。

2. **非官方项目**：本项目为个人开源自动化工具，**非任何官方项目**，与各三方源仓库、平台无任何隶属、合作、授权关系。

3. **使用风险自负**：本仓库及配套工具仅用于**个人学习、技术研究、开源交流**。使用者下载、运行、修改、使用所有同步资源产生的**一切直接或间接后果、风险、纠纷**，均由使用者自行承担，本项目作者不承担任何法律责任与连带赔偿责任。

4. **合规使用要求**：使用者必须严格遵守对应三方仓库的开源协议、GitHub 平台规则及所在地区法律法规。若因使用者违规商用、破解、篡改、非法传播资源引发的侵权、处罚、纠纷，全部责任由使用者自行承担。

5. **侵权处理机制**：若原作者发现本仓库同步内容存在版权侵权、违规收录等问题，可直接提交 Issue 或 PR，我将第一时间删除对应违规资源、终止对应仓库同步。

6. **无担保声明**：本自动化工具及同步资源均按「现状」提供，**不提供任何明示或暗示的担保**，包括但不限于可用性、完整性、安全性、适用性、无漏洞等保障，不保证同步资源的实时性、准确性。

7. **禁止违规用途**：严禁将本仓库资源用于破解、侵权、非法盈利、网络攻击、违规开发等违法违规场景，违规使用后果自负。

## 📄 开源许可

本项目**自动化脚本、工作流、配置文件、README 文档**基于 **MIT License** 开源。

同步的第三方技能资源，版权及许可协议以**原三方仓库声明为准**，请使用者自行甄别、合规使用。

## 🤝 补充说明

- 本项目初衷：简化个人技能资源管理，自动化聚合优质开源技能项目，降低学习与收纳成本

- 欢迎 Star、Fork、交流学习，禁止商用及违规二次分发

- 若有配置优化、功能建议，可提交 Issue 交流
