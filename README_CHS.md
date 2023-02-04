# NvidiaTeslaP40forGaming
在 Windows 11 22H2 系统上配合 Intel 核显输出，在 nVidia Tesla P40 计算卡上运行游戏的配置方法。[English Ver.](https://github.com/toAlice/NvidiaTeslaP40forGaming/blob/main/README.md)

## 摘要
在后矿潮时代，淘汰显卡大量涌入二手市场，使包含计算中心显卡、工作站显卡在内的大量消费市场罕见或原本难以负担的淘汰设备与产品也以相对低廉的价格上架，吸引有好奇心的消费者购买。其中Tesla P40以相对近代的架构与不凡的性能受到不少有大显存需求的用户青睐。然而，收到谷歌搜索结果排序策略影响，同时其对中文的搜索能力也较差，加之如今许多教程文章的地位已被视频等媒介代替，许多已经购买此类计算的消费者都会因为只能找到滞后或不完整的教程、或是遇到了教程未提及的问题而导致无法按预期使用购买到的计算卡进行游戏、跑分等闲暇时间的活动，某种程度上造成了一些遗憾。本文主要记录了作者在 Windows 11 系统配合 Intel 平台进行针对解锁 nVidia Tesla P40 计算卡的游戏功能的配置步骤与过程中额外遇到的问题，并提出了一种相对粗暴的解决方法，测试结果达到比较接近完美但少数遗留缺陷几乎不影响使用的成果，以供将来的自己与其他可能看到此文的用户参考。

## 要求
1) 带核显的 Intel CPU （作者使用的处理器为 i9 13900k）。 AMD 产品没有测试，独立亮机卡的操作可能不同；
1) 带显示输出的主板（作者使用的主板为 MSI Z790 EDGE WIFI D4），同时也需要 BIOS 中有提供 Above 4G Decoding 选项；
1) 较新版本的系统（作者使用的系统为 Windows 11 22H2）。

## 标准步骤
1) 在 BIOS 中打开 Above 4G Decoding 选项。该选项在部分较新的平台中已经默认开启；
1) 从 nVidia [官方网站](https://www.nvidia.com/download/index.aspx)上下载适用于 Titan Xp 或其它中/高端 Pascal GPU 的 Studio 驱动。截止此文发表时 Studio 驱动最新的稳定版版本为 528.24，包含了对 Tesla P40 等计算卡的支持，不需要任何修改可以直接安装，且与 GeForce 系列适配的 Game Ready 驱动基本同步更新，可以带来最好的体验。下载完成后直接双击打开并按提示操作，推荐选择清洁安装。安装后若没有提示重启则也无需重启；
1) 打开 "regedit" 注册表编辑器，在左侧拦中打开 "HKEY_LOCAL_MACHINE\\SYSTEM\\" 文件夹，使用 Ctrl + F 快捷键搜索 "Tesla P40" 或你购买的显卡型号。若编辑器顶部地址栏中显示的路径为 "Computer\\HKEY_LOCAL_MACHINE\\SYSTEM\\ControlSet001\\Control\\Class\\{4d36e968-e325-11ce-bfc1-08002be10318}\\xxxx" (结尾的四个x为数字，比如0001)，则表明已经搜索到正确的目标，否则继续搜索；
1) 在该路径下将 AdapterType 的值由 2 修改为 0 或 1，对结果没有明显影响，亦可以选择直接删除；添加一个名为 GridLicensedFeatures 的 DWORD ，值为 7；将 FeatureScore 的值由 0xCF 修改为 0xD1（可选，作者使用的平台上没有明显区别）；添加一个名为 EnableMsHybrid 的 DWORD ，值为 1 （可选，会禁用NV VGX提供的虚拟显示器）；
1) 在设备管理器中禁用并重新启用 Tesla P40 显卡。

## 不足之处
1) 在完成上述步骤之后，作者的 P40 计算卡已经可以正常运行游戏与跑分软件。然而在此之后一旦关机、重启，系统都会在再次登录账号时死机、蓝屏，无法正常使用；
1) 系统无法正常从睡眠状态（特制 S3 ，其他状态未测试）中恢复，会直接或数十秒内进入死机、蓝屏状态；
1) 在拖动窗口与调整窗口尺寸的时候性能下降明显。

## 问题分析
1) 经多次调整实验， AdapterType 的值若在系统启动时为 2，相关功能则不会在启动时被解锁，问题1亦不会发生；
1) 问题2则在涉及到的几个选项内无法通过调整数值解决，除非在睡眠前临时禁用设备，但可能影响程序运行，得不偿失，且对使用的影响相对较小；
1) 问题3亦无法通过调整几个数值解决。

## 实现
1) 将本项目提供的 [logon.ps1](https://github.com/toAlice/NvidiaTeslaP40forGaming/blob/main/logon.ps1) 文件放在 "%APPDATA%\\Tesla Workaround\\"  或其它文件夹下。需要根据自己的实际情况修改 DevName 与 RegPath 的数值，比如 RegPath 的结尾有可能不为 0000 ，或者使用的显卡不为 Tesla P40 ；
1) 打开 "taskschd.msc" 任务计划程序，在左侧栏中用鼠标右键单击“任务计划程序（本地）”并选择“创建任务(R)“；
1) 在弹出的窗口中随意输入一个任务名称，因为无法更改建议三思而后行。选中窗口左下角的“使用最高权限运行”；
1) 打开“触发器”分页，选择新建，将“开始任务(G)”更改为“登录时”，点击确定关闭窗口；
1) 打开“操作”分页，选择新建，在“程序或脚本”处填写 "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe" （含或不含双引号均可），并在“添加参数（可选）”处填写 '-File "%APPDATA%\\Tesla Workaround\\logon.ps1"' (without apostrophes) （不含单引号），点击确定关闭窗口；
1) （可选）按需调整其他设置如延迟等；
1) 点击确定关闭窗口并保存任务；
1) 重新启动系统或手动执行任务，并打开GPU-Z查看显卡状态。

## 优势
1) 使用系统组件自动完成配置工作，无需每次手动配置；
2) 通过自动还原注册表中的相关项目，使得显卡、驱动等组件每次都能以nVidia预设的状态进入系统，避免了进入桌面时卡死、蓝屏等问题的发生；
3) 使用不受驱动安装器影响的系统组件完成配置，使得相关功能在显卡驱动在更新、重装、清空设置后都可以正常工作。

## 缺陷
1) 系统从休眠中唤醒后无法正常工作；
2) 将来部分版本的驱动可能无法直接安装。