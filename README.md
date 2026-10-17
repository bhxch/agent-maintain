# agent 维护机器的工作区模板

这不是一个代码仓库，而是一台机器的**维护流水账**。所有由 agent（Claude Code 等）在本机执行的维护动作——改配置、装服务、动系统文件——都在这里留下可追溯的记录；工作区之外的文件在被修改前，会先快照进本仓库的 `backup/`。

把本仓库复制到任何一台机器上，即可作为该机器的 agent 维护工作区使用。完整的强制规范在 [`AGENTS.md`](AGENTS.md)（面向 agent），本 README 是面向人的说明。

## 目录结构

```
agent-maintain/
├── AGENTS.md          # agent 行为规范（强约束，每次会话必须遵守）
├── README.md          # 本文件
├── action/            # 维护流水账
│   └── 202610/        # 按 YYYYMM 建子目录
│       └── 20261017-概要.md   # 文件名 YYYYMMDD-概要，同一天内倒序追加
├── backup/            # 本机文件快照，按真实绝对路径映射
│   └── etc/hosts      # 如 /etc/hosts -> backup/etc/hosts
└── bin/
    └── backup-file    # 备份脚本：快照外部文件进 backup/ 并自动 commit
```

> `action/`、`backup/` 为空目录时不入 git。克隆后若不存在，先 `mkdir -p action backup`。

## 两类记录

### 1. `action/` —— 做了什么

agent 每执行一个维护动作，就往对应月份的文件里追加一条，至少包含：时间、对象、操作、原因、结果、回滚方式。规则：

- **边做边记**，不接受"先做完，回头补总结"
- **只追加不修改**，写错了就追加更正条目说明原条目错在哪
- 条目模板见 `AGENTS.md`

### 2. `backup/` —— 改之前长什么样

修改工作区之外的任何文件前，先运行：

```bash
bin/backup-file -m "为什么要改" /path/to/file
```

- 按**真实绝对路径**映射存入 `backup/`（如 `/home/<用户名>/.claude/CLAUDE.md` -> `backup/home/<用户名>/.claude/CLAUDE.md`）
- 内容无变化时自动跳过，不产生 commit
- 疑似密钥/凭据的路径（`.ssh`、`.pem`、`.key`、`.env`、`token` 等）默认拒绝备份
- 修改完成后再跑一次同一命令，保存修改后的版本

`backup/` 的 commit message 只有一句 `backup: why`；完整上下文（做了什么、结果、如何回滚）看对应的 `action/` 条目。

## 一次典型的维护动作

```bash
# 1. 改前快照
bin/backup-file -m "准备调整 sysctl 网络参数" /etc/sysctl.conf

# 2. 执行修改（省略）

# 3. 改后快照
bin/backup-file -m "调整 sysctl 网络参数" /etc/sysctl.conf

# 4. 在 action/202610/20261017-调sysctl网络参数.md 顶部追加条目
# 5. git add action/... && git commit（message 先说 why，再一句话概括 what）
```

## 用作新机器的模板

1. 把仓库放到新机器上（clone 或复制均可）
2. 按新机器实际情况修改 `AGENTS.md`（git 身份、push 方式、目录约定等）
3. 如目录不存在，`mkdir -p action backup`
4. 之后 agent 在该机器上的所有维护动作，一律按 `AGENTS.md` 记录到 `action/`

## 边界

- 不提交密钥、令牌、密码（`.env` 之类不进仓库，`backup-file` 会自动拦截）
- 不粘贴大段终端输出，只留与结论相关的内容
- 历史记录文件永不删除——过时本身也是记录的一部分
