# 实验 1 记录：第 9 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 9 组 |
| 仓库地址 | https://github.com/Qiyao-25/ops-lab1-9/ |
| 代码载体 | 统一示例  |
| runner 与语言版本 | `ubuntu-24.04`，Go 1.26.9 |
| 合并方式 | squash  |

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
| 规则集里启用的规则 | 规则集名称：opsRule，状态：Active，作用于默认分支 main。启用规则：① Restrict deletions；② Block force pushes；③ Require a pull request before merging，要求至少 1 名其他组员审批；④ Require status checks to pass，必需检查为 ci。 |
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

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | OCT 21, 2026, 00:08 GMT+8 | https://github.com/Qiyao-25/ops-lab1-9/pull/4 |
| 第一次 `ci` 失败（运行结束） | OCT 21, 2026, 00:08 GMT+8 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39454757987 |
| 推送修复提交 | OCT 21, 2026, 00:17 GMT+8 | https://github.com/Qiyao-25/ops-lab1-9/pull/4/changes/ce000f59958f4e8c2b3e32ccfefb1f343c0ef849 |
| `ci` 通过（运行结束） | OCT 21, 2026, 00:20 GMT+8 | https://github.com/Qiyao-25/ops-lab1-9/actions/runs/39456563807 |
| 合入 main | OCT 21, 2026, 00:34 GMT+8 | https://github.com/Qiyao-25/ops-lab1-9/pull/4 |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #4 | OCT 21, 2026, 00:03 GMT+8 | OCT 21, 2026, 00:34 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 34 小时 43 分 |
| #6 | OCT 21, 2026, 09:56 GMT+8 | OCT 22, 2026, 10:44 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 24 小时 50 分 |
| #8 | OCT 21, 2026, 11:33 GMT+8 | OCT 21, 2026, 23:20 GMT+8 | v1.0.0：OCT 22, 2026, 10:46 GMT+8 | 约 23 小时 13 分 |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| ci #1 | push（07885bd） | 44 s | 首次运行，Go 模块缓存未命中 |
| ci #2 | pull_request | 20 s | |
| ci #3 | push（3f02f9b） | 16 s | PR #1 合入后 |
| ci #4 | pull_request | 14 s | |
| ci #5 | push（3bd8d71） | 20 s | PR #2 合入后 |
| ci #6 | pull_request | 19 s | |
| ci #7 | push（fb5be4f） | 20 s | PR #3 合入后 |
| ci #8 | pull_request | 12 s | |
| ci #9 | pull_request（ce000f5） | 20 s | PR #4 第一轮，失败 |
| ci #10 | push（eee7836） | 20 s | PR #4 修复合入后 |
| ci #11 | pull_request | 20 s | |
| ci #12 | push（4913670） | 27 s | PR #5 合入后 |
| ci #13 | pull_request | 16 s | |
| ci #14 | pull_request（1f3b834） | 13 s | 第二轮：注入缺陷 |
| ci #15 | pull_request（169c944） | 20 s | 第二轮：补测试，失败 |
| ci #16 | pull_request（821d5d7） | 16 s | 第二轮：撤销缺陷 |
| ci #17 | pull_request | 12 s | |
| ci #18 | push（2b1c08c） | 19 s | PR #8 合入后 |
| ci #19 | push（cb6aefc） | 25 s | PR #7 合入后 |
| ci #20 | push（0e68161） | 23 s | PR #6 合入后，含 package 步骤 |
| ci #21 | pull_request | 14 s | |
| ci #22 | pull_request | 16 s | |
| ci #23 | push（5506cbe） | 17 s | PR #10 合入后 |
| ci #24 | pull_request | 15 s | |
| release #1 | push（0e68161） | 16 s | 发布 v1.0.0 |
| release #2 | push（73ee6af） | 8 s | 未合入分支上的 tag，被 gate 拒绝 |
| release #3 | push（0e68161） | 23 s | 发布 v1.0.1 |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

<填写>

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

<填写>

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | <填写> | <填写> |
| ② | 与 ① 完全相同，再构建一次 | <填写> | <填写> |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | <填写> | <填写> |
| ④ | 去掉 `--mtime=@0`，其余同 ① | <填写> | <填写> |
| ⑤ | 同 ④，间隔几秒后再构建一次 | <填写> | <填写> |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | <两次的值> | <两次的值> |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

<填写>

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

<填写>

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 已完成。基于 v1.0.0 发布制品构建镜像 `enroll-conflict:1.0.0`，镜像摘要（digest）`sha256:87b73ab26754df004dbebc26d799b3740fa0350a24d051d5e391b61db06ffc9d`，架构 `linux/amd64`（构建于 Windows x64 主机，Docker Desktop Linux engine）。`docker run --rm enroll-conflict:1.0.0 -version` 输出 `version=1.0.0 commit=0e68161b2924d975d970e3ca9c97b0935519f754 build_no=1 go=go1.26.9`，与 Release 附件一致。 |
| O2 缓存对反馈时间的影响 | 已完成。预期：`setup-go` 开启 `cache: true` 后，Go 模块依赖被缓存，后续运行的依赖下载步骤可跳过，整体 ci 时长缩短。数据：首次运行 ci #1（无缓存）耗时 44 s；开启缓存后 ci #3～#24 的时长集中在 12～27 s，中位数约 19 s。结论：缓存使首次冷启动后的 ci 反馈时间缩短约 40%～60%，对于"提交→测试"这一高频反馈环节效果明显；但缓存只加速依赖下载，对 `go vet`/`go test` 本身的耗时无影响，因此单次 ci 仍有十几秒的下限。 |
| O3 迁移到小组项目 | 迁移步骤已整理，待填入目标仓库链接。需复制：① `.github/workflows/ci.yml` 与 `.github/workflows/release.yml`；② `Makefile`、`scripts/trace.sh`、`Dockerfile`；③ 在 GitHub 仓库 Settings → Rules 中新建与 `opsRule` 等价的规则集（Restrict deletions、Block force pushes、Require PR before merging（≥1 审批）、Require status check `ci`）；④ 在 main 上打 `v1.0.0` tag 触发 release。目标仓库链接：<填写小组项目仓库地址> |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| Qiyao-25 | #1, #2, #3 | #4, #5, #10 |
| mohanlu | #4, #5 | #1, #2, #3, #8 |
| sjettu | #6, #10 | #7 |
| YiningZhang401 | #7, #8 | #6 |
| gulu060719 | #12 | 无 |
