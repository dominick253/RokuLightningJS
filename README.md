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
title=LightningApp
major_version=1
minor_version=0
build_version=00001
mm_icon_focus_hd=pkg:/images/icon_focus_hd.png
mm_icon_focus_sd=pkg:/images/icon_focus_sd.png
```

<h1>2. Create roku-wrapper/main.brs</h1>

```brightscript
sub Main()
    screen = CreateObject("roSGScreen")
    m.port = CreateObject("roMessagePort")
    screen.setMessagePort(m.port)

    scene = screen.CreateScene("MainScene")
    screen.show()

    while true
        msg = wait(0, m.port)
    end while
end sub
```


<h1>3. Add roku-wrapper/source/MainScene.brs</h1>


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



