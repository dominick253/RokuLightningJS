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
RokuLightningApp/
├── lightning-app/            # Your Lightning JS app source
│   └── dist/
│       └── index.js
├── roku-wrapper/             # Your Roku BrightScript wrapper
│   ├── manifest
│   ├── source/
│   │   └── main.brs
│   └── components/
│       ├── MainScene.xml
│       └── MainScene.brs
└── android-wrapper/ # (Optional) WebView wrapper for Android TV / Fire TV
```



<h1>1. Install Lightning CLI:</h1>

```bash
npm install -g @lightningjs/cli
```

<h1>2. Create your Lightning app:</h1>

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

<h1>1. Create manifest</h1>

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

<h1>2. Create source/main.brs</h1>

```brightscript
sub Main(args as dynamic)
    print "Lightning JS wrapper: Main started with args: "; args

    screen = CreateObject("roSGScreen")
    port = CreateObject("roMessagePort")
    screen.SetMessagePort(port)

    scene = screen.CreateScene("MainScene")
    print "Lightning JS wrapper: Showing screen"

    screen.Show()

    while true
        msg = wait(0, port)
        if type(msg) = "roSGScreenEvent" then
            if msg.IsScreenClosed() then
                print "Lightning JS wrapper: Screen closed, exiting app."
                exit while
            end if
        end if
    end while
end sub
```


<h1>3. Add components/MainScene.brs</h1>

```brightscript
sub init()
    m.canvas = m.top.findNode("canvas")
    m.canvas.url = "pkg:/index.js"
end sub
```

<h1>4. Add components/MainScene.xml</h1>

```brightscrip
<component name="MainScene" extends="Scene">
  <script type="text/brightscript" uri="pkg:/components/MainScene.brs" />
  <children>
    <CanvasView id="canvas" />
  </children>
</component>
```


<h1>Package Your Roku App</h1>

1. Place your Lightning build (index.js, assets) inside your wrapper folder (roku-wrapper/).
   
3. Zip the contents of (not the folder itself).

4. Sideload to your Roku:

 Enable Developer Mode on your Roku.

 Visit http://<ROKU_IP_ADDRESS> in your browser.

 Upload your zip file.

 Click Install.



<h1>Android TV / Fire TV Deployment</h1>
While Roku requires a wrapper, Android TV and Fire TV can run your Lightning app inside a WebView:

1. Create a basic Android Studio project.

2. Add a WebView that loads your local index.html from your Lightning build.

3. Build an .apk for sideloading or store deployment.



