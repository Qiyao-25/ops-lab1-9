# O3：门禁与发布流水线迁移

目标仓库：[YiningZhang401/ops-lab1-o3](https://github.com/YiningZhang401/ops-lab1-o3)。

## 实际迁移差异

| 方面 | 统一 Go 示例 | 迁移后的 Python 项目 |
|---|---|---|
| 应用 | 教学班冲突 HTTP 服务 | 统计词数与不区分大小写的不同词数 |
| 运行环境 | Go 1.26.9，编译为 linux/amd64 | 固定 Python 3.12.7，运行源码 |
| 检查 | make fmt-check、vet、test | python -m unittest test_wordcount.py |
| 制品 | 二进制与 BUILD_INFO | README、wordcount.py、测试及 BUILD_INFO 的源码包 |
| 版本注入 | ldflags 写入程序变量 | package.sh 写入 BUILD_INFO，无编译注入 |
| 平台条件 | 二进制依赖 OS/CPU 架构 | 源码包仍依赖兼容的 Python 解释器 |
| 发布保护 | tag 提交必须属于 origin/main | 保留相同祖先判断与失败退出机制 |

迁移时不能直接复制 Go 的 fmt/vet 命令或二进制路径，必须换成 Python 的测试入口及源码打包脚本。当前未新增 Python 格式或静态检查门禁，因此检查覆盖面与 Go 示例并不完全相同。源码打包保留了排序、固定属主与时间、gzip -n 和 .sha256；BUILD_INFO 改记 python=3.12.7，而非 go=。PR/push 两类事件、只在 main 打包、最少 1 个审批、必需 ci、版本 tag 触发和非主干发布拒绝机制均保留。

## 可核对证据

- [目标规则集](https://github.com/YiningZhang401/ops-lab1-o3/rules/25620230)：active，默认分支，禁止删除/强推，要求 1 个审批和 ci。
- [PR #1](https://github.com/YiningZhang401/ops-lab1-o3/pull/1)：增加不同词计数与测试，由另一位成员批准后合入。
- [PR CI](https://github.com/YiningZhang401/ops-lab1-o3/actions/runs/39745344094)：成功。
- [main CI](https://github.com/YiningZhang401/ops-lab1-o3/actions/runs/39746152869)：成功。
- [v0.1.0 Release](https://github.com/YiningZhang401/ops-lab1-o3/releases/tag/v0.1.0) 与 [发布运行](https://github.com/YiningZhang401/ops-lab1-o3/actions/runs/39740349516)：含源码压缩包及校验文件。
- [非主干 tag 的失败发布运行](https://github.com/YiningZhang401/ops-lab1-o3/actions/runs/39740360995)：未产生对应 Release。

该迁移证明同一门禁/发布结构能用于另一种语言，但不等于两个项目有相同测试覆盖率或部署方式。
