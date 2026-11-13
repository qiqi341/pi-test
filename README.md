# 实践课1：Pi Agent 安装与基础使用 实验报告
姓名: guoxiaqi
学号: 202430224023
GitHub: qiqi341
## 一、实验目的
- 在本地 Windows 环境下完成 Pi 客户端的部署，理解其“客户端+云端模型”的工作架构。
- 掌握通过 API Key 接入第三方大模型的方法，并测试基本对话。
- 熟悉模型切换（/model）与推理深度调节（/thinking）的差异。
- 掌握向 Agent 传递复杂上下文的技巧，包括 @ 引用本地代码与直接粘贴图片。
- 了解 Pi 的插件生态，完成 Extension 的安装与功能验证。
- （选做）利用 Agent 自动从 GitHub 拉取完整的 PPT Skill，并验证其被正确加载与调用，最终将项目托管至远程仓库。
## 二、实验环境
- 操作系统：Windows 11，Windows PowerShell 5.1
- Node.js：v24.x
- Git：2.47.1.windows.1
- Pi Agent：v1.5.0
- 大模型服务：DeepSeek（API Key方式），使用模型 deepseek-flash、deepseek-v4-pro
- 练习目录：D:\学习\系统设计与分析\pi-test
## 三、实验任务与结果
### 任务一：本地安装与初次启动
在 PowerShell 中通过官方脚本一键安装完成后，关掉了旧终端，并在新建的练习目录 pi-test 下唤出 Pi。
刚启动时，由于还未绑定任何模型，界面提示了 No models available，同时后台尝试下载 fd 和 ripgrep 时出现了超时警告。不过这不影响核心程序运行，底部状态栏清楚地显示了当前工作路径，说明客户端已就绪，等待接入“大脑”。
![Uploading S1.png…]()


### 任务二：配置 DeepSeek 模型并完成会话
前往 DeepSeek 开放平台生成 API Key 后，在 Pi 终端输入 /login，选择使用 API Key 登录，并选中了 DeepSeek 提供商。粘贴密钥回车后，提示配置成功。

随后发送了“Hello?”作为测试，模型立即给出了友好的回复。观察底部状态栏，确认当前挂载的模型是 deepseek-v4-pro。

### 任务三：动态切换模型与思考级别
输入 `/thinking` 打开Thinking Level菜单，将级别切换为 high。发送验证指令查看当前thinking level，确认 `PI_REASONING_LEVEL=high`，观察到底部状态栏的 thinking 标识发生了改变，这在处理复杂代码逻辑时能提供更深的思考过程。

接着通过 /model 命令调出选择器，将当前模型从 deepseek-v4-pro 切换成了响应更快的 deepseek-flash，切换后再次对话，验证依然畅通，后续任务截图右下角状态栏可见切换结果。


### 任务四：代码引用与多模态识图测试
代码引用：在目录下新建了 hello.py（包含加法函数），在输入框敲击 @ 唤出文件列表并选中它。发送提示词后，AI 准确解释了代码的作用，并算出了结果 5。

图片识别：截取了一张带有颜色和数字的小图到剪贴板，在终端按 Alt+V 粘贴，确认输入框出现了图片附件标识。发送“描述图片”的指令后，模型成功识别出了图中的信息，这证明 Pi 终端支持直接进行多模态交互


### 任务五：安装并使用一个插件
为了测试插件系统，从官方 Package 库找到了 pi-web-access（赋予大模型联网抓取网页的能力）。在终端执行 pi install npm:pi-web-access 安装后，完全退出并重新启动了 Pi。

再次启动时，确认 pi-web-access 已经成功挂载。随后让 AI 抓取一个网页的标题，AI 顺利调用了相关工具并返回了结果，证明插件运行正常。

### 任务六：让AI安装PPT Skill
安装：向 AI 下达了指令，要求从 GitHub 下载 anthropics/skills 下的 pptx 完整目录。由于 Windows 原生环境没有 Bash，AI 在报错后自动调整策略，启用了 PowerShell 进行下载。最终，56 个文件（含支持脚本和 .source.json 记录）被完整无缺地放置在了 D:\学习\系统设计与分析\pi-test\.pi\skills\pptx。


发现与调试：为了验证加载机制，彻底退出并重启了 Pi。启动时 [Skills] 区域清晰显示了 pptx。随后我输入 /skill:pptx 请说明这个 Skill 的名称、用途和所在目录，模型基于 Skill 说明给出了准确回答，并报出了正确的本地安装路径。

代码托管：最后，在本地终端使用 git init 将项目初始化，通过 git add 和 git commit 提交了所有更改，并将代码推送到了本人的 GitHub 远程仓库 qiqi341/pi-test 中，完成了作业的云端备份。

## 四、实验总结
1.架构解耦：客户端与模型独立
启动时的 No models available 仅代表尚未接入模型服务，并不代表客户端安装失败。Pi 本身作为本地终端载体，真正的推理能力、计费与具体型号均依赖所配置的 API 提供商，模型名必须以 /model 的实时列表为准。

2.策略权衡：模型选择与思考级别匹配
模型调用需在效率与深度间权衡。日常简单问答使用 deepseek-flash 即可满足；而在处理复杂解释或多工具联动任务时，切换至 deepseek-v4-pro 并开启 high 思考级别能获得更稳健的推理结果，但相应地会增加等待时间与 Token 消耗。

3.机制辨析：Extension 与 Skill 的本质差异
两者虽同为能力扩展，但机制不同：Extension 依赖 npm 安装，以工具形式被调用；而 Skill 则是放入 .pi/skills/ 的说明文件及支持脚本，需重启后被系统发现，并通过 /skill: 显式调用。因此，“安装成功、发现成功、调用成功、产出成功”构成了四个独立的验证层次，需分别进行确认。

4.环境适配：打破文档的“理想化”假设
实操证明，网络波动导致的 fd/ripgrep 下载失败不影响主体功能；同时，Skill 文档中假设的依赖环境（如 markitdown、Unix socket 等）在 Windows 本机未必成立。面对环境限制，应结合本地实际（如将执行环境从 Bash 切换为 PowerShell）进行绕行，而非死板照搬文档。

5.多模态交互与数据安全
@ 文件引用与剪贴板图片粘贴功能，赋予了 Agent “看代码、看图” 的多模态处理能力。与此同时，在涉及云端同步与代码共享时，必须严守安全底线，确保 API Key 截图脱敏，且坚决不将 auth.json 等包含敏感凭证的文件推送到远程仓库。
