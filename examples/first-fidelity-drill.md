# 第一次用 aws-emulator 演练 | First aws-emulator drill: install, fidelity check, telemetry off

把「先查保真度再信结果」走一遍，别装完就把本地通过当上线通过：

1. **省手装法**：确认 AWS CLI 在 2.35.0 以上，跑 `aws agent-toolkit add-skill --skill-name aws-emulator --region us-east-1`，让你的编程 agent 把模拟器起起来；不用 agent 的，走 Linux 原生包或 Docker 的服务器模式。
2. **先关遥测**：四选一先办一个——图省事用环境变量 `export EMULATOR_TELEMETRY=off`，要全机长期生效就改 `~/.aws/emulator/config.toml` 写 `[telemetry] enabled = false`。趁还没发请求，先关。
3. **指向本地端点**：按官方指南设环境变量把 CLI/SDK 指向 `https://127.0.0.1:9297`，跑一条你项目里最常用的命令，确认打在模拟器上、没出网、没产生 AWS 费用。
4. **查保真度分数**：去原仓 releases 页，找到你依赖的那几个服务在当前 release 的保真度分数；再到 `issues/things-that-do-not-work` 对一遍有没有你正好要用的操作。分数低或榜上有名的，先记下来，这些路径的结论不能只信本地。
5. **认 501**：故意调一个还没实现的操作，确认它返回 501 而不是假装成功——以后在 CI 里看到 501，先分清是代码错还是模拟器没实现，别白排查。
6. **IAM 结论打折**：挑一个靠 IAM 策略放行或拒绝的场景在本地跑一次，心里记牢：它不像真 AWS 那样评估 IAM 策略，本地放行的请求上线可能被拒，权限结论一律回真环境复核。
