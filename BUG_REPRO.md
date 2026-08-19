# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

多个调度请求并发分配同一作业单时全部返回成功。请修复并发状态更新，确保只能有一个请求完成分配。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-08
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-08.git
- parent SHA：91c61c66a515385fe181b97d501743011beec129

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-08.git bug-repro
cd bug-repro
git checkout --detach 91c61c66a515385fe181b97d501743011beec129
go test ./internal/workorder -run "^TestWorkOrder_ConcurrentAssignRace$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/workorder -run "^TestWorkOrder_ConcurrentAssignRace$" -count=1
--- FAIL: TestWorkOrder_ConcurrentAssignRace (0.01s)
    workorder_test.go:191: expected exactly 1 successful assign, got 5
FAIL
FAIL	portcoord/internal/workorder	0.011s
FAIL

```

stderr：

```text
warning: internal/workorder/workorder_test.go has type 100755, expected 100644
warning: internal/workorder/workorder_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/workorder -run "^TestWorkOrder_ConcurrentAssignRace$" -count=1
--- FAIL: TestWorkOrder_ConcurrentAssignRace (0.27s)
    workorder_test.go:191: expected exactly 1 successful assign, got 2
FAIL
FAIL	portcoord/internal/workorder	0.516s
FAIL

```

stderr：

```text
warning: internal/workorder/workorder_test.go has type 100755, expected 100644
warning: internal/workorder/workorder_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test -race ./internal/workorder -run ^TestWorkOrder_ConcurrentAssignRace$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
