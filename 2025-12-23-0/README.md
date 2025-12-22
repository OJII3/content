---
title: Unity のカットインのカメラワークを Blender で作る
date: 2025-12-23
tags:
  - blender
  - unity
---
[農工大アドベントカレンダー2025](https://qiita.com/advent-calendar/2025/tuat)23日目の記事です。

ちょっと時間がなくて書く予定だった事が書けなくなったので Blender の話でお茶を濁します。

Blender素人なので、もっといいやり方があったら教えて下さい(全然ありそう)。

## 背景

学祭に向けてUnityでゲームを作っており、キャラクターの入場カットインを担当していました。

今まで、全てのカメラワークは Unity の Cinemachine (と Timeline) で作っていて、キャラクターアニメーションのみ Blender 製でした。

というのも、Cinemachine が大変便利で優秀で、Blenderのカメラなんて初めに消してしまうだけのものだと思っているような私には Cinemachine の方が快適に思えたからです。

## Blenderに寄せた方がいいのでは？

しかし、カメラを直してアニメーションを直して……とUnityとBlenderで往復してるうち、カットインは全部 Blender で作った方が良いのではないかと薄々感じ始めました。

というわけで、Unity に移せるカメラワークを作ろうという流れになりました。

## Blender でのカメラワークの作り方の検討

過去に適当にやったときは、カメラをパスに載っけて自動的にスライドさせ、カメラの注視点をキーフレームで移動させました。

この方法だと、fbx出力してもカメラの動きはアニメーションデータとしてのらないので、Unityに持ち出す事ができません。

## リグアニメーションにする

一旦カメラごとUnityに持ち出すという考えをやめました。

キャラクターのアニメーションと同じように、ボーンをアニメーションさせてfbxにしてしまえば扱いやすそうです。

ただ、やっぱりBlenderのカメラを使いこなすには知識不足なので、カメラの座標と、カメラの注視点の座標のアニメーションデータを作成することにし、実際に注視させる設定は Unity に持っていってから Cinemachine で行うことにします。

## 実際に作ったリグ

こんな見た目です。

![camera rig](./0001-0600.gif)

このオブジェクトの中身は2つだけです。

- 注視点用のボーン(緑色)
- カメラ用のボーン(赤色)

扱いやすいように、ボーンはそれぞれ回転しないようなコンストレイントを入れ、メッシュで見た目を変えています。
下に見えるのはキーフレームで、あるフレームでの2つのボーンの座標が記録されており再生時には線形に補完されます。

このオブジェクトを fbx 形式で出力し、 Unity のシーンに配置します。
カメラ用のボーンオブジェクトの子要素に `Cinemachine Camera` を追加し、`Tracking Target` には注視点用のボーンオブジェクトを指定します。

![unity camera rig](./unity-camera-rig.png)

あとは、Timeline などで、fbx のアニメーションを再生するだけです。

![unity timeline](./unity-timeline.png)

## 完成

キャラクターのアニメーションと合わせると Blender ではこんな感じになります。

![camera rig anim](./0001-0250.gif)

最終的に完成したゲーム内ではこんな感じです。Timeline に多少エフェクトが追加されたりしています。

<iframe width="560" height="315" src="https://www.youtube.com/embed/TwIQHk3sJeo?si=6bVJsnoRsKQRAi1c" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

ゼ○ゼロっぽいカットイン。(かっこいいので作りたかった)

## まとめ

Blender でカメラワークを作り、Unity に持ち出す方法を紹介しました。あと作ったものを見せびらかしました。
