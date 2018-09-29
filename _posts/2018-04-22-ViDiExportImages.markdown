---
layout: post
title:  "How to Export Images in ViDi"
date:   2018-04-22 14:57:00
author: Alex Choi
categories: Deep-Learning
---

이번 포스팅에서는 ViDi에서 각 View 이미지를 일괄적으로 내보내는 방법에 대하여 알아보도록 하겠습니다.

ViDi의 기능 중 각 View를 하나씩 이미지로 내보낼 수 있는 방법은 제공됩니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-22-ViDiExportImages/01.png">
<center>그림 1. ViDi의 Export View 기능</center>

그러나, 안타깝게도 현재 ViDi는 모든 View를 이미지 파일로 일괄적으로 내보낼 수 있는 방법은 제공하고 있지 않습니다.

필요에 따라, 특히 각 View에 대하여 고객 측에서 Labeling을 해줘야 하는 경우 Blue Tool을 이용하여 Localization 된 View들을 이미지로 정리하여 전달해야 할 경우가 있습니다 - 삼성전기 MLCC의 건이 그랬습니다. (이미지 1장 내(Single Camera FOV 이내)에 수백개의 MLCC를 한꺼번에 촬영하고 Blue Tool로 Localization하여 개별 MLCC 이미지로 분리한 후 이 이미지들을 고객에게 보내어 Labeling을 요청하였습니다.)

가령, 여러 장의 이미지로부터 추출된 View의 개수가 1000개가 넘는다면 이를 일일히 Export View를 클릭하여 저장하기는 거의 불가능해 보입니다.

노가다(?)를 동원하지 않고도 간단한 설정으로 모든 View를 이미지로 내보낼 수 있는 방법에 대하여 알아보겠습니다.

------
## HTML 형식의 Report 생성하기
불행 중 다행(?)으로 ViDi에서 HTML 파일 형식으로 Report를 생성할 수 있으며, 이 파일 안에 [Base 64로 인코딩](https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%9D%B4%EC%8A%A464) 된 모든 View의 이미지 정보가 담겨 있습니다 (`ViDi Menu > Database > Create Report`).

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-22-ViDiExportImages/02.png">
<center>그림 2. Report 생성하기</center>

Report 생성 시 원하는 이미지 사이즈나 마킹 여부 등 이미지 옵션은 다음 이미지와 같이 설정합니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-22-ViDiExportImages/03.png">
<center>그림 3. Report 생성 옵션</center>

<br/>
이렇게 해서 생성된 Report HTML 파일을 텍스트 편집기를 이용해서 열어보면 중간중간 Base64로 인코딩 된 이미지 태그(`<img>`)를 보실 수 있습니다.

``` HTML
<img src="data:image/jpeg;base64,/9j/4AAQSkZJR...(중간생략)...=" usemap="#01.png:24" /><map name="01.png:24">
```

따라서, HTML 파일의 전체 `<img>` 태그를 따라가며, Base64로 인코딩 된 이미지를 다시 디코딩하여 이미지 파일로 저장하면 각 View를 일괄적으로 이미지 파일로 저장할 수가 있습니다.

------
## Python 개발환경 구축
제가 Python을 선택한 이유는, 단지 유사한 작업을 Python으로 해봤기 때문입니다. 당연히 다른 언어에서도 동일한 작업이 가능할 것입니다.

우선 코딩 작업을 위해 필요한 [BeautifulSoup Python 라이브러리](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)를 설치하도록 하겠습니다.

[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)은 HTML 또는 XML 파일로부터 유용한 정보를 얻어오는데 사용하는 매우 편리한 Python 라이브러리입니다. 웹크롤링(Web Crawling) 등을 하는데 많이 사용되며, 본 코드에서는 `<img>` 태그를 선택하여 Base64 정보를 얻어오는 역할을 합니다.

Python에서 라이브러리를 설치하는 방법은 다양합니다. 소스를 다운받아 설치할 수도 있으며, Python Package Indes 명령어(`pip`)를 이용하여 설치할 수도 있고, [Anaconda](https://www.anaconda.com/) 환경이라면 `conda install` 명령어를 사용하여 설치도 할 수 있습니다.

이 중 가장 선호하는 방법은, 우선 Anaconda 환경을 구성하고([Anaconda Download](https://www.anaconda.com/download/)) `conda` 명령을 통해 [ANACONDA CLOUD Repository](https://anaconda.org/cinema4dr12/dashboard)로부터 설치하는 것입니다.

``` HTML
$ conda install -c conda-forge beautifulsoup4
```

만약 Anaconda 환경이 아닌 순수한 Python 실행 환경인 경우에는 `pip` 또는 `easy_install` 명령을 통해 설치합니다 (소스를 다운받아 설치하는 것은 고생스럽기 때문에 별로 추천하지 않습니다).

``` HTML
$ pip install beautifulsoup4
```

또는

``` HTML
$ easy_install beautifulsoup4
```

------
## Python Code
우선 작성한 Python 전체 코드는 다음과 같습니다.

{% highlight python %}
# -*- coding: utf-8 -*-
"""
@ Base64 to images
@ Created on Wed Mar  7 17:40:44 2018
@ author: achoi
"""

###############################################################################
## USER DEFINITION
###############################################################################
MY_URL = "file:///D:/temp/ExportViews.html"
OutputPath = 'D:/temp/ExportViews'
OutputFilename = "bottle"


###############################################################################
## import python libraries
###############################################################################
import base64
import urllib.request
from bs4 import BeautifulSoup


###############################################################################
## @ function : main
###############################################################################
if __name__ == "__main__":
    # request URL
    req = urllib.request.Request(MY_URL)

    # fetch data from URL
    data = urllib.request.urlopen(req).read()

    bs = BeautifulSoup(data, 'html.parser')
    img_tag = bs.find_all('img')

    # loop for processing all the <img> tags
    cnt = 0
    for ii in range(len(img_tag)):
        img = img_tag[ii]
        src = img.attrs['src']
        img_str = str(src).split(',')[1]

        if(img_str.find('/9j/') > -1):
            img_b64 = str.encode(img_str)

            cnt += 1
            filepath = '%s/%s_%07d.png' % (OutputPath, OutputFilename, cnt)

            with open(filepath, "wb") as fh:
                fh.write(base64.decodebytes(img_b64))
{% endhighlight %}

본 코드를 사용하는데 있어 이해해야 할 사항은 사용자 정의(USER DEFINITION 블럭)

#### My URL
`MY_URL`은 Report HTML 파일을 웹 브라우저에서 열었을 때 주소창에 보이는 해당 HTML 파일의 파일 프로토콜 주소입니다. 가령, HTML의 드라이브 상의 경로가 `D:/temp/ExportViews.html`이라면 웹 브라우저에 보이는 주소창은, `file:///D:/temp/ExportViews.html` 이 됩니다. 이 경로를 복사하여 `MY_URL`에 등록하면 됩니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-22-ViDiExportImages/04.png">
<center>그림 4. 웹 브라우저 주소창의 파일 프로토콜 주소</center>
<br/>

#### OutputPath
이미지가 저장될 경로입니다.

#### OutputFilename
출력될 이미지의 Prefix입니다.

예를 들어, 위의 코드를 실행하면 아래 이미지와 같은 이미지 시퀀스를 얻을 수 있습니다.

<br/>
<img src="{{ site.baseurl }}/assets/posts/2018-04-22-ViDiExportImages/05.png">
<center>그림 5. 생성된 이미지 시퀀스</center>
<br/>

이상으로, ViDi에서 각 View 이미지를 일괄적으로 내보내는 방법에 대하여 알아보았습니다.
