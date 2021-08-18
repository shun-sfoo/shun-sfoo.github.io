# update_engine_client

## 源码分析

`const int kContinueRunning = -1;`

常量信号量 kContinueRunning 表示是否在初始化后持续运行

```cpp
const int kShowStatusRetryCount = 30;
const int kShowStatusRetryIntervalInSeconds = 2;
```

showStatus 请求在失败的情况下，会在 kShowStatusRetryIntervalInSeconds （2） 秒内重试 kShowStatusRetryCount（30） 次

## update_engine_client 类

属性：

```cpp

// Show the status of the update engine in stdout.
bool ShowStatus();

// Return whether we need to reboot. 0 if reboot is needed, 1 if an error
// occurred, 2 if no reboot is needed.
int GetNeedReboot();

// Main method that parses and triggers all the actions based on the passed
// flags. Returns the exit code of the program of kContinueRunning if it
// should not exit.
int ProcessFlags();


// Processes the flags and exits the program accordingly.
void ProcessFlagsAndExit();

// Copy of argc and argv passed to main().
int argc_;
char** argv_;

// Library-based client
unique_ptr<update_engine::UpdateEngineClient> client_;

// Pointers to handlers for cleanup
vector<unique_ptr<update_engine::StatusUpdateHandler>> handlers_;

```

集成了 brillo::Daemon ,此类

OnInit 初始化 daemon

内部类 ExitingStatusUpdateHandler : public update_engine::StatusUpdateHandler

内部类 WatchingStatusUpdateHandler 继承了 ExitingStatusUpdateHandler

内部类 UpdateWaitHandler 继承了 ExitingStatusUpdateHandler

### showStatus 方法

### ProcessFlags()

客户端 `update_engine_client` 中的方法 来源于他的父类 brillo::Daemon,
下面步骤中的都来自其中，是直接用父类的还是自己有实现，还要接着看。
一个很长的方法

1. 首先是用 `DEFINE_string` 和 `DEFINE_bool` 宏定义了很多语句
2. 样板初始命令
3. 判断参数合法性 确保 flag 有对应的 value `--flag=value`
4. 如果需要的话，更新状态
5. 更改移动网络下的更新状态
6. 显示移动网络下的更新状态
7. 修改/显示 队列提示 (cohort hint)
8. 更改 p2p 可用性设置
9. 显示回滚操作的可用性
10. 展示现在 p2p 可用性设置
11. 如果被请求， 首先更新目标信道
12. 展示当前信道和目标信道， 如果两者其一不存在就报错，log 显示当前信道，目标信道即将更新
    更新和回滚操作也不能同时进行， 如果有回滚请求，执行回滚。如果开启某项功能和禁止某项功能都
    存在并且相当报错，执行启用操作，执行禁止操作。
13. 如果需要的话，初始化更新检查
14. 最后的选项是排除掉相互冲突的的选项,然后分别执行

FLAG_status 查看状态
FLAG_follow 等待更新完整
FLAG_watch_for_updates
FLAG_reboot 重启
FLAG_Prev_version 回到上一个版本
FLAG_is_reboot_needed 重启

15.最后返回 0

### 方法

1. fn AttemptUpdate(s:app_version, s:omaha_url)

2. fn AttemptUpdateWithFlags(s:app_version, s:omaha_url, i: flags)

3. fn AttemptInstall(s: omaha_url, as: dlc_ids)

4. fn AttemptRollback(b: powerwash) -> (b: can_rollback)

5. fn ResetStatus()

6. fn SetDlcActiveValue(b: is_active, s: dlc_id)

7. fn GetStatusAdvanced() -> (ay: status)

8. fn RebootIfNeeded()

9. fn SetChannel(s: target_channel, b: is_powerwash_allowed)

10. fn GetChannel(b: get_current_channel) -> (s:channel)

11. fn SetCohortHint(s: cohort_hint)

12. fn SetP2PUpdatePermission(b: enabled)

13. fn GetP2PUpdatePermission() -> (b:enabled)

14. fn SetUpdateOverCellularPermission(b: allowed)

15. fn SetUpdateOverCellularTarget(s: taget_version, x: target_size)

16. fn GetUpdateOverCellularPermission() -> (b: allowed)

17. fn ToggleFeature(b: feature, b: enabled)

18. fn GetDurationSinceUpdate() -> (x: usec_wallclock)

19. Signal StatusUpdateAdvanced() -> (ay : status)

20. fn GetPrevVersion() -> (s: prev_version)

21. fn GetRollbackPartition() -> (s: rollback_partition_name)

22. fn GetLastAttemptError() -> (i: last_attempt_eror)
