---
layout: post
title:  "ViDi / How to Develop a Plugin for ViDi"
date:   2018-05-08 23:15:00
author: Alex Choi
categories: Deep-Learning
---

ViDi의 Plugin을 만드는 방법에 대하여 튜토리얼 형식으로 자세하게 알아보도록 하겠습니다.

본 Plugin의 개발 환경은 다음과 같습니다:

- ViDi : Ver. 2.1
- Windows 10 64bit
- Visual Studio : 2015(Ver. 14)

------
## 1.  프로젝트 생성하기
Visual Studio의 메뉴에서 `File > New > Project`를 클릭하여 새로운 프로젝트를 생성합니다.

아래 이미지와 같이 `New Project` 창에서 `Templates > Visual C# > Class Library`를 선택하고 프로젝트 생성 경로(Location)를 지정하고 `Project Name`을 정의합니다. 설명 편의상, 본 튜토리얼에서 `Project Name`은 `ViDi.CSharp.Example.Plugin`으로 하였습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/01.png">

[OK] 버튼을 클릭하여 새 프로젝트 생성을 완료합니다.

프로젝트 생성이 완료되면, 기본적으로 `Class1.cs`이라는 C# 소스파일이 추가됩니다. 역시 설명 편의상, `Plugin.cs`로 변경하였습니다. 소스파일 이름이 바뀌면 그에 따라 class 이름도 바뀝니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/02.png">

------
## 2. NuGet 패키지 설치하기
ViDi API를 기반으로 하는 모든 개발은 ViDi C# [NuGet 패키지](https://docs.microsoft.com/ko-kr/nuget/what-is-nuget) 설치로부터 시작합니다. ViDi NuGet 패키지를 설치하려면 Solution Explorer로부터 프로젝트명(`ViDi.CSharp.Example.Plugin`)을 마우스 오른쪽 버튼을 클릭하여 `Manage NuGet Packages...`를 선택합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/03.png">

NuGet Package Manager 창이 열리면, `Package source`를 추가하기 위하여 창의 우측에 있는 톱니 모양의 아이콘을 클릭합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/04.png">

`Options` 창이 열리면 Name으로 `vidi`를, Source로 `C:\Program Files\ViDi Systems\Cognex ViDi Suite 21\develop\nuget`를 선택합니다. 본 경로는, ViDi 2.1 설치 시 기본경로이며, Source 텍스트 박스에 직접 타이핑을 하지말고 [...] 버튼을 클릭하여 경로를 마우스로 클릭하여 지정해야 합니다. Source 경로 지정이 완료되면 [Update] 버튼을 클릭하여 ViDi NuGet 패키지 경로 설정을 완료합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/05.png">

이제 Package source를 `vidi`로 선택하고, Browse 탭에서 `vidi` 키워드로 검색하면 아래 이미지와 같이 4개의 `ViDi.NET.*` 패키지들이 검색됩니다. 각 패키지들을 차례로 클릭하여 이들을 모두 설치합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/06.png">

`ViDi.NET Package`들을 모두 설치 완료하면, 아래 이미지와 같이 `References`에 참조 항목들이 등록됩니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/07.png">

------
## 3.  Reference 등록
프로젝트에 다음 세가지의 참조 항목을 추가합니다 :

* (1) PresentationCore
* (2) PresentationFramework
* (3) WindowsBase

이들 참조 항목을 추가하려면 `References` 폴더에서 마우스 오른쪽 버튼을 클릭하고 컨텍스트 메뉴로부터 `Add Reference...`를 클릭합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/08.png">

`Reference Manager` 창이 열리면, 언급한 3개의 참조 항목을 체크한 후, [OK] 버튼을 클릭합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/09.png">

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/10.png">

참조 항목을 추가하면 아래 이미지와 같이 `Refereces`에 등록된 것을 확인할 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/11.png">

------
## 4.  코드 작성
주지하는 바와 같이, 플러그인이란, 응용 프로그램 내에서 동작하도록 되어 있는 응용 프로그램의 기능을 확장하는 어플리케이션입니다. 응용 프로그램 내에서 동작해야 하므로, 응용 프로그램에서 제공하는 프로그래밍 규약을 따라야 하는데, 이런 규약을 위해 ViDi는 `IPlugin interface`를 통해 함수를 추상화하였습니다.

우선, `IPlugin interface` 코드를 살펴보겠습니다.

<font size="5pt" face="Consolas" color="green"><strong>IPlugin interface</strong></font>
{% highlight csharp %}
namespace ViDi2.Training.UI
{
    //
    // Summary:
    //     Represents the common interface to UI plugins
    public interface IPlugin
    {
        //
        // Summary:
        //     Gets the plugin description
        string Description { get; }
        //
        // Summary:
        //     Gets the name of the plugin
        string Name { get; }
        //
        // Summary:
        //     Gets the plugin version
        int Version { get; }

        //
        // Summary:
        //     DeInitializes the plugin
        void DeInitialize();
        //
        // Summary:
        //     Initializes the plugin
        //
        // Parameters:
        //   context:
        //     the context
        void Initialize(IPluginContext context);
    }
}
{% endhighlight %}

위의 코드를 살펴보면, `IPlugin interface`에 `Description{}`, `Name{}`, `Version{}`, `DeInitialize{}`, `Initialize{}`라는 5개의 methods가 정의되어 있는 것을 확인할 수 있습니다.

즉, 본 interface를 상속받는 class에는 이들 5개의 methods가 모두 정의되어야 에러 없이 빌드가 가능합니다.

각 method는 그 이름만으로 어느 정도 역할을 알 수 있는데, 굳이 내용을 정리하자면 다음과 같습니다:

- Description : 플러그인에 대한 설명을 얻음
- Name : 플러그인의 이름을 얻음
- Version : 플러그인의 버전 정보를 얻음
- DeInitialize : 리소스 반납 등 플러그인 종료 시 관련 작업
- Initialize : 리소스 확보 등 플러그인의 초기화 관련 작업

그럼 이제 `Plugin.cs` 코드를 살펴보도록 하겠습니다. `using` 지시문을 통해 사용할 `namespace`를 정의합니다.

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using ViDi2.Training;
using ViDi2.Training.UI;
{% endhighlight %}

Line 8~9는 본 예제 플러그인 코딩에 필요한 ViDi `namespace` 사용을 정의합니다. `ViDi2.Training`은 `IWorkspace` (ViDi Workspace 인스턴스), `IStream` (ViDi Database Stream 인스턴스), `ITool` (ViDi Tool 인스턴스) 등에 대한 참조이며, `ViDi2.Training.UI`는 `IPlugin interface` 참조와 관련이 있습니다.

이제 ExamplePlugin class 내부를 살펴보도록 하겠습니다. ExamplePlugin class는 아래 코드와 가이 `IPlugin` interface를 상속받습니다:

{% highlight csharp %}
public class ExamplePlugin : IPlugin
{
    /// <summary>
    /// Context of other plugins, can be used to list other plugins.
    /// </summary>
    IPluginContext context;

    MenuItem pluginMenuItem;

    /// <summary>
    /// Gives a human readeable name of the plugin
    /// </summary>
    string IPlugin.Name { get { return "Example Plugin"; } }

    /// 중략
}
{% endhighlight %}

Line 6을 보면 context라는 IPluginContext 인스턴스가 정의되어 있는 것을 볼 수 있는데, IPluginContext의 이름에서 알 수 있듯 이것 또한 interface 입니다. 아래와 같이 IPluginContext 소스를 살펴보면, IPluginContext의 역할을 파악할 수 있습니다.

<font size="5pt" face="Consolas" color="green"><strong>IPluginContext.cs</strong></font>
{% highlight csharp %}
using System.Collections.Generic;

namespace ViDi2.Training.UI
{
    //
    // Summary:
    //     Represents the context within which plugins are exectued
    public interface IPluginContext
    {
        //
        // Summary:
        //     Gets a reference to the main window
        IMainWindow MainWindow { get; }
        //
        // Summary:
        //     Gets the list of loaded plugins to allow cross-plugin interaction
        ICollection<IPlugin> Plugins { get; }
    }
}
{% endhighlight %}

Summary 주석에서 설명되었듯, IPluginContext은 플러그인이 실행되는 지점의 context를 얻어오는 역할을 하며, 이 context를 통해 현재 ViDi의 Main Window를 얻어올 수 있으며, Main Window(`IMainWindow` interface)를 통해 현재의 ViDi Worksapce, Stream, Tool(Red/Blue/Green)에 접근을 할 수 있습니다. 따라서, `IPluginContext`은 ViDi 플러그인에서 항상 필요한 매우 중요한 interface라 할수 있습니다.

위의 코드에서 Line 17에 보시면 `ICollection` interface가 정의되어 있음을 볼 수 있습니다. 이것은, [Generic Collection](http://www.tutorialsteacher.com/csharp/csharp-generic-collections)을 다룰 수 있는 method들을 정의하고 있습니다. C#에서 Generic Collection은 `ArrayList`, `Queue`, `Stack`, `Hashtable` 등 어떠한 타입의 데이터라도 저장할 수 있는 집합을 의미합니다. `ICollection` interface의 역할은 ViDi의 모든 플러그인(`IPlugin`)을 얻어서 ViDi 내에서 플러그인 간 데이터를 공유하여 상호작용을 할 수 있도록 하는 것입니다.

다시 ExamplePlugin class 코드로 돌아가겠습니다.

{% highlight csharp %}
public class ExamplePlugin : IPlugin
{
    /// <summary>
    /// Context of other plugins, can be used to list other plugins.
    /// </summary>
    IPluginContext context;

    MenuItem pluginMenuItem;

    /// <summary>
    /// Gives a human readeable name of the plugin
    /// </summary>
    string IPlugin.Name { get { return "Example Plugin"; } }

    /// <summary>
    /// Gives a short description of the plugin
    /// </summary>
    string IPlugin.Description
    {
        get { return "Example of a ViDi Gui Plugin \n shows the current workspace/stream/tool"; }
    }
}
{% endhighlight %}

Line 8에 `MenuItem` class 인스턴스를 정의하는데, 이것은 간단히 말해서 Window의 메뉴 컨트롤과 관련이 있는 class입니다.

Line 13은 플러그인의 이름을 정의하며, Line 18 ~ 21에서는 플러그인에 대한 설명을 정의합니다.

이제 `IPlugin` interface에 정의되어 있는 메써드들을 구현한 부분에 대하여 하나씩 살펴보도록 하겠습니다.

{% highlight csharp %}
/// <summary>
/// Initilization method of the plugin. Called by the ViDi Gui directly after the plugin is loaded.
/// </summary>
void IPlugin.Initialize(IPluginContext context)
{
    this.context = context;

    var pluginContainerMenuItem =
        context.MainWindow.MainMenu.Items.OfType<MenuItem>().
        First(i => (string)i.Header == "Plugins");

    pluginMenuItem = new MenuItem()
    {
        Header = ((IPlugin)this).Name,
        IsEnabled = true,
        ToolTip = ((IPlugin)this).Description
    };

    pluginMenuItem.Click += (o, a) => { Run(); };

    pluginContainerMenuItem.Items.Add(pluginMenuItem);
}
{% endhighlight %}

Line 8~10은 작성하는 플러그인을 ViDi 메뉴 중 어느 곳에 위치시키느냐를 결정하는 부분입니다. 본 코드에서는 "Plugins"라는 이름으로 되어 있는 메뉴를 `pluginContainerMenuItem`라는 변수로 불러옵니다. 아래 이미지에서 보는 바와 같이 ViDi 메뉴 중 Plugins라는 메뉴 아이템에 작성하는 플러그인을 등록하겠다는 것입니다. 다른 의미로 보면, 작성하는 플러그인을 메뉴의 다른 아이템에도 등록이 가능하다는 것을 의미합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/12.png">

Line 12~17은 메뉴 아이템의 특성을 정의하는 코드입니다.

해당 컨트롤은 `System.Windows.Controls.HeaderedItemsControl.cs`에 정의되어 있으며, 대부분은 default 값을 가지고 있으므로 별도의 Property를 정의하지 않아도 동작은 하지만, 플러그인의 특성을 정의하기 위해 필요한 코드입니다. Line 14는 플러그인의 Header를 정의하며, Line 15는 플러그인의 활성화 여부, Line 15는 Tool Tip으로 표시할 플러그인 간단 설명을 정의하는 코드입니다.

Line 19에서는 본 플러그인의 클릭 시 어떤 동작을 해야할지를 정의하는 코드로써, 클릭을 하면 `Run()` 메써드를 호출하도록 하고 있습니다. 참고로, `=>` 는 [Lambda Operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator)이며, `(o, a) => { Run(); }` 는 Lambda 표현식 형태로 Click Event에 대하여 Event Lister를 추가하는 코드입니다. 이에 대한 메써드는 `System.Windows.RoutedEventHandler.cs`에 정의되어 있습니다:

<font size="5pt" face="Consolas" color="green"><strong>RoutedEventHandler.cs</strong></font>
{% highlight csharp %}
namespace System.Windows
{
    public delegate void RoutedEventHandler(object sender, RoutedEventArgs e);
}
{% endhighlight %}

즉, `(o, a)` 중 `o`는 object를, `a`는 RoutedEventArgs를 상징합니다.

Line 21에서는 정의된 플러그인 메뉴 아이템을 컨테이너 메뉴 아이템에 추가합니다.

이번에는 `DeInitialize()` 메써드를 살펴보도록 하겠습니다.

{% highlight csharp %}
void IPlugin.DeInitialize()
{
    var pluginContainerMenuItem =
        context.MainWindow.MainMenu.Items.OfType<MenuItem>().
        First(i => (string)i.Header == "Plugins");

    pluginContainerMenuItem.Items.Remove(pluginMenuItem);
}
{% endhighlight %}

`DeInitialize()` 메써드는 ViDi 종료 시 호출되며, ViDi의 메뉴 중 Plugins 메뉴로부터 본 플러그인을 제거하는 작업을 수행합니다.

마지막으로 플러그인이 실제 동작하는 부분인 `Run()` 메써드를 살펴보겠습니다.

{% highlight csharp %}
void Run()
{
    string message;
    IWorkspace workspace = context.MainWindow.WorkspaceBrowser.CurrentWorkspace;
    if (workspace == null)
        message = "No current workspace";
    else
    {
        message = "Current workspace is : " + workspace.Name + "\n";
        IStream stream = context.MainWindow.ToolChain.Stream;

        if (stream != null)
        {
            message += "Current stream is : " + stream.Name + "\n";

            ITool tool = context.MainWindow.ToolChain.Tool;
            if (tool != null)
            {
                message += "Current tool is : " + tool.Name + " (" + tool.Type.ToString() + ")";
            }
        }
    }

    MessageBox.Show(message, "Example Plugin", MessageBoxButton.OK, MessageBoxImage.Information);
}
{% endhighlight %}

자세한 코드 설명 이전에 본 플러그인은 단순히 현재 열려 있는 ViDi Workspace의 이름과 현재 선택되어 있는 Stream 이름, 그리고 현재 선택되어 있는 Tool의 이름을 표시해 주는 기능을 하고 있습니다.

Line 4에서 ViDi Workspace 인스턴스를 얻고, 얻어진 Workspace가 유효하다면(Line 7), 현재 Workspace 이름을 `message` 변수에 저장(Line 9)합니다. `Stream` 및 `Tool`에 대해서도 유사하게 처리를 하며(Line 10~22), Line 24에서 Workspace, Stream, Tool의 메시지 박스로 표시합니다.

------
## 5.  플러그인 등록하기
위의 코드를 작성하고 무사히 빌드를 마치면, 프로젝트 출력 경로에 `ViDi.CSharp.Example.Plugin.dll`이 생성됩니다. 이 파일을 `C:\Program Files\ViDi Systems\Cognex ViDi Suite 21\plugins` 경로(ViDi가 기본경로에 설치되어 있다고 가정)에 복사하고, ViDi Suite GUI를 실행합니다.

플러그인 DLL 파일이 ViDi Plugins 경로에 존재한다고 해서 ViDi가 플러그인을 자동으로 불러오지는 않습니다 (한번 등록 후에는 자동으로 불러옴).

플러그인을 등록하기 위해서 `ViDi Menu > Plugins > Manage Plugins ...`를 클릭합니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/13.png">

다음과 같이 Plugin Manager 창이 열리면, [Add] 버튼을 클릭하고,

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/14.png">

`ViDi.CSharp.Example.Plugin.dll` 파일을 선택하면,

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/15.png">

아래 이미지와 같이 Plugin Manager에 Example Plugin 등록 완료된 메시지가 나오며,

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/16.png">

ViDi의 Plugins 메뉴에 작성한 플러그인이 등록된 것을 확인할 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/17.png">

만약 현재 ViDi에 <font face="Consolas">"Basic 04 - Dial Inspection"</font>이라는 이름의 Workspace가 하나 열려있고 `my stream`이라는 Stream이 선택되어 있으며, `My Red`라는 Red Tool이 선택되어 있는 경우, 플러그인을 실행하면 다음과 같은 메시지 박스가 나타납니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/18.png">

------
## 6.  Visual Studio에 플러그인 템플릿 등록하기
플러그인을 제작할 때마다 기본적으로 필요한 부분을 코딩하는 것은 무척 번거롭습니다. 따라서, 어느 정도 뼈대 작업을 한 ViDi 플러그인 제작을 위한 Visual Studio 템플릿을 공유하고자 합니다.

첨부된 압축 파일을 다운로드 한 후,

`C:\Users\{USERNAME}\Documents\Visual Studio 2015\Templates\ProjectTemplates` 경로에 저장합니다.

 [ViDi.CSharp.Example.Plugin.zip 다운로드](http://cinema4dr12.tistory.com/attachment/cfile10.uf@99163E365AF472DF36D1B2.zip)

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/19.png">

Visual Studio의 `File > New > Project`를 클릭하시면, 아래와 같이 `ViDi.CSharp.Example.Plugin` 템플릿이 있는 것을 확인할 수 있으며, 프로젝트를 생성하고 빌드 후 플러그인 제작을 위한 코딩 작업을 하시면 됩니다.

<img src="{{ site.baseurl }}/assets/posts/2018-05-08-PluginDevelopment/20.png">
