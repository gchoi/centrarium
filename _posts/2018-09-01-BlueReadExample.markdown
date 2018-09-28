---
layout: post
title:  "ViDi API / Blue Read Example"
date:   2018-09-01 23:19:00
author: Alex Choi
categories: Deep-Learning
---

Today I'd like to share an ViDi API example of Blue Read tool with my experiencing from SDI battery OCR evaluation project.

The target of the project is to read the characters, which are marked with laser, on the upper part of batteries - there may be more than three batteries in an image, so our mission is to read all the characters on those batteries at once!

<img src="{{ site.baseurl }}/assets/posts/2018-09-01-BlueReadExample/01.png" title="Original">

<img src="{{ site.baseurl }}/assets/posts/2018-09-01-BlueReadExample/02.png" title="Read">

Firstly, I wanna share my example solution (zipped) file - Click the link below to download it!

[Download ViDi.Runtime.BlueRead.zip](https://cognexcorporation-my.sharepoint.com/:u:/g/personal/alex_choi_cognex_com/EaDaMpG5Wr9Mivo24B4XG8ABv7XXjIikk7F_yulN6oOErg?e=fPH57d)

By unzipping it and navigating to `\ViDI.Runtime.BlueRead\resource\vidi-training-workspace\` you can find the ViDi training workspace file : `SDI-Battery-OCR(Model).vwsa`.

Feel free to import the workspace to look through and analyze the workflow :)

------
## Development Environment
- **OS** : Windows 10 x64
- **ViDi Suite** : v3.1.1.10701 ([Latest build from our official website](https://support.cognex.com/en/downloads/detail/deep-learning/3676/1033))
- **IDE** : Visual Studio 2015 with .NET Framework 4.6.1

------
## Resources
If you navigate to `.\ViDI.Runtime.BlueRead\ViDI.Runtime.BlueRead\resource\` path in the example solution file after unzipped, three folders, `test-images`, `vidi-runtime-workspace` and `vidi-training-workspace` are found in it which are described as below:

- **vidi-training-workspace** : the path where ViDi training workspace file, "SDI-Battery-OCR(Model).vwsa" exists, which is actually not necessary for executing the runtime API, rather just for reference of how to train the OCR!
- **vidi-runtime-workspace** : the path where ViDi runtime workspace file, "Battery-OCR(Model) - Test.vrws" exists, which is essential for executing this runtime API.
- **test-images** : the path where the test images are included.

In the code, all the resource paths are defined by _relative path_ not _absolute path_ in order for anyone to run the code under any root path you locate the solution file (You shall see what this means in CODE explanation part).

BUT, keep in mind that the resource path location starts from the binary execution file path - `.\ViDI.Runtime.BlueRead\ViDI.Runtime.BlueRead\bin\Release\` if the configuration (either  debug or release) is set to "release", for example.

------
## Code
Since most of code lines are thought to be self-explanatory, I'll just briefly outline some important parts.

#### Global Definitions
The following code is to globally define paths and stream name.

``` csharp
static class Constants
{
    // path where ViDi runtime workspace exists
    public const string VRWS_PATH = "..\\..\\resource\\vidi-runtime-workspace";

    // ViDi Runtime workspace filename without ".vrws"
    public const string VRWS_FILENAME = "Battery-OCR(Model) - Test";

    // stream name
    public const string STREAM_NAME = "3EA-MetalUpper";

    // Blue Read tool name
    public const string TOOL_NAME = "Read";
    // test image path
    public const string IMG_PATH = "..\\..\\resource\\test-images";
}
```

- `VRWS_PATH` : relative path to ViDi runtime workspace
- `VRWS_FILENAME` : ViDi runtime workspace file name
- `STREAM_NAME` : stream to be used for runtime API
- `TOOL_NAME` : name of Blue Read tool
- `IMG_PATH` : relative path to test images

#### Instance for ViDi Runtime Main Control
It's an essential instance for runtime code and like workspace container.

``` csharp
// holds the main control
ViDi2.Runtime.IControl control = new ViDi2.Runtime.Local.Control();
```

#### Runtime Workspace from File
The following code opens the runtime workspace from file.

``` csharp
// opens a runtime workspace from file
string strRuntimeWorspacePath = String.Concat(Path.Combine(Constants.VRWS_PATH, Constants.VRWS_FILENAME), ".vrws");
IWorkspace workspace = null;

try
{
    workspace = control.Workspaces.Add(Constants.VRWS_FILENAME, strRuntimeWorspacePath);
}
catch(System.Exception e)
{
    System.Console.WriteLine("{0} Exception caught.", e);
    System.Console.WriteLine("\nFailed to load ViDi Runtime Workspace.");
    System.Console.WriteLine("\nPress enter to exit....");
    strTmp = System.Console.ReadLine();
    return;
}
```

`String.Concat()` method is used to define the combine the runtime workspace path and its filename.

Workspace is added to `control` by `control.Workspaces.Add()` method in `try-catch` block to avoid the exception error in case that it fails to load workspace.

#### Load Stream
The following code defines the stream to be used in runtime.

``` csharp
// load stream
IStream stream = null;

try
{
    stream = workspace.Streams[Constants.STREAM_NAME];
}
catch (System.Exception e)
{
    Console.WriteLine("{0} Exception caught.", e);
    System.Console.WriteLine("\nFailed to find stream.");
    System.Console.WriteLine("\nPress enter to exit....");
    strTmp = System.Console.ReadLine();
    return;
}
```

#### Load Images from Image Path
In the following code, `Directory.GetFiles()` method gets the list of the files from the specified (test-images) path.

If either the method fails to locate the path or no test images exist in that path, then the program will terminate.

``` csharp
// load images from image path
string[] fileEntries = null;

try
{
    fileEntries = Directory.GetFiles(Constants.IMG_PATH);
}
catch (System.Exception e)
{
    Console.WriteLine("{0} Exception caught.", e);
    System.Console.WriteLine("\nFailed to find image path.");
    System.Console.WriteLine("\nPress enter to exit....");
    strTmp = System.Console.ReadLine();
    return;
}

if (fileEntries.Length < 1)
{
    System.Console.WriteLine("No image filename in the specified path.");
    System.Console.WriteLine("\nNo images in the path.");
    System.Console.WriteLine("\nPress enter to exit....");
    strTmp = System.Console.ReadLine();
    return;
}
```

#### Processing Image One-by-One
We'are almost done. The last part of the example code is as following:

``` csharp
foreach (string strImageFilename in fileEntries)
{
    string[] words = strImageFilename.Split('\\');
    string strCurrentImageFilename = words[words.Length - 1];

    System.Console.WriteLine("=======================================================");
    System.Console.WriteLine(" Current Image Filename : {0}", strCurrentImageFilename);
    System.Console.WriteLine("=======================================================");

    // load an image from file
    using (IImage image = new FormsImage(strImageFilename))
    {
        System.Diagnostics.Stopwatch globalSw = new System.Diagnostics.Stopwatch();
        globalSw.Start();

        // process image with the blue read tool
        ISample sample = stream.Tools[Constants.TOOL_NAME].Process(image);

        Console.WriteLine("\nProcessed in {0} ms \n", globalSw.ElapsedMilliseconds);
        globalSw.Stop();

        IBlueMarking blueReadMarking = sample.Markings[Constants.TOOL_NAME] as IBlueMarking;

        if (blueReadMarking != null)
        {
            foreach (IBlueView view in blueReadMarking.Views)
            {
                foreach (IMatch match in view.Matches)
                {
                    double score = match.Score;
                    System.Console.WriteLine("This model says \"" + match.FeatureString + "\" with a score of " + score);

                    int nCnt = 0;
                    foreach (IFeature feature in match.Features)
                    {
                        System.Console.WriteLine("Feature [{0}]: {1} / Position: ({2}, {3})", nCnt, feature.Name, feature.Position.X, feature.Position.Y);
                        nCnt++;
                    }

                    System.Console.WriteLine();
                }
            }
        }
    }
}
```

`foreach` loop is used for processing the image one by one in the test image path. `using (IImage image = new FormsImage(strImageFilename))` envelops the part of the processing the currently loaded image.

The variable `globalSw()` plays role of recording the processing time, which is called "takt time" in  machine vision field.

From `IBlueMarking` interface instance we get the views (also called ROIs from Blue-Localize tool) from which characters are read.

In `foreach (IMatch match in view.Matches)` loop, we can get the results for each "match" of `view.Matches` - those are:

- **score** : matching score of the model we built (in Blue Read tool, "model" is a sequence of character(s), which is also called "feature".)
- **match.Features** : each character component in the model - actually just character(s)
- **Position of match.Features** : literally, the position of each character(feature) that composes the model

------
## Result
<img src="{{ site.baseurl }}/assets/posts/2018-09-01-BlueReadExample/03.png" title="Result">

The image below is a part of the execution result shown in command line window:

Firstly, we can see the current image being processed (60.bmp) and the processing time(317 ms) below it.

Then the model, sequence of the characters that read by ViDi, with its matching score comes after the processing time.

Finally, the positions of the corresponding characters are shown.
