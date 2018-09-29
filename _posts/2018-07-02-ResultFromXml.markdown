---
layout: post
title:  "ViDi / How to Get Results from XML Data"
date:   2018-07-02 16:17:00
author: Alex Choi
categories: Deep-Learning
---

## 개발환경
본 샘플 프로그램 개발환경 및 실행환경은 다음과 같습니다.

- OS : Windows 7 x64 / Windows 10 x64
- ViDi : Ver.2.1
- IDE : Visual Studio 2015

------
## 샘플 프로그램 다운로드 및 구성
샘플 프로그램의 경로 구성은 다음과 같습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-02-ResultFromXml/01.png">

------
## 샘플 프로그램 설명

<br/>
#### 전체적인 플로우
샘플 프로그램의 전체적인 순서는 다음과 같습니다:

1. Result 폴더가 존재 여부를 체크한 후 존재하지 않을 경우 이를 생성합니다.
2. 지정된 ViDi Workspace 경로로부터 ViDi Workspace를 엽니다.
3. 지정된 이미지 경로 내의 모든 이미지를 불러옵니다.
4. 각 이미지에 대한 ViDi Training 정보를 얻어오고 이를 CSV 파일로 기록합니다.

<br/>
#### 전역변수 설명
샘플 프로그램 내 전역변수는 아래 표와 같이 주로 경로 또는 파일이름 등을 지정하는 역할을 합니다.

|      변수명     |                 변수 설명                |
|:---------------:|:----------------------------------------:|
|   <font face="Consolas">working_path</font>  | ViDi Workspace의 경로를 지정합니다.      |
|     <font face="Consolas">img_path</font>    | 이미지 경로를 지정합니다.                |
|   <font face="Consolas">result_path</font>   | Result 파일(CSV 파일) 경로를 지정합니다. |
| <font face="Consolas">result_filename</font> | Result 파일명을 지정합니다.              |
|  <font face="Consolas">workspace_name</font> | ViDi Workspace의 이름을 지정합니다.      |
|   <font face="Consolas">stream_name</font>   | Workspace 내 stream의 이름을 지정합니다. |
|    <font face="Consolas">tool_name</font>    | Workspace 내 tool의 이름을 지정합니다.   |

<img src="{{ site.baseurl }}/assets/posts/2018-07-02-ResultFromXml/02.png">

전역변수 설정에 대한 코드는 아래와 같습니다.

{% highlight csharp %}
////////////////////////////////////////////////////////////
// DEFINE GLOBAL VARIABLES
////////////////////////////////////////////////////////////
//where the workspace is located
const std::string working_path("../vidi-resource/ws/");
const std::string img_path("../vidi-resource/imgs/");
const char* result_path = "../Result/";
const std::string result_filename("result.csv");

const char* workspace_name = "Green";
const char* stream_name = "default";
const char* tool_name = "Classify";
{% endhighlight %}

<br/>
#### Result 폴더 생성
본 샘플 프로그램에서는 결과를 CSV 파일로 저장하며, 이 파일이 위치할 폴더명을 Result라고 지정하였습니다.

지정된 이름으로 폴더를 생성하는 함수인 `CreateFolder()` 구현은 다음과 같으며,

{% highlight csharp %}
void CreateFolder(const char * path)
{
  if (!CreateDirectory(path, NULL))
  {
    return;
  }
}
{% endhighlight %}

`main()` 함수 내에서 이를 호출하여 Result 폴더를 생성합니다.

{% highlight csharp %}
CreateFolder(result_path);
{% endhighlight %}

<br/>
#### ViDi Workspace 오픈
본 샘플 프로그램에서 사용되는 ViDi Workspace는 ViDi GUI Suite으로부터 Export된 Worskpace 파일이 아니며, 실제 ViDi Workspace를 의미합니다.

즉, Visual Studio 솔루션 내 `vidi-resource/ws/Green` 폴더를 확인하시면 실질적인 ViDi Workspace 파일들이 존재하는 것을 확인할 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-02-ResultFromXml/02.png">

전역변수 설정에 대한 코드는 아래와 같습니다.

{% highlight csharp %}
////////////////////////////////////////////////////////////
// DEFINE GLOBAL VARIABLES
////////////////////////////////////////////////////////////
//where the workspace is located
const std::string working_path("../vidi-resource/ws/");
const std::string img_path("../vidi-resource/imgs/");
const char* result_path = "../Result/";
const std::string result_filename("result.csv");

const char* workspace_name = "Green";
const char* stream_name = "default";
const char* tool_name = "Classify";
{% endhighlight %}

<br/>
#### Result 폴더 생성
본 샘플 프로그램에서는 결과를 CSV 파일로 저장하며, 이 파일이 위치할 폴더명을 `Result`라고 지정하였습니다.

지정된 이름으로 폴더를 생성하는 함수인 `CreateFolder()` 구현은 다음과 같으며,

{% highlight csharp %}
void CreateFolder(const char * path)
{
  if (!CreateDirectory(path, NULL))
  {
    return;
  }
}
{% endhighlight %}

`main()` 함수 내에서 이를 호출하여 `Result` 폴더를 생성합니다.

{% highlight csharp %}
CreateFolder(result_path);
{% endhighlight %}

<br/>
#### ViDi Workspace 오픈
본 샘플 프로그램에서 사용되는 ViDi Workspace는 ViDi GUI Suite으로부터 Export된 Worskpace 파일이 아니며, 실제 ViDi Workspace를 의미합니다.

즉, Visual Studio 솔루션 내 `vidi-resource/ws/Green` 폴더를 확인하시면 실질적인 ViDi Workspace 파일들이 존재하는 것을 확인할 수 있습니다.

<img src="{{ site.baseurl }}/assets/posts/2018-07-02-ResultFromXml/03.png">

ViDi C API를 통해 Workspace를 여는 코드는 다음과 같습니다:

{% highlight csharp %}
// open the workspace
CHECK_DLL_CALL(vidi_training_open_workspace(workspace_name, workspace_path.c_str(), ));
{% endhighlight %}

<br/>
#### Training Images 불러오기
Training에 사용된 Image 경로에 있는 모든 Images를 불러옵니다.

{% highlight csharp %}
// fetch all image files from image path
std::vector<std::string> vec_files;
vec_files = get_all_files_names_within_folder(img_path);
{% endhighlight %}

`get_all_files_names_within_folder()` 함수 구현은 다운로드 한 솔루션 파일 내 소스를 참고하시기 바랍니다.

<br/>
#### 각 이미지에 대한 Training 정보 얻어오기
앞서 얻어 온 모든 이미지 리스트(vector)로부터 각 이미지에 대한 Training 정보를 얻어옵니다.

얻어오는 정보는 다음과 같습니다:

- 이미지 파일명
- 작업자가 지정한 Tag
- ViDi 프로세스에 의한 결과 Tag
- 결과 Tag에 대한 스코어

이들 결과를 가져오는 ViDi C API 함수는 `vidi_training_tool_get_marking()` 입니다:

<img src="{{ site.baseurl }}/assets/posts/2018-07-02-ResultFromXml/04.png">

For Loop를 이용하여 각 이미지를 불러오고 이를 `vidi_training_tool_get_marking()`로부터 정보를 가져온 후 각 정보를 CSV 파일로 저장합니다:

{% highlight csharp %}
or (std::vector<int>::size_type i = ; i < vec_files.size(); ++i)
{
    img_name = vec_files[i].c_str();
    CHECK_DLL_CALL(vidi_training_tool_get_marking(workspace_name, stream_name, tool_name, img_name, "", &marking));

    // parse node data
    doc.parse<0>(marking.data);
    root_node = doc.first_node("marking");
    view_node = root_node->first_node("view");
    tool_node = view_node->first_node("green");
    result_tag_node = tool_node->first_node("tag");
    label_node = tool_node->first_node("label");
    label_tag_node = label_node->first_node("tag");

    // record result into file
    ofs << img_name << ", "
        << label_tag_node->first_attribute("name")->value() << ", "
        << result_tag_node->first_attribute("name")->value() << ", "
        << result_tag_node->first_attribute("score")->value() << "\n";

    // display progress status
    std::cout << "Progress(%) : " << (100.0f * (i + 1) / vec_files.size()) << std::endl;
}
{% endhighlight %}

각 이미지에 대한 결과 정보는 `marking`이라는 `VIDI_BUFFER` 변수에 지정되며, XML 형식으로 저장됩니다. 예를 들어, 다음과 같은 형식을 갖습니다:

{% highlight xml %}
<marking tool_type='green' tool_name='Classify' feature_size='100' added='2018-May-25 15:41:34' processed='2018-May-25 16:38:46.239702' duration='0.0198455'>
    <image_info size='384x288' channels='3' depth='8'/>
    <view size='384x288' pose='1.000000,0.000000,0.000000,1.000000,0.000000,0.000000' group=''>
        <green threshold='0.500000'>
            <tag name='1' score='0.983638'/>
            <tag name='9' score='0.011774'/>
            <label unknown='false'>
                <training_flag value='true' manual='false'/>
                <tag name='1'/>
            </label>
        </green>
    </view>
</marking>
{% endhighlight %}

이러한 형식을 바탕으로 위의 코드에 대한 힌트를 얻으시기 바랍니다.

------
## 결과 파일로부터 원하는 결과 도출하기
코드를 실행하면 Solution 폴더 내의 Result 경로에 `result.csv`가 생성되며, 다음과 같은 형식을 갖습니다:

{% highlight text %}
ImageName, LabeledTag, ResultTag, ResultScore
100_l1c1.png, 100, 100, 0.999944
100_l1c2.png, 100, 100, 0.998920
100_l1c3.png, 100, 100, 0.923282
100_l2c1.png, 100, 100, 0.999991
100_l2c2.png, 100, 100, 0.999830
100_l2c3.png, 100, 100, 0.999052
100_l3c1.png, 100, 100, 0.999998
100_l3c2.png, 100, 100, 0.999987
100_l3c3.png, 100, 100, 0.999979
100_l4c1.png, 100, 100, 0.999984
100_l4c2.png, 100, 100, 0.999997
100_l4c3.png, 100, 100, 0.999996
100_l5c1.png, 100, 100, 0.999986
...
{% endhighlight %}
