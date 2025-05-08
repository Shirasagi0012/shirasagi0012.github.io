# 在 Windows On ARM 设备上编译并运行 ARM64 架构的 osu!lazer

osu!lazer 是著名音游 osu! 的下一代开源客户端，它经过彻底重构，从 .NET Framework 迁移到现代的 .NET Core，并且借助 .NET Core 的强大跨平台能力实现了一套代码库在多平台（Android、iOS/iPadOS、MacOS、Linux、Windows）和多架构（x64、x86、ARM64、LoongArch64）上运行。不过，由于反作弊系统的原因，ppy选择只支持了其中一部分平台。但是 osu!lazer 是开源的，这意味着我们可以手动编译一份能在ARM64架构上运行的 osu!lazer！在这篇教程中，我会阐述编译过程你会遇到的坑，以及该怎么解决。

## 事先准备

你需要一台安装有 .NET 8 SDK 和 Visual Studio 2022（不是必须，但接下来操作会以VS2022为例） 的设备。不限架构，x64\ARM64均可。

从 [Un4seen Bass | ARM64 Windows](https://www.un4seen.com/forum/?topic=20601)处下载最新的 ARM64 架构编译的 Bass 音频库。

一台能开机且GPU正常工作的ARM64架构的Windows设备。

## 编译

编译过程很简单。从 [GitHub | ppy/osu](https://github.com/ppy/osu) 将最新的 osu!lazer 源代码 clone 下来，然后打开目录下的`osu.Desktop.slnf`或`osu.sln`。接下来需要进行一点操作，默认是使用Any CPU配置编译的，需要在配置管理器里创建一个ARM64配置和x64配置。选择启动项目为`osu.Desktop`，然后就可以开始以`Release`模式编译了。需要注意的是，你不仅要编译一份ARM64的 osu!lazer，还需要一份x64的。

如果没有什么意外，编译完成后产物会在`.\osu.Desktop\bin\ARM64\Release\net8.0`和`.\osu.Desktop\bin\x64\Release\net8.0`下。

## 运行

首先，你需要把刚才下载的ARM64 Bass dll手动复制到`osu.exe`所在的目录。这还没完，如果你尝试执行，只会在log里看见缺少`veldrid-spirv`库，这个库并没有Windows ARM64的二进制构建，手动构建也非常麻烦。它主要用于编译着色器，然后缓存下来，因此我们只需要先运行一遍x64架构的osu!lazer，就会自动生成着色器缓存，之后启动就不会用到这个库了。如果你无法启动x64架构的osu!lazer，这通常是因为 osu!lazer 优先使用 Vulkan API，并且fallback逻辑有点诡异导致的，你可以在`%appdata%\osu\framework.ini`里将`Renderer=`一栏改成`Direct3D11`或`Deferred_Direct3D11`，强制使用 DirectX 11 运行。

进行了刚才的一系列操作，应该能正常启动刚才编译的 ARM64 架构的 osu!lazer 了。osu!lazer 在 Windows on ARM 设备上运行良好，即使在 Snapdragon 8cx Gen 1/2 这样的电子垃圾上，不是太复杂的谱面甚至能达到 400+fps@2K 的帧率表现（但在Storyboard炫技的`world.execute(me); - Mili`的最高强度片段还是只有20fps :( ，作为对比，Ryzen 7940HS和它的Radeon 780M在同样的场景可以跑到30fps，但780M的理论性能数倍于Adreno 685）。骁龙855/860设备应该也能跑到 ~200fps@2K，这可比 osu!lazer 的安卓版高到不知道哪里去了。

## 优化建议

在任务管理器里，详细信息，osu!.exe，设置相关性，将osu!.exe分配到大核心上（对于Snapdragon 855/860/8cx Gen 1/2这样小核极为弱鸡的设备），这通常能提供20%左右的性能提升

如果你发现FPS计数器虽然高，但体感掉帧非常严重，这是由于 DWM强制垂直同步 和 帧生成时间与显示器刷新率不匹配 导致的。前者理论上可以通过开启全屏独占解决，但是高通设备似乎很难开启全屏独占。后者通常会发生在没有自适应垂直同步的设备上，但很可惜几乎所有ARM64设备都不支持诸如G-Sync和Freesync这样的自适应垂直同步方案（ARM64设备的游戏体验没老黄下场真不行QAQ）。因此解决办法是开启垂直同步或者主动限制帧数到 2x 刷新率，以主动控制帧生成时间以提供和显示器刷新率匹配的稳定帧节奏。

如果你准备使用触摸屏游玩，不要忘了关闭Windows的三指/四指手势，它们会导致断触。你还可以去玩玩模仿maimai的Sentakki模组，不过需要注意的是，初次游玩该Ruleset同样需要编译着色器，因此你得先拿x64版跑一遍。

## 总结

由于osu!lazer是一个开源项目，以及.NET Core的强大跨平台能力，使osu!lazer在并不受官方支持的平台上运行成为了可能，并且事实上运行起来非常良好，性能表现达到同级别x86设备的表现。不只是ARM64，也有人成功在LoongArch上运行了osu!lazer。但是，原生库和反作弊系统依旧会成为跨平台运行的一大阻碍。