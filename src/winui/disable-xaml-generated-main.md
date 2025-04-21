---
title: "如何禁用App.xaml自动生成的Main方法"
license: CC BY-SA 4.0 International
date: 2024-12-05
category: 
  - WinUI 3
tag:
  - C#
  - .NET
  - WinUI
---

App.xaml作为应用程序定义时会自动生成一个Program类和Main方法作为程序的入口点。但有的时候，我们并不希望程序一启动就直接执行App类的初始化，比如在你的程序主要目的是执行后台任务，而WinUI 3应用只作为其前端时。保持App类存在会很消耗内存（大约100MB），若是在不需要界面的时候能彻底关掉WinUI 3应用那就太好了😋。这就需要我们先关闭自动生成的Main方法。很简单：

```xml
<DefineConstants>DISABLE_XAML_GENERATED_MAIN</DefineConstants>
```

在项目属性中加入这一行即可。如果你已经有DefineConstants，可以使用分号分隔或这种形式：

```xml
<DefineConstants>$(DefineConstants);DISABLE_XAML_GENERATED_MAIN</DefineConstants>
```

然后我们需要自己搓一个Main方法。最简的Main方法长这样：

```csharp
public static void Main ( )
{
	WinRT.ComWrappersSupport.InitializeComWrappers();

	Application.Start((p) =>
	{
		SynchronizationContext.SetSynchronizationContext(
			new DispatcherQueueSynchronizationContext(DispatcherQueue.GetForCurrentThread())
		);
		_ = new App();
	});
}
```

完事！不过，既然决定使用自己搓的Main方法，肯定是为了实现一些高级用法吧。

## 单实例应用

我们可以实现单实例应用。借助WASDK的Activation相关API，可以很方便实现同时只会运行一个应用。

```csharp
// Credits: AIDevGallery by Microsoft

public static partial class Program
{
    public static void Main ( )
    {
        WinRT.ComWrappersSupport.InitializeComWrappers();

        if ( !DecideRedirection() )
            Application.Start((p) =>
            {
                SynchronizationContext.SetSynchronizationContext(
                    new DispatcherQueueSynchronizationContext(DispatcherQueue.GetForCurrentThread())
                );
                _ = new App();
            });
    }

    private static bool DecideRedirection ( )
    {
        bool isRedirect = false;
        AppActivationArguments args = AppInstance.GetCurrent().GetActivatedEventArgs();
        AppInstance keyInstance = AppInstance.FindOrRegisterForKey("WinUIBase.DemoApp");

        if ( !keyInstance.IsCurrent )
        {
            isRedirect = true;
            RedirectActivationTo(args, keyInstance);
        }

        return isRedirect;
    }

    [LibraryImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    private static partial bool SetForegroundWindow (IntPtr hWnd);

    private static void RedirectActivationTo (AppActivationArguments args, AppInstance keyInstance)
    {
        keyInstance.RedirectActivationToAsync(args).AsTask().Wait();
        Process process = Process.GetProcessById((int)keyInstance.ProcessId);
        SetForegroundWindow(process.MainWindowHandle);
    }
}
```

当你尝试在应用已经运行时再次打开时，已经打开的应用的窗口会自动唤起，而不会再次初始化App类。
