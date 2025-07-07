# RokuLightningJS
How to transition, and setup, a Roku Brightscript and scenegraph app to LightningJS.

I can't find any good documentation for Roku LightningJS, so I will make my own! Any contributions, typos, errors, please point out and help build this repo of information, including boilerplate and eventually starter templates. End goal is for a full CI/CD pipeline for all three platforms (Roku, Android TV, and Fire TV) to test, build, and deploy all versions automatically. 

Long way to go but every journey starts with a single step.
Code Taken From https://rdkcentral.github.io/Lightning-SDK/#/getting-started
Please see the above website for more information.


Roadmap
 Create minimal BrightScript wrapper

 Automate build steps with scripts

 CI/CD pipeline for .zip and .apk generation

 Add sample Lightning components



Project Structure
RokuLightningJS/
├── lightning-app/ # Your Lightning JS app
├── roku-wrapper/ # BrightScript wrapper for Roku
│ ├── manifest
│ ├── main.brs
│ ├── source/
│ └── pkg/
└── android-wrapper/ # (Optional) WebView wrapper for Android TV / Fire TV




1. **Install Lightning CLI:**

```bash
npm install -g @lightningjs/cli
```

Create your Lightning app:
```bash
lng create my-lightning-app
cd my-lightning-app
```


3. Develop and Test:
```bash
lng dev
```

4. Build the app
```bash
lng build
```



Prepare Roku Wrapper
Since Roku does not run web apps directly, we use a BrightScript + SceneGraph wrapper that loads the Lightning Renderer with your app.

1. Create roku-wrapper/manifest
```plaintext
title=LightningApp
major_version=1
minor_version=0
build_version=00001
mm_icon_focus_hd=pkg:/images/icon_focus_hd.png
mm_icon_focus_sd=pkg:/images/icon_focus_sd.png
```

2. Create roku-wrapper/main.brs
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


3. Add roku-wrapper/source/MainScene.brs


Package Your Roku App
1. Place your Lightning build inside roku-wrapper/pkg/lightning-bundle/.

2. Zip the contents of roku-wrapper/ (not the folder itself).

3. Sideload to your Roku:

 Enable Developer Mode on your Roku.

 Visit http://<ROKU_IP_ADDRESS> in your browser.

 Upload your zip file.

 Click Install.



Android TV / Fire TV Deployment
While Roku requires a wrapper, Android TV and Fire TV can run your Lightning app inside a WebView:

1. Create a basic Android Studio project.

2. Add a WebView that loads your local index.html from your Lightning build.

3. Build an .apk for sideloading or store deployment.



