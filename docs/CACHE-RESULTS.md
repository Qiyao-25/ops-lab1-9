# O2 缓存对照结果

实验 PR：[#15](https://github.com/Qiyao-25/ops-lab1-9/pull/15)，保留但不合入 main。
实验前预期已先提交到该 PR 的 docs/CACHE-EXPERIMENT.md，随后依次测量 cache:true 和 cache:false，各 5 次。
基线 main 提交 a04614b0ec47bdcbbbedd355ac6454419e391b28；开启组分支 head 为 c60ae66396fac2d75027c2976e72cdb5207ba644，关闭组为 94d9c0417fed03f3d002b968030cd9044b2ec30f。
两组 PR 临时合并 SHA 分别为 4d8abe9e26210d3b8cc8a38b3123b201c59498d7、394be44f4f24b87ab3f0fe038b68fb47c6451001，组内 5 次尝试一致。

测量 UTC 区间：2026-10-23T10:22:39Z 至 2026-10-23T10:32:32Z。北京时间为 UTC+8。工作流计时统一采用最后一个 ci job 的 completed_at 减该 attempt 的 run_started_at，含 job 调度等待；job 计时只用 started_at 到 completed_at。步骤时间来自 GitHub 秒级时间戳，0 s 不代表实际零耗时。

| cache | 运行 / attempt | 工作流计时 | job 计时 | setup-go / vet / test | 缓存结果 | runner 镜像版本 |
|---|---|---|---|---|---|---|
| true | [ci #33 / 1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39938656519/attempts/1) | 12 s | 8 s | 1/1/1 s | hit | 20261012.334.1 |
| true | [ci #33 / 2](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39938656519/attempts/2) | 21 s | 17 s | 3/5/1 s | hit | 20261012.334.1 |
| true | [ci #33 / 3](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39938656519/attempts/3) | 19 s | 15 s | 2/7/1 s | hit | 20261012.334.1 |
| true | [ci #33 / 4](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39938656519/attempts/4) | 22 s | 18 s | 4/5/2 s | hit | 20261012.334.1 |
| true | [ci #33 / 5](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39938656519/attempts/5) | 15 s | 11 s | 1/3/1 s | hit | 20261012.334.1 |
| false | [ci #34 / 1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39939386365/attempts/1) | 29 s | 25 s | 1/15/3 s | disabled | 20261012.334.1 |
| false | [ci #34 / 2](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39939386365/attempts/2) | 29 s | 25 s | 1/16/2 s | disabled | 20261012.334.1 |
| false | [ci #34 / 3](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39939386365/attempts/3) | 32 s | 27 s | 1/17/3 s | disabled | 20261012.334.1 |
| false | [ci #34 / 4](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39939386365/attempts/4) | 32 s | 28 s | 1/19/3 s | disabled | 20261012.334.1 |
| false | [ci #34 / 5](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39939386365/attempts/5) | 34 s | 28 s | 1/18/3 s | disabled | 20261012.334.1 |

开启缓存的工作流计时为 [12.0, 21.0, 19.0, 22.0, 15.0] s，中位数 19 s；关闭缓存为 [29.0, 29.0, 32.0, 32.0, 34.0] s，中位数 32 s。job 计时中位数分别为 15 s 和 27 s。以关闭缓存组中位数为基准，本次开启缓存的工作流计时减少约 40.6%。

本例没有第三方依赖，不能把差异解释为依赖下载量的减少；setup-go 的 Go 构建缓存也影响后续 vet/test。两组 setup-go/vet/test 步骤中位数分别为 2/5/1 s 与 1/17/3 s，可结合缓存恢复日志解释各步骤的变化。两组均采用相同 ubuntu-24.04 镜像版本和 Go 1.26.9，PR 事件及应用代码不变，第二个提交只改变 cache 配置。组内波动仍来自共享 runner 负载、网络、缓存传输及秒级计时粒度；两个组顺序执行、每组只有 5 次，所以这个比例只是本组本次条件下的观测，不是对其他项目或后续运行的保证。

开启组的缓存命中来自 setup-go 日志，关闭组日志均确认 cache:false；原始摘要见 [cache-comparison.json](../evidence/cache-comparison.json)。原先使用历史“首次未命中”与后续混合事件作对照而得到的 40%～60% 说法撤回，本次结论只使用以上受控的 10 个样本。
