How to transition, and setup, a Roku Brightscript and scenegraph app to LightningJS.

I can't find any good documentation for Roku LightningJS, so I will make my own! Any contributions, typos, errors, please point out and help build this repo of information, including boilerplate and eventually starter templates. End goal is for a full CI/CD pipeline for all three platforms (Roku, Android TV, and Fire TV) to test, build, and deploy all versions automatically. 

Long way to go but every journey starts with a single step.
Code Taken From https://rdkcentral.github.io/Lightning-SDK/#/getting-started
Please see the above website for more information.


<h1>Roadmap</h1>
 1. Create minimal BrightScript wrapper

 2. Automate build steps with scripts

 3. CI/CD pipeline for .zip and .apk generation

 4. Add sample Lightning components



<h1>Project Structure</h1>

```
RokuLightningJS/
├── lightning-app/ # Your Lightning JS app
├── roku-wrapper/ # BrightScript wrapper for Roku
│ ├── manifest
│ ├── main.brs
│ ├── source/
│ └── pkg/
└── android-wrapper/ # (Optional) WebView wrapper for Android TV / Fire TV
```



<h1>1. Install Lightning CLI:</h1>

```bash
npm install -g @lightningjs/cli
```

<h1>Create your Lightning app:</h1>

```bash
lng create my-lightning-app
cd my-lightning-app
```


<h1>3. Develop and Test:</h1>

```bash
lng dev
```

<h1>4. Build the app</h1>

```bash
lng build
```



<h1>Prepare Roku Wrapper</h1>
Since Roku does not run web apps directly, we use a BrightScript + SceneGraph wrapper that loads the Lightning Renderer with your app.

<h1>1. Create roku-wrapper/manifest</h1>

```plaintext
# Copyright (c) 2020 Roku, Inc. All rights reserved.

# Channel Details
title=[Channel Title]
major_version=1
minor_version=1
build_version=1

# Channel Assets
mm_icon_focus_hd=pkg:/images/image1.png

# Splash Screen
splash_screen_hd=pkg:/images/image2.png

# Resolution
ui_resolutions=hd

supports_input_launch=1
enable_deeplinking=true
```

<h1>2. Create roku-wrapper/main.brs</h1>

```brightscript
sub Main(args as dynamic)
    print "Main started with args: "; args
    contentId = args.contentId
    mediaType = args.mediaType
    print "mediaType: "; mediaType
    print "contentId: "; contentId

    xfer = CreateObject("roURLTransfer")
    xfer.SetCertificatesFile("common:/certs/ca-bundle.crt")
    xfer.SetURL("[Your URL Here]")
    feed = xfer.GetToString()

    m.screen = CreateObject("roSGScreen")
    m.port = CreateObject("roMessagePort")
    m.screen.SetMessagePort(m.port)

    m.scene = m.screen.CreateScene("MainScene")
    print "Showing screen"
    m.scene.feedJson = feed

    m.screen.Show()

    ' Handle deep link if present
    if contentId <> invalid and type(contentId) = "roString"
        print "Checking contentId: "; contentId
        if valid_contentId(contentId)
            print "ContentId valid, calling playContent"
            ' Wait briefly to ensure scene is initialized
            sleep(100)
            m.scene.callFunc("playContent", {
                contentId: contentId,
                feedJson: feed
            })
        else
            print "Invalid contentId: "; contentId
        end if
    else
        print "No valid contentId provided or contentId is invalid"
    end if

    while true
        msg = wait(0, m.port)
        if type(msg) = "roSGScreenEvent" then
            if msg.IsScreenClosed() then
                exit while
            end if
        else if type(msg) = "roInputEvent" then
            if msg.IsInput()
                info = msg.GetInfo()
                if info.DoesExist("mediaType") and info.DoesExist("contentId")
                    mediaType = info.mediaType
                    contentId = info.contentId
                    print "Input event - mediaType: "; mediaType; ", contentId: "; contentId
                    if contentId <> invalid and type(contentId) = "roString" and valid_contentId(contentId)
                        print "Input event: Calling playContent"
                        m.scene.callFunc("playContent", {
                            contentId: contentId,
                            feedJson: feed
                        })
                    else
                        print "Input event: Invalid contentId: "; contentId
                    end if
                end if
            end if
        end if
    end while
end sub

function valid_contentId(contentId as string) as boolean
    print "Validating contentId: "; contentId
    xfer = CreateObject("roURLTransfer")
    xfer.SetCertificatesFile("common:/certs/ca-bundle.crt")
    xfer.SetURL("[Your URL Here]")
    rsp = xfer.GetToString()

    json = ParseJson(rsp)
    if json = invalid then
        print "Invalid JSON in valid_contentId"
        return false
    end if

    skipKeys = ["providerName", "lastUpdated", "language"]

    for each key in json
        skip = false
        for each s in skipKeys
            if s = key then skip = true : exit for
        end for

        if not skip
            items = json[key]
            if type(items) = "roArray"
                for each item in items
                    if item.DoesExist("id") and item.id = contentId
                        print contentId + " exists in the JSON feed >>>>>>"
                        return true
                    end if
                end for
            end if
        end if
    end for

    print "ContentId not found: "; contentId
    return false
end function
```


<h1>3. Add roku-wrapper/source/MainScene.brs</h1>

```brightscript
' ********** Copyright 2020 Roku Corp. All Rights Reserved. **********

sub Init()
    print "Initializing MainScene"
    m.top.signalBeacon("AppLaunchComplete")
    m.top.signalBeacon("AppDialogInitiate")
    m.top.signalBeacon("AppDialogComplete")

    feedJson = m.globalFields?.feedJson
    if feedJson <> invalid
        json = ParseJson(feedJson)
        if json = invalid
            print "JSON was invalid"
        else
            print "Feed JSON parsed successfully"
            ' Do your logic here with it
        end if
    else
        print "No feedJson passed in"
    end if


    m.top.backgroundColor = "0x662D91"
    m.top.backgroundUri = "pkg:/images/image2.png"
    m.loadingIndicator = m.top.FindNode("loadingIndicator")
    InitScreenStack()
    ShowGridScreen()
    RunContentTask()
end sub

sub onContentIdChange()
    contentId = m.top.contentId
    if contentId <> invalid and contentId <> ""
        print "contentId field changed, calling playContent: "; contentId
        playContent(contentId)
    else
        print "Invalid contentId in onContentIdChange"
    end if
end sub

function playContent(data as object) as void
    contentId = data.contentId
    feed = data.feedJson
    json = ParseJson(feed)

    if json = invalid
        print "Invalid JSON passed to playContent"
        return
    end if

    print "<<<<<<<<<< Valid Json received in PlayContent Function >>>>>>>>>>>>>>>>"

    skipKeys = ["providerName", "lastUpdated", "language"]

    for each key in json
        skip = false
        for each s in skipKeys
            if s = key then skip = true : exit for
        end for

        if not skip
            items = json[key]
            if type(items) = "roArray"
                for each item in items
                    if item?.DoesExist("id") and item.id = contentId
                        if item?.content?.videos?[0]?.url <> invalid
                            videoContent = CreateObject("roSGNode", "ContentNode")
                            videoContent.url = item.content.videos[0].url
                            videoContent.streamFormat = "mp4"
                            videoContent.title = item?.title
                            videoContent.description = item?.shortDescription

                            parentNode = CreateObject("roSGNode", "ContentNode")
                            parentNode.Update({ children: [videoContent] }, true)

                            overhang = m.top.findNode("overhang")
                            if overhang <> invalid
                                print "Hiding overhang"
                                overhang.visible = false
                            end if
                            loading = m.top.findNode("loadingIndicator")
                            if loading <> invalid
                                print "Hiding loadingIndicator"
                                loading.visible = false
                            end if

                            menuHidden = false
                            for each menuId in ["GridScreen", "MainMenu", "HomeScreen", "Grid"]
                                menu = m.top.findNode(menuId)
                                if menu <> invalid
                                    print "Hiding menu node: "; menuId
                                    menu.visible = false
                                    menuHidden = true
                                    exit for
                                end if
                            end for
                            if not menuHidden
                                print "Warning: No main menu node found. Listing all nodes:"
                                children = m.top.getChildren(-1, 0)
                                for each child in children
                                    print "Node ID: "; child.id; ", Type: "; child.subType(); ", Visible: "; child.visible
                                end for
                            end if

                            print "Calling ShowVideoScreen for contentId: "; contentId
                            ShowVideoScreen(parentNode, 0)
                            m.video = m.videoPlayer
                            print "Now playing: "; videoContent.url
                            if m.videoPlayer <> invalid
                                print "Video player state: "; m.videoPlayer.state
                            else
                                print "Error: m.videoPlayer is invalid after ShowVideoScreen"
                            end if
                            return
                        else
                            print "Invalid video data for contentId: "; contentId
                            return
                        end if
                    end if
                end for
            end if
        end if
    end for

    print "Content ID not found: "; contentId
end function

sub OnVideoStateChange()
    if m.video <> invalid and (m.video.state = "stopped" or m.video.state = "finished" or m.video.state = "error")
        print "Video state changed to: "; m.video.state
        CloseScreen(m.videoPlayer)
        m.top.removeChild(m.video)
        m.video = invalid
        overhang = m.top.findNode("overhang")
        if overhang <> invalid
            print "Restoring overhang"
            overhang.visible = true
        end if
        ShowGridScreen()
    end if
end sub

function OnKeyEvent(key as string, press as boolean) as boolean
    result = false
    if press
        if key = "back"
            numberOfScreens = m.screenStack.Count()
            if numberOfScreens > 1
                CloseScreen(invalid)
                result = true
            end if
        end if
    end if
    return result
end function
```


<h1>Package Your Roku App</h1>

1. Place your Lightning build inside roku-wrapper/pkg/lightning-bundle/.

2. Zip the contents of roku-wrapper/ (not the folder itself).

3. Sideload to your Roku:

 Enable Developer Mode on your Roku.

 Visit http://<ROKU_IP_ADDRESS> in your browser.

 Upload your zip file.

 Click Install.



<h1>Android TV / Fire TV Deployment</h1>
While Roku requires a wrapper, Android TV and Fire TV can run your Lightning app inside a WebView:

1. Create a basic Android Studio project.

2. Add a WebView that loads your local index.html from your Lightning build.

3. Build an .apk for sideloading or store deployment.



