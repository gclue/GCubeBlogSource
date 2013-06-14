---
layout: post
title: "Android用開発環境の構築と実行"
date: 2013-06-12 12:33
comments: true
categories: 
---

本記事ではGCubeをAndroidの開発環境で利用する方法を紹介しています。

AndroidSDK及びNDKの導入部分にも触れていますが、既に環境を整えている場合は不要なので飛ばしてください。

## GCubeのダウンロード

GCube本体を[GitHub][github]よりダウンロードします。

{% img /images/gcube/TryAndroid/web-github-zip.png 'zip' 'zip' %}

## SDK及びNDKのダウンロード
Androidの公式サイトから、SDKとNDKのzipファイルをダウンロードしていきます。

[Get the Android SDK][sdk]
{% img /images/gcube/TryAndroid/web-sdk.png 'sdk' 'sdk' %}

[Android NDK][ndk]
{% img /images/gcube/TryAndroid/web-ndk.png 'ndk' 'ndk' %}

それぞれ解凍してFinderで開くと、フォルダ構成は以下のようになっています。
{% img /images/gcube/TryAndroid/finder-sdk.png 'finder-sdk' 'finder-sdk' %}

{% img /images/gcube/TryAndroid/finder-ndk.png 'finder-ndk' 'finder-ndk' %}

## SDK(ADT)の起動

解凍したSDKのフォルダの`eclipse`フォルダ以下にある`Eclipse.app`をダブルクリックして起動していきます。
ADTでは、初めて起動した場所のパスでSDKの設定を行いますので、パスに日本語が含まれないところで起動すると余計な心配が減らせると思います。NDKの配置場所も同じ理屈で決めておくとよいでしょう。

ADTを起動すると次のようなスプラッシュ画像が表示されます。
初めて起動する場合はワークスペースの設定を促されますが、デフォルトの場所もしくは好きな場所で大丈夫です。

{% img /images/gcube/TryAndroid/adt-splash.png 'splash' 'splash' %}

## 各種設定

ADTを起動した時点で、いくつかの設定を行います。

`ADT` > `詳細設定` > `Android` > `NDK` と開いて自分のNDKのパスを設定します。

{% img /images/gcube/TryAndroid/adt-preferences-ndk.png 'p-ndk' 'p-ndk' %}

GCubeのデフォルトの設定で使用するのでAndroid SDK ManagerでAPI 7のパッケージをインストールします。

{% img /images/gcube/TryAndroid/adt-sdkmanager-7.png 'manager' 'manager' %}

## NDKのバグ修正

`nkd-r8e`の`ndk-build clean`にはバグがあるようです。

[Error on ndk-build clean][clean]

記事の通り、
	$(ndk-dir)/build/core/build-binary.mk
というファイルの49行目
	$(cleantarget): PRIVATE_CLEAN_FILES := ($(my)OBJS)
を
	$(cleantarget): PRIVATE_CLEAN_FILES := $($(my)OBJS)
と修正してください。

## GCubeプロジェクトの導入

GCubeのプロジェクトをADTに読み込ませます。
現在指定しているワークスペースに解凍したGCubeのプロジェクトを配置して下さい。

配置した状態でADTの`File` > `Import` > `Existing Android Code Into Workspace`を開いて、`Root Directory`に
	(GCubeのパス)/platforms/Android
を入力します。

{% img /images/gcube/TryAndroid/adt-import.png 'import' 'import' %}

このAndroidフォルダが普通のAndroidアプリのプロジェクトフォルダになっています。これでFinishを押すとプロジェクトがインポートされます。

次はこのプロジェクトに対するNDKのビルドに関する設定を行なっていきます。

プロジェクトを右クリックして、`Properties` > `C/C++ Build`を開き、`Build Command:`のパスを
	(ndkのパス)/ndk-build
に変更します。

{% img /images/gcube/TryAndroid/adt-preferences-CBuild.png 'c-build' 'c-build' %}

続いて`C/C++ General` > `Paths and Symbols`を開きます。`Languages`の部分に`Assembly`、`GNC C`、`GNU C++`とあります。この各言語ごとに設定されている、
	/Application/android-sdk/ndk/platforms/android-9/arch-arm/usr/include
の部分を全て
	(ndkのパス)/platforms/android-9/arch-arm/usr/include
に変更します。

{% img /images/gcube/TryAndroid/adt-preferences-Paths-and-Symbols.png 'path' 'path' %}

さらに`C/C++ General` > `Indexer` を開いて、`Enable project specific setting`にチェックを入れ、`Allow heuristic resolution of includes`の`Skip files larger than:`を`8MB`から`32MB`まで増やします。
この値を増しておくことでcppファイルのコンパイルエラーが出にくくなります。

{% img /images/gcube/TryAndroid/adt-preferences-indexer.png 'indexer' 'indexer' %}

## ソースコード修正

NDKのパスに依存する部分を修正する必要があります。

`jni/mk/bullet.mk`の7行目

	SYSROOT := (your_ndk_dir)/platform/android-9/arch-arm

ここで`(your_ndk_dir)`となっている部分に自分の環境のNDKのパスを入力してください。

## 実行

ここまで来たならば後は普通に実行するのみです。「Hello World!」という文字が大きく表示されると思います。

{% img /images/gcube/TryAndroid/sc-hello.png 'hello' 'hello' %}



# 関連リンク




[github]:https://github.com/gclue/GCube "github"
[sdk]:http://developer.android.com/sdk/index.html#download
[ndk]:http://developer.android.com/tools/sdk/ndk/index.html
[clean]:http://stackoverflow.com/questions/15982658/error-on-ndk-build-clean
