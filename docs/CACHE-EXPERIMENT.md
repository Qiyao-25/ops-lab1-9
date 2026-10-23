# O2：缓存对反馈时间的对照实验

## 实验前预期

本项目只依赖 Go 标准库，第三方模块下载量很小，因此不能预设缓存一定显著缩短反馈。setup-go 还会保存 Go 构建缓存，可能影响 vet/test；缓存恢复与保存也有成本。在共享 runner 上，净收益的大小和方向都需要数据验证。

本文件先于测量提交，保留实验前预期。此实验 PR 只用于测量，**不合入 main**。

## 固定条件与执行顺序

1. 从 main 提交 a04614b0ec47bdcbbbedd355ac6454419e391b28 建立 experiment/cache-comparison 分支。
2. 先只提交本说明，ci.yml 保持 cache:true，创建 PR 后记录首次运行，并对同一运行使用 Re-run all jobs 再做 4 次，共 5 次。
3. 然后在同一 PR 中只把 setup-go 的 cache:true 改为 cache:false，记录该提交的首次运行，再对该运行重跑 4 次，共 5 次。
4. 两组测量期间不更新 main 或应用代码，不合并本 PR，保留所有尝试。两组都固定 ubuntu-24.04、Go 1.26.9，仍为 pull_request 事件，不执行 main 的 package/upload。
5. 记录完整 run_id、run_attempt、head SHA 与 github.sha、UTC 起止时间、总时长、setup-go/vet/test 步骤时长、缓存命中或禁用信息，以及 runner 镜像版本。
6. 把全部 10 个样本列入最终 LAB1.md，分别给出中位数与范围；如 runner 镜像改变或执行失败，保留并注明，不能只挑有利样本。

当前准备的 ci.yml 是第 3 步的最终 cache:false 文件；第 2 步应保留原 main 的 cache:true，避免先推送 false 而丢失实验前的对照。

## 待实测记录

| 配置 | 样本 | run_id / attempt | 总时长 | setup-go / vet / test | 缓存状态 | runner 镜像 |
|---|---|---|---|---|---|---|
| cache:true | 1 | 待运行 | — | — | — | — |
| cache:true | 2 | 待运行 | — | — | — | — |
| cache:true | 3 | 待运行 | — | — | — | — |
| cache:true | 4 | 待运行 | — | — | — | — |
| cache:true | 5 | 待运行 | — | — | — | — |
| cache:false | 1 | 待运行 | — | — | — | — |
| cache:false | 2 | 待运行 | — | — | — | — |
| cache:false | 3 | 待运行 | — | — | — | — |
| cache:false | 4 | 待运行 | — | — | — | — |
| cache:false | 5 | 待运行 | — | — | — | — |

cache:true 组的首次运行也可能缓存未命中，应如实记录，不能把它改称 cache:false。两组采用顺序执行，仍可能受时段、网络和共享 runner 负载影响；样本很少，也不能代表大型依赖项目。若计算节省比例，应以同类事件、同类计时边界的两组中位数为基准，并同时展示全部原始数据。

