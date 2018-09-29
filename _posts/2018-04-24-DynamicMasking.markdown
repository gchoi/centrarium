---
layout: post
title:  "How to Set up Dynamic Masking Training Workspace"
date:   2018-04-24 16:32:00
author: Alex Choi
categories: Deep-Learning
---

일반적으로 ViDi에서 Masking 시, Edit Mask 기능을 이용하여 이미지에 직접 Masking 영역을 Painting할 수도 있지만 이미 만들어 놓은 Masking Image(Single Channel + 8-bit Depth)를 Import하여 적용할 수도 있습니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/01.png">
<br/>

만약 이미 다른 만들어 놓은 다수의 Masking Image를 해당 이미지에 각각 적용하려면 위의 방법으로 일일이 하기에는 시간이 많이 소요됩니다. 실제로, L사의 경우 자체 솔루션으로 Masking Image Sequence를 만들고 이들을 동적으로 해당 이미지에 쭈~욱 적용하는 솔루션을 요구한 바 있으며, 당시에는 ViDi C++ API를 통해 솔루션을 만들고 밤늦게 석방되었는데, 아쉽게도 코드를 가지고 나올 수 없어 눈물을 머금고 C#으로 다시 개발하였는데 거의 기억이 나지 않아 고생했습니다 (나이가 드니 점점 기억력이... 혹시라도 저한테 돈빌려 가신 분은 갚아주세요).

간단한 튜토리얼 형식으로 Dynamic Masking Image 합성하는 코드를 작성하고 테스트하는 방법에 대하여 알아보도록 하겠습니다.

------
## ViDi Workspace
우선 본 튜토리얼을 통해 만들고자 하는 결과물은 아래 이미지와 같이, Green Tool이 생성되어 있고, 3개의 배터리와 각 배터리에 대하여 미리 준비된 Masking Image가 합성된 ViDi Workspace입니다 (Advanced Tutorial 4 - Dynamical Masking을 참고하시면 됩니다).

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/02.png">
<br/>

------
## Image Data Set
시간을 절약하기 위해 미리 준비한 Base Image와 Mask Image Set을 공유합니다.

- Base Image Set 다운로드:  [base_images.zip](http://cinema4dr12.tistory.com/attachment/cfile10.uf@9909EC3D5ADEE54521D883.zip)
- Mask Image Set 다운로드:  [mask_images.zip](http://cinema4dr12.tistory.com/attachment/cfile7.uf@997C7F3C5ADEE545141CD2.zip)

다운받은 이미지 세트를 아무 경로에 해제하는 것은 자유이나 본 튜토리얼을 좀 더 쉽게 이해하기 위해, Base Image Set와 Mask Image Set 모두 `D:/temp/`에 압축해제 합니다.

따라서, Base Image들이 저장되어 있는 경로는, `D:/temp/base_images`이며, Mask Image들이 저장되어 있는 경로는, `D:/temp/mask_images`입니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/03.png">
<br/>

위의 이미지와 같이, 각 3장의 Base Image들과 Mask Image들을 준비하였습니다. 특히, Base Image 경로 내 이미지 파일의 이름이 `Cat{d}-XXXXX.png` 형식으로 되어 있음을 주목하시기 바랍니다. Base Image 파일 이름은 차후 설명하겠지만 순차적으로 Mask Image를 적용하는 중요한 Filtering 옵션으로 활용됩니다.

------
## Code Structure
코드의 전체구조는 다음과 같습니다.

1. 사용자 경로 설정
2. 생성할 ViDi Workspace 경로 설정
3. ViDi Workspace 생성
4. Stream 생성
5. Tool 생성
6. Base Image 경로로부터 모든 Base Image 불러오기
7. Database Image Process
8. Mask Image 경로로부터 모든 Mask Image 불러오기
9. 각 Base Image에 해당 Mask Image 합성
10. ViDi Workspace 닫기

그러면, 이제 위에 나열된 항목들에 해당하는 내용을 코드를 통해 알아보도록 하겠습니다.

#### 1. 사용자 경로 설정
사용자가 설정할 경로는 3가지 입니다:

- `strWorkspacePath` : ViDi Workspace를 생성할 경로
- `strBaseImagePath` : Base Image가 저장되어 있는 경로
- `strMaskImagePath` : Mask Image가 저장되어 있는 경로

이들 경로들은 다음과 코드로 정의하였습니다- 절대적인 것은 아니며 사용자의 상황에 따라 수정이 가능합니다.

{% highlight csharp %}
// 1. define paths
//////////////////////////////////////////////////////////////////////
// USER DEFINITIONS
//////////////////////////////////////////////////////////////////////
string strWorkspacePath = @"D:\temp\workspace";
string strBaseImagePath = @"D:\temp\base_images\";
string sttMaskImagePath = @"D:\temp\mask_images\";
//////////////////////////////////////////////////////////////////////
{% endhighlight %}

#### 2. 생성한 ViDi Workspace 경로 설정
{% highlight csharp %}
// 2. set workspace path
ViDi2.Training.IControl control = null;

var workspaceDirectory = new ViDi2.Training.Local.WorkspaceDirectory()
{
    Path = strWorkspacePath
};

var libraryAccess = new ViDi2.Training.Local.LibraryAccess(workspaceDirectory);

control = new ViDi2.Training.Local.Control(libraryAccess, GpuMode.SingleDevicePerTool, new List<int>());
{% endhighlight %}

위의 코드는 Training Control을 생성한 후, Workspace Directory를 생성하여 Control에 이를 등록하는 과정입니다. Code의 각 라인에 대한 상세한 내용은 API 문서를 참고하시기 바랍니다.

#### 3. ViDi Workspace 생성
{% highlight csharp %}
// 3. create a new workspace
var workspace = control.Workspaces.Add("workspace");
{% endhighlight %}

API를 통해 Workspace를 새로 생성할 수도 있으며, 기존 Workspace를 열어 기능을 추가할 수도 있습니다. 위의 코드의 경우는 새로운 Workspace를 생성하는 경우입니다.

#### 4. Stream 생성
{% highlight csharp %}
// 4. create a stream
var stream = workspace.Streams.Add("default");
{% endhighlight %}

ViDi Stream은 Database 세트를 구성하는 컨테이너입니다. 위의 코드는 Stream의 이름을 설정하는 코드입니다. `default`라는 이름의 Stream을 추가하였습니다.

#### 5. Tool 생성
{% highlight csharp %}
// 5. create a green tool at the root of the toolchain
var green = stream.Tools.Add("classify", ToolType.Green) as IGreenTool;
{% endhighlight %}

아시다시피, ViDi에는 Blue, Red, Green Tool이 있는데, 위의 코드를 참고하여 자유롭게 Toolchain을 구성할 수 있습니다.

#### 6. Base Image 경로로부터 모든 Base Image 불러오기
{% highlight csharp %}
// 6. load base images from local directory
var ext = new List<string> { ".jpg", ".bmp", ".png" };
var myImagesFiles = Directory.GetFiles(strBaseImagePath, "*.*", SearchOption.TopDirectoryOnly).Where(s => ext.Any(e => s.EndsWith(e)));

foreach (var file in myImagesFiles)
{
    using (var image = new FormsImage(file))
    {
        stream.AddImage(image, Path.GetFileName(file));
    }
}
{% endhighlight %}

Line 1~2에서 불러올 이미지 파일의 포맷을 설정하고 `myImageFiles` 변수에 해당 경로의 모든 이미지를 불러옵니다. Line 5~11에서 이미지를 하나씩 불러서 생성한 Stream에 추가합니다.

#### 7. Database Image Process
{% highlight csharp %}
// 7. process
green.Database.Process();

green.Wait();
{% endhighlight %}

이제 Stream에 저장된 이미지들을 프로세싱합니다. Line 4에서는 프로세싱을 마칠 때까지 다른 작업이 실행되지 않도록 합니다.

#### 8. Mask Image 경로로부터 모든 Mask Image 불러오기
{% highlight csharp %}
// 8. load mask images from local directory
System.Console.WriteLine();
System.Console.WriteLine("Image composition started....");
var myMaskImage = Directory.GetFiles(strMaskImagePath, "*.*", SearchOption.TopDirectoryOnly).Where(s => ext.Any(e => s.EndsWith(e)));
{% endhighlight %}

Line 4에서 Mask Image 경로 내의 모든 이미지를 불러 `myMaskImage`에 정보를 저장합니다.

#### 9. 각 Base Image에 해당 Mask Image 합성
{% highlight csharp %}
int nIdx = 0;
foreach (var file in myMaskImage)
{
    nIdx++;
    using (var image = new FormsImage(file))
    {
        // 9. set mask image
        string strFilter = string.Format("\'Cat{0:00}\'", nIdx);

        green.Database.SetViewMask(strFilter, image);

        System.Console.WriteLine("  {0}-th image composed", nIdx);
    }
}
{% endhighlight %}

Line 2~14는 8번 과정에서 불러온 Mask Image를 하나씩 불러서 Base Image와 합성하는데, Line 9가 합성을 위한 핵심 코드라고 보시면 됩니다. `SetViewMask()` 메써드의 첫번째 Argument인 `strFilter`는 특정 View를 선택하는 Filter이며 아래 이미지와 같이 ViDi의 Display Filter에 입력되는 형식과 동일합니다. 두번째 Argument인 `image`는 현재 선택된 Mask Image입니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/04.png">
<br/>

Line 8에 변수 `strFilter`에 `foreach` Loop문을 통해 순차적으로 출력되는 결과는,

```
'Cat01'
'Cat02'
'Cat03'
```

입니다. 즉, 첫번째 Mask Image `Mask-01.png`는 '`Cat01`'로 필터링 된 View인 Base Image(`Cat01-00000_0.png`)에 합성되며, 두번째 Mask Image `Mask-02.png`는 '`Cat02`'로 필터링 된 View인 Base Image(`Cat02-00010_0.png`)에 합성되며, 세번째 Mask Image `Mask-03.png`는 '`Cat03`'으로 필터링 된 View인 Base Image(`Cat03-00005_0.png`)에 합성됩니다.

#### 10. ViDi Workspace 닫기
Workspace 설정 및 저장이 완료되면 열어두었던 모든 Workspace를 닫고, control도 메모리 리소스로부터 해제합니다.

{% highlight csharp %}
workspace.Save();

// 10. close the workspaces
foreach (var w in control.Workspaces)
    if (w.IsOpen) w.Close();

control.Dispose();
{% endhighlight %}

------
## Workspace 결과 확인

이제 앞서 지정한 Workspace 출력 경로에 Workspace가 생성되었음을 확인할 수 있으며,

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/05.png">
<br/>

해당 Workspace를 ViDi GUI로 열면 다음과 같이 Mask Image가 합성되어 있음을 확인할 수 있습니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-24-DynamicMasking/06.png">
<br/>

------
## 전체 코드

마지막으로 작성한 C# 코드를 소개로 본 튜토리얼을 마무리합니다.

{% highlight csharp %}
using System.IO;
using System.Linq;
using System.Collections.Generic;
using ViDi2;

/// <summary>
/// @author: Alex Choi
/// @date: 2018.04.22
/// @description: This is a ViDi API example for composing dynamic mask
/// </summary>
///
namespace Example.Training.Console.Mask
{
    using ViDi2.Training;

    class Program
    {
        static void Main(string[] args)
        {
            // 1. define paths
            //////////////////////////////////////////////////////////////////////
            // USER DEFINITIONS
            //////////////////////////////////////////////////////////////////////
            string strWorkspacePath = @"D:\temp\workspace";
            string strBaseImagePath = @"D:\temp\base_images\";
            string strMaskImagePath = @"D:\temp\mask_images\";
            //////////////////////////////////////////////////////////////////////

            // 2. set workspace path
            ViDi2.Training.IControl control = null;

            var workspaceDirectory = new ViDi2.Training.Local.WorkspaceDirectory()
            {
                Path = strWorkspacePath
            };

            var libraryAccess = new ViDi2.Training.Local.LibraryAccess(workspaceDirectory);

            control = new ViDi2.Training.Local.Control(libraryAccess, GpuMode.SingleDevicePerTool, new List<int>());

            // 3. create a new workspace
            /**
             * 워크 스페이스를 새로 생성하여 사용하는 경우에는 Add 함수 사용
             * 이미 생성된 워크스페이를 오픈 할때는 control.Workspaces["워크스페이스명"].Open();
             * */
            var workspace = control.Workspaces.Add("workspace");

            // 4. create a stream
            var stream = workspace.Streams.Add("default");

            // 5. create a green tool at the root of the toolchain
            var green = stream.Tools.Add("classify", ToolType.Green) as IGreenTool;

            // 6. load base images from local directory
            var ext = new List<string> { ".jpg", ".bmp", ".png" };
            var myImagesFiles = Directory.GetFiles(strBaseImagePath, "*.*", SearchOption.TopDirectoryOnly).Where(s => ext.Any(e => s.EndsWith(e)));

            foreach (var file in myImagesFiles)
            {
                using (var image = new FormsImage(file))
                {
                    stream.AddImage(image, Path.GetFileName(file));
                }
            }

            // 7. process
            green.Database.Process();

            green.Wait();

            // 8. load mask images from local directory
            System.Console.WriteLine();
            System.Console.WriteLine("Image composition started....");
            var myMaskImage = Directory.GetFiles(strMaskImagePath, "*.*", SearchOption.TopDirectoryOnly).Where(s => ext.Any(e => s.EndsWith(e)));

            int nIdx = 0;
            foreach (var file in myMaskImage)
            {
                nIdx++;
                using (var image = new FormsImage(file))
                {
                    // 9. set mask image
                    string strFilter = string.Format("\'Cat{0:00}\'", nIdx);

                    green.Database.SetViewMask(strFilter, image);

                    System.Console.WriteLine("  {0}-th image composed", nIdx);
                }
            }

            workspace.Save();

            // 10. close the workspaces
            foreach (var w in control.Workspaces)
                if (w.IsOpen) w.Close();

            control.Dispose();
        }

    }
}
{% endhighlight %}
