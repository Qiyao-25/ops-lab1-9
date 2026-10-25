# 实验 1 记录：第 9 组

补充实验于 2026-10-23 完成，相关代码及此前记录已通过 PR #16 审批并合入 main。main CI #36、v1.0.2 Release 和 GHCR 镜像发布均成功；原版本与修订版的追溯、运行及回退结果见第 4、9 节。O1 镜像、O2 缓存对照及 O3 迁移证据均已补齐。main 受保护，收尾记录仍通过独立 PR 和其他组员审批合入。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 9 组 |
| 仓库地址 | https://github.com/Qiyao-25/ops-lab1-9/ |
| 代码载体 | 统一示例  |
| runner 与语言版本 | `ubuntu-24.04`，Go 1.26.9 |
| 合并方式 | squash；已核对仓库仅允许 squash，并启用合并后自动删除分支。 |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | name=enroll-conflict；version=1.0.0；build_no=1；run_id=local；commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1；go=go1.26.9 |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | BUILD_INFO 未变化，但二进制文件的 SHA-256 发生变化。第一次：f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e；第二次：6bcad78cf2239992e60985ae0e11215dab59c1fd7d9809d0a7438df8ebe0b61c。|
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | vcs=git、vcs.revision 和 vcs.time 均相同，只有 vcs.modified 从 false 变为 true，表明第二次构建存在未提交的代码修改。|
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 只能确定程序名称、版本及记录的提交号，无法确定构建机器和构建时间。代码来源信息可能不准确：第二次构建包含未提交的修改，但 commit 仍指向原提交，无法完整反映实际使用的源码。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | 规则集名称：opsRule，状态：Active，作用于默认分支 main，Bypass list 已核对为空。启用规则：① Restrict deletions；② Block force pushes；③ Require a pull request before merging，要求至少 1 名其他组员审批；④ Require status checks to pass，必需检查为 ci。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 见下方实际终端输出。GitHub 返回 GH013，明确提示必须通过 PR 修改 main，且需要通过 ci 状态检查。 |
| 第一轮：失败的 PR 运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39454757987 |
| 第一轮：修复后通过的 PR 运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39456563807 |
| 第一轮：合入 main 后的运行（含制品） | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39458610700 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | `weeksOverlap` 在双方都是双周时返回没有交集，违背“双周对双周才有交集”的规则。现有测试覆盖了单周对单周、单周对双周和双周对每周，没有双周对双周，所以原先的 `make test` 不能发现它。 |
| 第二轮：仅含缺陷时仍然通过的运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535035335 |
| 第二轮：补测试后失败的运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535181115 |
| 第二轮：撤销缺陷后通过的运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535403658 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713593313 |
| Release v1.0.0 | https://github.com/Qiyao-25/ops-lab1-9/releases/tag/v1.0.0 |
| Release v1.0.1 | https://github.com/Qiyao-25/ops-lab1-9/releases/tag/v1.0.1 |
| 任务 2：下载 main 制品并追溯 | [ci #32](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39754271411)，artifact ID 12414887025；下载、解压后 trace 退出码 0，见 [输出](evidence/trace-ci32.txt)。 |

[main e3e5e33] test: direct push
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 186 bytes | 186.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/Qiyao-25/ops-lab1-9/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Changes must be made through a pull request.
remote:
remote: - Required status check "ci" is expected.
remote:
To https://github.com/Qiyao-25/ops-lab1-9.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/Qiyao-25/ops-lab1-9.git'
HEAD is now at 3bd8d71 basci info (#2)
## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | `0e68161b2924d975d970e3ca9c97b0935519f754` | `0e68161b2924d975d970e3ca9c97b0935519f754` |
| 引入该提交的 PR | [#6](https://github.com/Qiyao-25/ops-lab1-9/pull/6) | [#6](https://github.com/Qiyao-25/ops-lab1-9/pull/6) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713235034 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39723834091 |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `4a5cea3713d011296c8d6fe18269f82aed47b220a98e26fb246dee569b82416c` | `1f68ffc908cef53991bbc55697bd20b0de4ef5b9df39c8d59b4a56892587e0f5` |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？

不能。`github.run_number` 在每个工作流中独立计数，因此相同构建号可能对应不同运行。v1.0.0 的构建号为 1：在 `ci` 中，对应 [ci #1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39383173108)；在 `release` 中，对应 [release #1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713235034)。实际构建 v1.0.0 发布附件的是后者，可通过附件 `BUILD_INFO` 中的 `run_id=39713235034` 唯一定位。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`4a5cea3713d011296c8d6fe18269f82aed47b220a98e26fb246dee569b82416c`。与上表中发布当时的值是否一致？一致。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39713235034
commit=0e68161b2924d975d970e3ca9c97b0935519f754
go=go1.26.9
== 3. 提交 0e68161b2924d975d970e3ca9c97b0935519f754
作者: Lu Yakun <3336702498@qq.com>
时间: 2026-10-22T10:44:17+08:00
标题: ci: add gated tag release workflow (#6)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/Qiyao-25/ops-lab1-9/commit/0e68161b2924d975d970e3ca9c97b0935519f754
构建它的运行: https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713235034
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39723834091
commit=0e68161b2924d975d970e3ca9c97b0935519f754
go=go1.26.9
== 3. 提交 0e68161b2924d975d970e3ca9c97b0935519f754
作者: Lu Yakun <3336702498@qq.com>
时间: 2026-10-22T10:44:17+08:00
标题: ci: add gated tag release workflow (#6)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/Qiyao-25/ops-lab1-9/commit/0e68161b2924d975d970e3ca9c97b0935519f754
构建它的运行: https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39723834091
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
运行 v1.0.1 时：
{"version":"1.0.1","commit":"0e68161b2924d975d970e3ca9c97b0935519f754","build_no":"3","go":"go1.26.9"}

换回 v1.0.0 后：
{"version":"1.0.0","commit":"0e68161b2924d975d970e3ca9c97b0935519f754","build_no":"1","go":"go1.26.9"}
```

### 发布顺序核对与补充复测

v1.0.0 与 v1.0.1 均指向 PR #6 的提交。v1.0.1 发布时间为 2026-10-22 12:42:26+08:00，而准备发布的 [PR #10](https://github.com/Qiyao-25/ops-lab1-9/pull/10) 于 12:47:29+08:00 才合入，因此 v1.0.1 未包含该新改动。此处如实保留；不移动、不删除、不复用已发布 tag。增强版本接口测试等改动已经 [PR #16](https://github.com/Qiyao-25/ops-lab1-9/pull/16) 审批合入，随后于 2026-10-23 19:06:18+0800 发布修订版 [v1.0.2](https://github.com/Qiyao-25/ops-lab1-9/releases/tag/v1.0.2)，已补充新版本追溯和回退。v1.0.1 的 Release 说明已添加更正及修订版链接；旧标签和附件保持原样。是否认可其补足任务 6 指定 v1.0.1 步骤由教师核验。

2026-10-23 复测：直接使用原 Release 附件，先运行 1.0.1，再在同一仅绑定 127.0.0.1 的端口切换为 1.0.0，得到上列两次 /version 内容；每次停止自己启动的进程后再切换。两个压缩包的哈希与表中一致，两个 trace 均退出 0。见 [复测记录](evidence/local-validation.json)、[v1.0.0 trace](evidence/trace-v1.0.0.txt)、[v1.0.1 trace](evidence/trace-v1.0.1.txt)。

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线（北京时间，精确到秒；提交时间不等同于服务器收到推送的时间）：

| 事件 | 时间 | 链接或依据 |
|---|---|---|
| 发起 PR #4 | 2026-10-21 00:08:09+08:00 | [PR #4](https://github.com/Qiyao-25/ops-lab1-9/pull/4) 的 created_at |
| 失败 CI 触发 / 结束 | 00:08:14 / 00:08:26+08:00 | [ci #8](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39454757987) |
| 修复提交的提交时间 | 00:17:24+08:00 | [修复提交](https://github.com/Qiyao-25/ops-lab1-9/pull/4/commits/ce000f59958f4e8c2b3e32ccfefb1f343c0ef849) 的 committer 时间 |
| 修复 CI 触发 / 结束 | 00:20:28 / 00:20:48+08:00 | [ci #9](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39456563807) |
| 另一成员批准 | 00:32:18+08:00 | PR #4，Qiyao-25 的 APPROVED 记录 |
| squash 合入 main | 00:34:15+08:00 | PR #4 merged_at；提交 eee7836dcea975d3281df198c1bd62c03cfff9a6 |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #4 | OCT 21, 2026, 00:03 GMT+8 | OCT 21, 2026, 00:34 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 34 小时 43 分 |
| #6 | OCT 21, 2026, 09:56 GMT+8 | OCT 22, 2026, 10:44 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 24 小时 50 分 |
| #8 | OCT 21, 2026, 11:33 GMT+8 | OCT 21, 2026, 23:20 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 23 小时 13 分 |

工作流运行时长：以下保留 2026-10-22 17:17:46+08:00 之前的历史快照，共 32 次 ci 和 3 次 release，均为 attempt 1；2026-10-23 的在线补充另列。统一使用 updated_at − created_at 的墙钟时间，含调度和收尾开销；不是 CPU 时间。若后续重跑，另列 attempt，不覆盖这份原始快照。

| 运行 | 事件 | 时长 | 结果与备注 |
|---|---|---|---|
| [ci #1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39383173108) | push | 44 s | success； |
| [ci #2](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39410831475) | pull_request | 20 s | success； |
| [ci #3](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39411996609) | push | 16 s | success； |
| [ci #4](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39414745281) | pull_request | 14 s | success； |
| [ci #5](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39415231970) | push | 20 s | success； |
| [ci #6](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39416746594) | pull_request | 19 s | success； |
| [ci #7](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39416914626) | push | 20 s | success； |
| [ci #8](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39454757987) | pull_request | 12 s | failure；第一轮测试失败 |
| [ci #9](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39456563807) | pull_request | 20 s | success；第一轮修复通过 |
| [ci #10](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39458610700) | push | 20 s | success； |
| [ci #11](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39463619809) | pull_request | 20 s | success； |
| [ci #12](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39464180284) | push | 27 s | success； |
| [ci #13](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39530419549) | pull_request | 16 s | success； |
| [ci #14](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535035335) | pull_request | 13 s | success；第二轮缺陷漏检 |
| [ci #15](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535181115) | pull_request | 20 s | failure；第二轮补测试失败 |
| [ci #16](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39535403658) | pull_request | 16 s | success；第二轮修复通过 |
| [ci #17](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39539267635) | pull_request | 12 s | success； |
| [ci #18](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39624800252) | push | 19 s | success； |
| [ci #19](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39624847247) | push | 25 s | success； |
| [ci #20](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713022852) | push | 23 s | success； |
| [ci #21](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39714221823) | pull_request | 14 s | success； |
| [ci #22](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39716100651) | pull_request | 16 s | success； |
| [ci #23](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39724341711) | push | 17 s | success； |
| [ci #24](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39725330504) | pull_request | 15 s | success； |
| [ci #25](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39741348128) | pull_request | 18 s | success； |
| [ci #26](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39743818667) | push | 19 s | success； |
| [ci #27](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39744456229) | pull_request | 21 s | success； |
| [ci #28](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39747810023) | pull_request | 16 s | success； |
| [ci #29](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39753237966) | pull_request | 20 s | success； |
| [ci #30](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39753656914) | push | 21 s | success； |
| [ci #31](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39754102168) | pull_request | 14 s | success； |
| [ci #32](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39754271411) | push | 16 s | success； |
| [release #1](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713235034) | push | 16 s | success；v1.0.0 |
| [release #2](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39713593313) | push | 8 s | failure；发布 gate 拒绝 |
| [release #3](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39723834091) | push | 23 s | success；v1.0.1 |

32 次 ci 中位数 19 s、范围 12～44 s；其中 19 次 PR 运行中位数 16 s，13 次 main push 中位数 20 s。事件类型的步骤不同，不能把两者时长差全部归因于缓存。第 5 节原“ci #9 失败”已纠正为 ci #8 失败、ci #9 修复通过。

### O2 在线补充：缓存开启与关闭各 5 次

实验 PR：[#15](https://github.com/Qiyao-25/ops-lab1-9/pull/15)，该 PR 不合并。先提交实验前预期，再测 cache:true，随后只改为 cache:false；基线、运行事件和应用代码保持一致。统一计时边界及 UTC 起止时间见 [完整说明](docs/CACHE-RESULTS.md)，原始数据见 [JSON](evidence/cache-comparison.json)。

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

开启缓存的工作流计时为 [12.0, 21.0, 19.0, 22.0, 15.0] s，中位数 19 s；关闭缓存为 [29.0, 29.0, 32.0, 32.0, 34.0] s，中位数 32 s。job 计时中位数分别为 15 s 和 27 s。以关闭缓存组中位数为基准，本次开启缓存的工作流计时减少约 40.6%。 这是相同 runner 镜像与固定 Go 下的 5+5 次小样本，顺序执行和共享 runner 噪声限制了因果推广，不能只用一次冷启动或把历史 PR/push 混在一起推断收益。

## 6. 分析问题

**问题 1：gate 的收益与成本。**

数据：第一轮 PR #4，ci #8 失败后 ci #9 修复通过；第二轮 PR #7，ci #14 漏检、#15 补测试失败、#16 修复通过；时间与运行链接见第 3、5 节。

- gate 拦住了什么：第一轮拦住了 PR #4 的格式错误（fmt-check 失败）；第二轮补测试后拦住了双周对双周的冲突缺陷。
- gate 没拦住什么：第二轮最初 ci 全绿却包含双周对双周缺陷，说明没写断言的缺陷拦不住。
- 第一轮因失败多花的时间：返工窗口 00:08:26 至 00:20:48，共 12 分 22 秒。
- 其中等待 ci 的时间：修复 ci 本身 20 秒；失败 ci 12 秒 + 修复 ci 20 秒 = 32 秒。
- 其中人工修改的时间：失败结果到修复提交 8 分 58 秒 + 提交到第二次 ci 触发 3 分 4 秒 = 12 分 2 秒（含人工处理、提交、推送等待）。
- 一次 ci 大约多久：32 次 ci 中位数 19 秒。
- 在"提交到发布的间隔"中占比：三个 PR 的间隔为 24～35 小时，ci 仅占约 0.015%～0.023%，可忽略；主要耗时在人工审批、协作和发布等待。
- 第二轮补测试后仍无法发现的问题：需求理解错误、权限与安全、真实环境性能、未建模的兼容性问题——这些不是单元测试断言能覆盖的。

**问题 2：PR 中的构建与 main 上的构建。**

数据：[ci #9 日志](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39456563807) 打印 event_name=pull_request、ref=refs/pull/4/merge、github.sha=87c3b2d01cd26e518d76fb453f9ba9e572d5a93b、pull_request.head.sha=ce000f59958f4e8c2b3e32ccfefb1f343c0ef849；[PR #4](https://github.com/Qiyao-25/ops-lab1-9/pull/4) squash 后 main 提交为 eee7836dcea975d3281df198c1bd62c03cfff9a6。

- github.sha 与 pull_request.head.sha 是否相同：不同。ci #9 中 github.sha=`87c3b2d`，pull_request.head.sha=`ce000f5`。
- 它们能否在 main 历史中找到：不能。github.sha 是平台构造的临时合并提交（`refs/pull/4/merge`），head.sha 是分支提交，两者均不在最终 main 历史中。
- 本组使用的合并方式：squash 合并。
- 合并后 main 上这次改动的提交号：`eee7836dcea975d3281df198c1bd62c03cfff9a6`。
- 为什么待发布制品只能从 main 上的提交或 tag 构建：PR 制品基于临时合并提交，可能含未合入代码或旧基线，无法保证"制品 ↔ 已合入代码"的对应关系；只有 main 上的提交经过审批合入，release gate 也限制 tag 必须指向 main 历史中的提交。

**问题 3：可重复构建。**

实验时间为 2026-10-23；同一提交 a04614b0ec47bdcbbbedd355ac6454419e391b28，Go 1.26.9，linux/amd64。固定输入：VERSION=1.0.0、BUILD_NO=100、RUN_ID=local-repro-20261023、COMMIT=上述 SHA；③仅变更 BUILD_NO。④、⑤在隔离副本临时修改 Makefile，实验结束已逐字恢复，Git 工作区重新干净。[原始测量](evidence/reproducibility.json)。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | 固定全部四个构建参数，工作区干净 | `4694424b73c4072392efbf18eed43ad55b40eb3c26352a9dba7903d49fc3f41b` | `5a4350341b9f491b93a3cf2403d7956266563bf34060068bc0acf9cfc2b3a380` |
| ② | 与 ① 完全相同 | `4694424b73c4072392efbf18eed43ad55b40eb3c26352a9dba7903d49fc3f41b` | `5a4350341b9f491b93a3cf2403d7956266563bf34060068bc0acf9cfc2b3a380` |
| ③ | 仅 BUILD_NO 从 100 改为 101 | `931dfa10151c0257660c3893dd1d36893c4953549554bcfb89ea95ae107d99aa` | `d7f7d2a9bdecc89218a859a22bca2e096319c4f380b651e980aa694b367203f1` |
| ④ | 去掉 --mtime=@0，其余同 ①；Makefile 未提交 | `fa377c27dfd57f7d4e069ac39629885c74b3ea8d3fe47e6d02b8a7d4ec18902a` | `59ea4d83b3cc38925b7f8202dd9ffdfc53a4e052c7c096537403cdd0bb0a29a7` |
| ⑤ | 同 ④，间隔 3 秒 | `fa377c27dfd57f7d4e069ac39629885c74b3ea8d3fe47e6d02b8a7d4ec18902a` | `9888de09b247b05b30d5874b9714bd7934e765c5f5d49bde2f894353fa5ccd07` |
| Actions attempt 1 | main ci #32，run_id=39754271411 | a5082c805b860b6cde24208d203aa36bba544e9f547d8f2845cb8e9caeeec2ed | f3704fd7a57e7951e8ede14eb8a8ca28d85029c2072f4ab54e3ebbe8910bf558 |
| [Actions attempt 2](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39754271411/attempts/2) | 同一 run_id=39754271411、构建号 32 的完整重跑 | a5082c805b860b6cde24208d203aa36bba544e9f547d8f2845cb8e9caeeec2ed | f3704fd7a57e7951e8ede14eb8a8ca28d85029c2072f4ab54e3ebbe8910bf558 |

- ① 与 ②：二进制和 tar.gz 均相同。原因：全部输入（源码、构建参数、工具链、平台、Git 元数据、归档规则）完全一致，构建可重复。
- ③：二进制和 tar.gz 均不同。原因：BUILD_NO 从 100 改为 101，构建号被注入二进制并写入 BUILD_INFO。
- ④ 与 ⑤：二进制相同，tar.gz 不同。原因：去掉 `--mtime=@0` 后文件修改时间进入 tar，每次压缩时间不同；二进制本身不受影响。
- ④ 的二进制与 ① 是否相同：不同。原因：临时修改了 Makefile，Git 工作区变脏，Go 自动记录 `vcs.modified=true` 并影响主模块版本信息，导致二进制变化（不是 tar 直接改写二进制）。
- 制品中应包含：版本号、构建号、运行 ID、源码提交 SHA、工具链版本，以及可核对的构建条件。
- 制品中不应包含：无必要的墙钟构建时间、随机值；也不能把未提交工作伪装为仅使用某一干净提交。
- release 为何在同一提交上再次编译而不直接发布 ci 主干制品：为了把正式版本号和该次发布的构建号注入程序；main 制品使用 `0.0.0-main` 及 CI 构建号，输入不同，不能直接重命名附件获得相同程序身份。
- 与"一次构建、多处部署"是否冲突：冲突。代价是增加耗时、环境漂移风险和重复验证成本；正式系统应先构建不可变制品再晋级部署，或把可变发布元数据移出程序本体。

**问题 4：发布与回退。**

数据：第 4 节两次 /version 及 [本次复测](evidence/local-validation.json)；v1.0.0 原附件哈希为 4a5cea3713d011296c8d6fe18269f82aed47b220a98e26fb246dee569b82416c，复测仍一致；问题 3 的额外重建/标签对照提供了“重新编译是否同一文件”的实际数据。

- v1.0.1 发布后发现缺陷如何处理：先把运行入口切回经校验的 v1.0.0 原附件，再经修复、PR、审批和检查后发布新版本号，并在旧版本说明中交代问题。
- 为何不应删除 v1.0.1 tag 再用同一版本号重发：同一版本号可能对应不同字节，下载缓存、校验值和故障追溯都会失去稳定含义。
- 从 v1.0.1 切回 v1.0.0 用的是哪个文件、是否重新构建：用的是从 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 解出的 `enroll-conflict` 二进制，没有重新构建。
- 两次 /version 输出说明：运行 v1.0.1 时 `{"version":"1.0.1","commit":"0e68161...","build_no":"3"}`；换回 v1.0.0 后 `{"version":"1.0.0","commit":"0e68161...","build_no":"1"}`。commit 相同，仅 version 和 build_no 变化，说明是制品切换而非源码回退。
- 从源码重新构建 v1.0.0 能否得到相同文件：不必然相同。问题 3 已测到，仅同提交上多了后续 tag 就会改变 Go 嵌入的模块版本；需固定标签集合并固定全部输入才能复现原二进制。
- 有人替换 v1.0.0 附件和 .sha256，trace.sh 能否发现：不能。trace.sh 只检查当前提供的附件与 .sha256 是否一致、以及提交是否属于 main，没有第三方可信的旧哈希做对照。
- 第 4 节记录的值能否发现：能。第 4 节发布时保存了 v1.0.0 的 sha256，与被替换后的新值比对即可发现不同字节；前提是该记录本身可信且未被一并篡改。本组未开启 GitHub Immutable releases（v1.0.0、v1.0.1 的 immutable 均为 false），附件和 tag 发布后仍可被改动。

## 7. 选做记录

| 选做 | 当前状态与证据 |
|---|---|
| O1 容器镜像 | 已完成。[release #4](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39943710686) 中的 image job 在 release 成功后下载并校验 v1.0.2 发布附件，以原二进制构建 linux/amd64 镜像，没有重新编译。已推送 `ghcr.io/qiyao-25/ops-lab1-9:v1.0.2`，实际 registry 引用为 `ghcr.io/qiyao-25/ops-lab1-9@sha256:2c769f8fc5ea1f5a08cd106bbc6e75f15c073e28dfb4a3545f80c6dfada35603`。工作流已按摘要运行并使用空 Docker 登录配置完成匿名拉取；主机架构和组内服务器复核见 [镜像证据](evidence/container-publication.json)、[最终验证](evidence/final-release-validation.json)。 |
| O2 缓存对反馈时间 | 已完成。实验前预期先提交；[PR #15](https://github.com/Qiyao-25/ops-lab1-9/pull/15) 已关闭且未合并，实验分支和运行记录保留；cache:true/false 各 5 次，全部样本见第 5 节和 [完整分析](docs/CACHE-RESULTS.md)。工作流计时中位数分别为 19 s、32 s，本次观测减少约 40.6%；解释包含 Go 构建缓存、计时波动及小样本局限。原历史混合事件的 40%～60% 结论撤回。 |
| O3 迁移到项目 | 已迁移到 [Python 词数项目](https://github.com/YiningZhang401/ops-lab1-o3)，详见 [迁移说明](docs/O3-MIGRATION.md)。目标项目有启用的 main 门禁、PR CI、main CI、v0.1.0 发布与非主干 tag 被拒的记录。 |

O1 镜像 job 与 release 放在同一工作流并用 needs 连接，是因为使用 GITHUB_TOKEN 创建 Release 不会再触发另一个基于该 Release 事件的工作流；见 [GitHub 工作流触发说明](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow)。GHCR 包的初始可见性可能为私有，仓库公开不能代替实际匿名拉取检查；见 [Container registry 官方说明](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)。本次实际 registry digest、主机架构和匿名拉取结果已记录；匿名拉取已成功，因此未另行修改包可见性。

## 8. 成员贡献

按已合入 PR 与 APPROVED 记录核对，每位成员至少一个本人发起并合入的 PR。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| Qiyao-25 | #1, #2, #3, #16 | #4, #5, #10, #12 |
| mohanlu | #4, #5 | #1, #2, #3, #8, #16 |
| sjettu | #6, #10, #11 | #7, #14 |
| YiningZhang401 | #7, #8, #14 | #6, #11 |
| gulu060719 | #12 | 无 |


## 9. PR #16 合并后的收尾验证

[PR #16](https://github.com/Qiyao-25/ops-lab1-9/pull/16) 于 2026-10-23 18:47:08+08:00 获得 mohanlu 审批，18:48:22+08:00 合入 main，对应提交 `b2547a988336ac54b1d0bbde4ff94ce373a1fb8b`。[main CI #36](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39941679659) 成功；本次验证使用该运行下载的原产物，版本为 `0.0.0-main`，构建号 36，Go 为 go1.26.9。

2026-10-23 19:02+08:00，在独立目录完成以下验证：校验和及 `scripts/trace.sh` 通过，提交位于 origin/main 历史；`GET /version` 返回相同的版本、提交和构建号；`GET /healthz` 返回 ok。使用相同二进制构建 linux/amd64 本地容器，容器版本输出与直接运行一致。临时 HTTP 服务已停止，原服务器仓库及其 LAB1.md 未覆盖。详见 [结构化记录](evidence/post-merge-validation.json) 和 [追溯日志](evidence/trace-main-ci36.log)。

| 项目 | SHA-256 |
|---|---|
| CI #36 二进制 | `a247d0972d3cb0bdca56c67abbaf496c30c5beee29fd5ed6614eb96739dfc971` |
| CI #36 归档 | `9a7a30d9862f8b7ab36ba5eaef1026c9509e20ec9b3c77e466302acc22b117ef` |

本地容器 image ID 仅标识本地镜像，不作为 GHCR registry digest。[缓存实验 PR #15](https://github.com/Qiyao-25/ops-lab1-9/pull/15) 已关闭且未合并，分支和运行记录保留；O2 结果已通过 PR #16 进入 main。


### v1.0.2 正式发布、镜像与再次回退

v1.0.2 以已审批合入的提交 `b2547a988336ac54b1d0bbde4ff94ce373a1fb8b` 为目标，发布时间为 2026-10-23 19:06:18+0800。tag 触发的 [release #4](https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39943710686) 中，release 和 image 两个 job 均成功。

| 项目 | v1.0.2 实测 |
|---|---|
| 关联 PR | [#16](https://github.com/Qiyao-25/ops-lab1-9/pull/16) |
| 构建号 / run_id | 4 / 39943710686 |
| Go / 平台 | go1.26.9 / linux/amd64 |
| 归档 SHA-256 | `fdafd5232600bf8b1da12238d4e04e14c897334d18c48a58d6776a703d4fe779` |
| 二进制 SHA-256 | `20c997e673bb6d3892fc2f3e18a6588bd6fa357a7ea3a0bc65143c13646bc2b1` |
| registry digest | `ghcr.io/qiyao-25/ops-lab1-9@sha256:2c769f8fc5ea1f5a08cd106bbc6e75f15c073e28dfb4a3545f80c6dfada35603` |

在本机不携带认证信息直接下载两个公开 Release 附件，HTTP 均为 200，哈希与 GitHub asset digest 一致，见 [匿名下载记录](evidence/anonymous-release-download.json)。服务器直连附件下载出现网络等待，因此通过 SFTP 将这份已验证的原附件传入独立克隆的 `dist/release/`；未重新打包。三个版本均通过校验和与 trace，v1.0.0 和 v1.0.1 的原附件哈希保持不变。运行验证于 2026-10-23 19:16:54+0800 开始，详见 [最终验证](evidence/final-release-validation.json)。

修订版 trace 输出：

```text
== 1. 校验和
enroll-conflict_1.0.2_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.2
build_no=4
run_id=39943710686
commit=b2547a988336ac54b1d0bbde4ff94ce373a1fb8b
go=go1.26.9
== 3. 提交 b2547a988336ac54b1d0bbde4ff94ce373a1fb8b
作者: Nuo Wang <2489397324@qq.com>
时间: 2026-10-23T18:48:22+08:00
标题: lab: complete reproducibility analysis and container verification (#16)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.2
版本号: 1.0.2    构建号: 4
提交页面（含关联的 PR）: https://github.com/Qiyao-25/ops-lab1-9/commit/b2547a988336ac54b1d0bbde4ff94ce373a1fb8b
构建它的运行: https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39943710686
```

修订版运行后切回原 v1.0.0，在同一仅监听本机的端口查询到：

```json
{"version":"1.0.2","commit":"b2547a988336ac54b1d0bbde4ff94ce373a1fb8b","build_no":"4","go":"go1.26.9"}
{"version":"1.0.0","commit":"0e68161b2924d975d970e3ca9c97b0935519f754","build_no":"1","go":"go1.26.9"}
```

本次回退不仅改变版本和构建号，源码提交也从 b2547a9 切回 0e68161；使用的是保留的原 v1.0.0 二进制，未重建旧版本。原要求的 v1.0.1 → v1.0.0 也再次复核，完整四次输出均在最终验证 JSON 中，临时服务均已停止。

镜像直接使用校验后的 v1.0.2 发布二进制。GitHub image job 的 `docker run --network none <registry digest> -version` 输出为：

```text
enroll-conflict version=1.0.2 commit=b2547a988336ac54b1d0bbde4ff94ce373a1fb8b build_no=4 go=go1.26.9
```

组内服务器拉取受到网络限制；按摘要运行和匿名拉取成功的证据来自上列 GitHub image job，服务器网络错误也保留在最终验证 JSON 中。 该 image job 记录的 runner 标签为 ubuntu-24.04，按 [GitHub 官方 runner 规格](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories) 可核对其运行主机为 Linux/x64，构建及运行的镜像平台为 linux/amd64。 GitHub 的匿名拉取同样成功，无需额外修改包可见性。镜像证据 artifact 保留 14 天；关键引用、版本和拉取结果同时写入仓库的 [镜像证据](evidence/container-publication.json)，长期制品以 Release 附件与 GHCR 镜像为准。
