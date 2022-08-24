---
title: "VRChat"
date: 2022-01-15
tags: ["VRChat"]
---

# VRChat

[VRChat](https://hello.vrchat.com/)  
[Documentation](https://docs.vrchat.com/docs)  
[SDK](https://vrchat.com/home/download)

## Unity のインストール

[Currently Supported Unity Version](https://docs.vrchat.com/docs/current-unity-version)  
こちらの Click here to install the current version of Unity via Unity Hub を押す

Microsoft Visual Studio Community 2019  
Android Build Support  
Android SDK & NDK Tools  
OpenJDK にチェックし Install

## Avatar

### Initial Setup and Test

Create a new Unity project.  
Import VRCSDK3-AVATAR by "Assets -> Import Package -> Custom Package...".  
Import VRCQuestTools.  
Import lilToon.  
Import an avatar.  
Put *.unity of the avatar into Hierarchy.  
Copy the avatar from its scene to "Sample Scene".  
Rename the avatar name.  
Delete its scene.  
Select an avatar in Hierarchy to be converted by VRCQuestTools.  
Convert the avatar by VRCQuestTools by "VRCQuestTools -> Convert Avatar for Quest".  
Choose a directory to save a converted avatar. e.g. Assets/KRT/QuestAvatars/  
Press the button of "変換"  
Get its PhysBones matched between PC and Quest.  
Reduce its PhysBones. The maximum is 8.  
Test the avatar locally.  
Upload the avatar for PC.  
Change its build mode into Android by "File -> Build Settings...".  
Change its shader for transparency.  
Open "VRChat SDK -> Content Manager" and press "Copy ID" of the avatar you uploaded for PC.  
Paste it on "Inspector -> Blueprint ID (Optional)" of the Quest avatar.  
Press "Attach (Optional)".  
Upload it for Quest and Test.  

You can press the button saying "ASTC でテクスチャを圧縮" being showed by VRCQuestTools but ASTC needs a good CPU to build.  

[VRCQuestTools](https://kurotu.booth.pm/items/2436054)  
[ASTC](https://omega.hatenadiary.jp/entry/2019/07/24/032528)  
[lilToon](https://booth.pm/ja/items/3087170)  

### Upload for PC

Change its build mode into PC by "File -> Build Settings...".  

### Its face is too red in Quest.

Change the shader of the face alpha from "Hidden/lilToonTransparent" to "VRChat/Mobile/Particles/Multiply".  
e.g. The name the shader is attached to is like "M_Face_alpha_from_blah-blah-blah" for Kikyo avatar.  
Caution: "VRChat/Mobile/Particles/Multiply" can not be seen by PC players.  

[https://twitter.com/till0196_vrchat/status/1364809328801193985](https://twitter.com/till0196_vrchat/status/1364809328801193985)  
[https://twitter.com/ring_say_rip/status/1364706722359562241](https://twitter.com/ring_say_rip/status/1364706722359562241)  

### PhysBones

[VRChat PB対応メモ書き(Quest化含む)](https://note.com/hukube_vrc/n/n76ef8fb112c4)  
[【VRChat】PhysBoneの設定方法・Componentについて](https://signyamo.blog/phys-bone_component/)  

### Shader

[VRChatterのためのシェーダーの基本、完全に理解した](https://qiita.com/hirakichi/items/8282ff70c28548c0eb16)  

### Get the base of Kikyo

[シャツセットアップガイド](https://docs.google.com/document/d/1T8Dy269GhTnmI7LadW9rg5NTxkZfW4cXNv_3rgxh_YE/edit#heading=h.dd58ddyj1og8)  

## World

[U#](https://github.com/MerlinVR/UdonSharp)  
[Trigger2to3](https://www.wicurio.com/trigger2to3/)

### 新しいプロジェクトのつくりかた

Unity Hub -> Projects -> New project -> 3D  
Project name と Location を設定して Create project

Assets に Scenes だけがあるが，その右の広いところで右クリック  
-> Import Package -> Custom Package... -> VRCSDK3-WORLD -> 開く -> Import -> I Made a Backup. Go Ahead!

もう一度 Assets の広いところで右クリック  
-> Import Package -> Custom Package... -> UdonSharp -> 開く -> Import

### VRCWorld の場所

Assets -> VRChat Examples -> Prefabs  
VRCWorld は，Hierarchy にドラッグ & ドロップ

### アップロードのしかた

VRChat SDK -> Show Control Panel  
ログインする  
Builder -> Setup Layers for VRChat -> Do it! -> Set Collision Matrix -> Do it! -> Build & Publish for Windows -> 設定して Upload  
File -> Build Settings... -> Android -> Switch Platform  
VRChat SDK -> Show Control Panel -> Builder -> Build & Publish for Android -> 設定して Upload

ヒント: Upload の画面で Scene に戻り VRCCam を動かすと，プレビューの変更ができる

### uGUI

とりあえず Create Empty をつくって空の GameObject をつくり，その中につくっていくのがよさそう  
U# のソースファイルも空の GameObject に Add Component -> Udon Behaviour でつくる

#### Canvas

Scale -> Z を 0 にする  
Render Mode を World Space にする  
Add Component -> VRC Ui Shape

#### Panel

Source Image を None にする

#### Button

On Click () のところに U# ファイルの入った GameObject を入れて UdonBehaviour.Interact () を選ぶ  
もちろん U# ファイルには Interact() メソッドの実装を書いておく  

```
public override void Interact() {
    // Write what you want to invoke...
}
```

## Playing

フレンドが 30人 になった瞬間 New User -> User になった
