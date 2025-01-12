---
layout: default
title: メタボロミクス・質量分析インフォマティクスのための環境構築
description: メタボロミクス・質量分析インフォマティクスのためのデータ解析・プログラミング環境に関する解説
lang: ja_JP
---


# 質量分析インフォマティクスのための環境構築

このページでは質量分析データ（主にメタボロミクス）をインフォマティクスで解析するために有益な情報に関して、主に解析環境に関して記述します。



## はじめに
質量分析装置にはたいていベンダーが作成したソフトウェアが付いてくると思います。分析データを閲覧するためのviewerや、定量解析をするための解析ツールや化合物同定のツールもあると思います。
それらは使いやすい一方で、データ解析の自由度が必ずしも高くなかったり、実際のデータ処理の中身が分かりずらかったりと、必ずしも使っていて”楽しい”ものではないかもしれません。
また、インストールできるPCの数がライセンスで決まっていたりと、実用的・実務的なな問題があるかもしれません。

従来の特定のターゲット化合物だけを定量する、というようなアプリケーションではそのようなツールでも問題はなかったかもしれませんが、スループット性が上がりノンターゲット分析の重要度が増している近年では、膨大な分析データを目的に応じて処理・解析する上でベンダーのソフトウェアでは対応しきれない場面というのが出てきていると思います。

このような背景から、質量分析データを扱い・解析する上で、ベンダーのソフトウェアに縛られず、自由な解析を行うための環境構築を独自で行うことが重要であると考えています（勧めています）。

このチャプターではそのような独自の解析用の環境構築のヒント・きっかけになるような情報を提供することを目的としています。
また、ここで紹介する環境・ツール・言語は管理者の経験や環境に基づいてお勧めするものであり、誰にとってもベストなものというわけではありません。
独自の解析環境の構築が意外と簡単であること、興味が湧いたときにとりあえず手を付けてみるための情報、といったものの提供が目的になります。


## プログラミングかワークフローか
データ解析環境、というのは非常にあいまいな表現です。コンピューターでデータを解析する方法としては大体以下の2つが真っ先に思いつくかもしれません。

- (A) データを既存のソフトウェア・ツールで処理し、出力を他のソフトウェアで処理
- (B) プログラミングにより、データ処理プロセスを一から記述する（コーディング）

Aは例えばベンダーのソフトウェアの出力をエクセルで処理する、といった流れです。もちろんAとBの組み合わせもあり得ます。
このAとBの間には大きなプログラミングという大きな壁が存在していて、それが現在いわゆる質量分析インフォマティクス人口が少ない一因ではないかと思います。

それに対して、第３の選択肢としては (C)ワークフロー（パイプライン）作り   が挙げられると思います。
バイオインフォマティクス分野ではGalaxyというシステム、より一般的なデータサイエンス分野ではKNIMEが有名です。
簡単に言ってしまうと、元のデータに対して様々な処理法がモジュールとして存在しており、それをつなげることで複雑なデータ処理を自動化するというしくみです（モジュールをつなげるという意味ではパイプライン作りという呼び方がしっくりきます）。プログラミングというと黒い画面に長いコードを打ち込んで・・・というイメージをお持ちの方が多いかもしれませんが、このようなワークフロー作りではGUI上でモジュール間を線でつなげるだけでOKです。

# どっちを選ぶ？
コーディングによるプログラミングととワークフロー作り、どちらを選ぶかはなかなか難しい問題ではあります。

- コーディングとかプログラミング言語なんて無理 or GUIがないと頭が働かない　=>　(C)ワークフロー作り 
- プログラミングに興味がある　=>　(B)コーディング 
- 長期的にインフォマティクス関係も取り組みたい　=>　(B)コーディング
- ちょっとしたデータ処理を自動化してみたいだけ　=>　(C)ワークフロー作り 

ただ、ワークフロー作りは視覚的に分かりやすくはあるのですが、各モジュールの仕様が分かりにくく使用しにくいケースがあったり、
ループ・分岐構造が直感的に分かりにくいことがあったり、と実際にやや複雑なデータ処理フローをつくるときに寧ろ難しいと感じることは結構あります。
また、一般的なプログラミング言語であれば分析化学以外の多方面でも活用できるケースは多々あると思うので、「技術が身につく」ということになると思います。


## 質量分析データ解析ライブラリ
ベンダーのソフトウェアからなかなか離れられない一つの原因は質量分析のデータ解析のスタート地点が質量分析装置が吐き出すrawデータ（生データ）であり、ここからデータ読む方法が思いつかない・面倒ということがあるのではないかと思います。これは必ずしも比較定量や未知化合物のアノテーションといった高度な解析ではなく、単にLC-MSデータをTICクロマトグラムで見てみる、スタンダードのXICを書いてみる、という日常的な操作ですらベンダーのソフトウェアがないと出来ない・面倒というふうに考えているかもしれません。逆に、rawデータから容易に自分の欲しい情報を得ることができれば、データ解析の自由度はずっと高くなるはずです。

前置きが長くなりましたが、そのような質量分析データの基本的な操作を行うライブラリがOpenMSになります。（https://www.openms.de/）
各社の装置のRawデータの読み込み自体はベンダーのdllファイルやライブラリが必要にはなりますが、それさえインストールすればrawデータからデータの読み込み、ピーク検出、アライメント、比較定量、といった質量分析によく使われるデータ処理がすべて網羅されています。

ライブラリのコアはC++で書かれていますが、Pythonから直に使えるpyOpenMSがあり、コーディング目的ではPythonから使うのがお勧めです。
また、コーディングを行わずに、前述のワークフローとしてもKNIME, Galaxy上で利用することが可能です。


## プログラミング
近年ではプログラミングの敷居はかなり低くなっているというのは多くの人が感じていると思います。参考書を片手にFortranやCを使ってコーディングしていた時代とは比較にならないほどプログラミングのしやすさ・できることの範囲は広がっています。とはいえ、新しいことをはじめるとなるとなかなか腰が重くなるというのも事実です。また、近年ではプログラミング言語の選択肢が多くなっていることもあり「どれを選べばいいのか」という悩みも出てきていると思います。本ドキュメントの管理者はプログラミングの専門家ではありませんが、質量分析・分析化学という分野で仕事をするという視点からいくつかプログラミング言語に関して触れてみたいと思います。

### Python (https://www.python.org/)
現在プログラミングとして一番目に留まる機会のある言語だと思います。比較的容易で読みやすい書き方をするように設計されていることもあり教育機関でプログラミングの授業で使われることも多いそうです。長所の一つは膨大なライブラリをを容易に使用することができるということにあります。科学演算関連のscipyや、データ・統計解析用のPandas、Numpy、機械学習用のScikitlearnに加え、前述の質量分析データ処理ライブラリのpyOpenMSやケモインフォマティクスライブラリRDKit等一通りのライブラリが整っており、比較的容易に広範囲の仕事をこなすプログラムが書けます。欠点としては処理速度が非常に遅いということが挙げられます。したがって膨大な質量分析データを解析するアプリケーション等にその影響を受けることが考えられます。Cython等の高速化のツールもありますが、Python自体の書きやすさが損なわれるため、とくに初心者にはハードルになってしまうかもしれません。

### Julia (https://julialang.org/)
Juliaは比較的新しい言語であることもあり、容易なコーディングやライブラリ管理などPythonのような現代的な書きやすさ・扱いやすさがあります。Juliaの特徴としてはPythonのような書きやすさと高速な処理を両立させているところが挙げられます。処理内容によりますが、一般的に高速と呼ばれているC言語と同程度のパフォーマンスがあるようです。したがってPythonのような書きやすさとC言語のようなパフォーマンスを両立させたい場合はJuliaは良い選択肢と言えます。欠点としては（主にPythonnと比較して）、ライブラリがPythonほど充実していないということ、またドキュメント・ネット上のリソースがそれほど充実していない点にあります。

### R (https://www.rstudio.com/)
管理者個人的には最近はあまり使っていませんが、科学分野・データサイエンス分野で高い人気を誇る言語で豊富なライブラリがあります。質量分析分野でもRで使う優れたライブラリが多数あり、それらを使うためだけにでも習得する価値があると思います。一方で、単純にエクセルテーブルを読み込んで簡単なデータ解析をするというような使い方であればPythonでも同様な処理がPandas, Numpyを使えば可能です。

### C# (https://docs.microsoft.com/en-us/dotnet/csharp/)
C++やJavaなどと同じ傾向を持つオブジェクト指向言語で、特にWindows用のアプリケーション開発に向いていて高いパフォーマンスを持つようです。Windows用のしっかりしたGUI付きのソフトウェアを開発するときは最適だと思います。質量分析の分野としてはMS-FinderやMS-DIALといった優れたソフトウェアの開発言語であり、そのソースコードを読めるようになるという点でも価値があると思います。

### Java (https://www.java.com/en/)
Javaは2000年代前後に広く普及して、当時のバイオインフォマティクス分野でも広く使われていました、が現在となってはアドバンテージはあまりないように個人的に思えます。


### どれを選ぶ？
どのような仕事をするかどれくらい時間的コストをかけるかによりますが、とりあえず触ってみたいというならPythonで始めてみるのが良いと思っています。そのうえで高いパフォーマンスが必要になったら必要に応じてJuliaを、他のユーザーにも配布するようなGUI付きのアプリケーションを作りたいようなケースはC#を使っていくという流れになると思います。



---
このページに関する質問やデータ解析・解析システム開発のコンサルティング等のお問い合わせは下記リンクのフォームから本ドキュメントの管理者（早川）にご連絡ください。
[問い合わせ](https://docs.google.com/forms/d/e/1FAIpQLSe6AOt0oZvLJeJqJulQ3PcHuT05Lmu0SMUHUM82rRntMgCNmw/viewform?usp=pp_url)
