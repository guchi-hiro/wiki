================================
OpenStackでMachine Learning
================================

この記事は
`OpenStack Advent Calendar 2016 <http://www.adventar.org/calendars/1739>`_
の20日目の記事です。

この度、OpenStack上でMachine Learningをサービスとして提供する
Machine Learning as a Serviceに関する新しいプロジェクトを作りました。

今日はその紹介をしたいと思います。

0.モチベーション
----------------

* 単純に機械学習が流行っているし、興味があったから

  機械学習(特にApache Spark)に関する記事を目にする機会が多くやってみようと思った

* 機械学習をするための環境がOpenStackに既にそろっている
  
  機械学習をやるためには「大きなデータを置く場所」と「大きなデータをさばく場所」が必要
 
  SwiftとSaharaでできる。しかもSaharaはSparkをサポートしている

* OpenStackの適用領域としてBig Dataが普及している

  先日のバルセロナサミットでのuser surveyの結果、Big Dataは適用領域として第5位
 
  １. IaaS, ２.Telecom NFV, ３.Storage/Backup/Archive, ４.Business Application, ５.Big Data

1.名前
--------

プロジェクトをやるには名前を決めないといけません。
日本人が作ったことをアピールするために日本語の名前にしようかと思ったけどいいのが浮かばなかった。。

結局「Meteos」にしました。

「Meteos」は「Meteorologist（気象学者）」と「OS（OpenStack）」を組み合わせた造語です。

2.Meteosがやること
------------------

MeteosはApache SparkをベースとしたMachine Learning as a Serviceです。

Meteosがやることは非常にシンプルで以下の２つです。

**1. 大量のデータから学習する**

**2．学習したデータを元にある値を予測する**

（たとえば過去の気象データから学習し、明日の天気を予想するなど）

大量のデータから学習した際、Meteosは学習モデルというものを作成します。

この学習モデルに対してある値を与えてやると、その値をもとに予測値を返却してくれます。

Inputとしてどういう値を受け付けるか、Outputとしてどういう値を返すかについては学習モデル作成時に決定されます。

3.Meteosの学習モデル
-------------------

（ここは完全に機械学習の話。）

Meteosは現状学習モデルとして以下のものをサポートしています。

Apache Spark自体はもっと多くの学習モデルをサポートしていますが
それらについては今後対応予定です。

* 教師あり学習

  ロジスティック回帰モデル
  
  線形回帰モデル
  
  決定木モデル

* 教師なし学習

  KMeans(クラス分け)モデル
  
  Recommendationモデル
  
  Word2Vecモデル

学習モデルは「教師あり学習」と「教師なし学習」に分けられます

* 教師あり学習とは

 教師あり学習とは、予測したい項目がすでに学習データに含まれているものです。

 たとえば過去の売り上げ情報から、未来の売上高を予測するモデルを作成するとします。
 この時、予測したい項目は「売上高」であり、その項目は学習データに含まれています。
 よって、「休日は売上高が高い」や「雨の日は売上高が低い」など過去の情報から
 予測したい項目の傾向を把握することができます。その傾向から未来の値を予測します。

 売上高のようにある数量を予測したい場合は、「線形回帰モデル」を利用します。
 その他にもある値に対して1か0を予測する「ロジスティック回帰モデル」や「決定木モデル」
 があります。

* 教師なし学習とは

 教師なし学習とは、予測したい項目が学習データに含まれてないものをさします。

 たとえば、ある値を測定するセンサーがあるとします。
 大量のセンサーからの測定値一覧を収集し、どのセンサーが故障しているかを予測するとします。
 この時、予測したい項目は「センサーが故障しているかどうか」であり、その項目は
 学習データに含まれていません。学習データは測定値一覧のみです。

 なので測定値一覧から、このセンサーがとり得る値を想定し、その値の範囲から外れたもの＝故障
 という感じで予測していきます。

 このようにデータを元に対象をクラス分けする場合は「KMeansモデル」を利用します。
 その他に、ユーザの好みを分析して、似たような好みを持つユーザが好むアイテムを薦める
 「Recommendationモデル」や、文字をベクトル（数量）に変換してある文字の類義語を予測する
 「Word2Vecモデル」などがあります。

4.Meteosアーキテクチャ
-----------------------

アーキテクチャは以下の通り。

![Architecture](http://raw.githubusercontent.com/guchi-hiro/wiki/master/Meteos-architecture.png)

機械学習の場を「Experiment」と呼んでます。
このExperimentについてはSaharaのAPIをたたいて作成します。
実体はVMの集合です。

Experimentを作成する前に、Templateを作成します。
Templateには、「Sparkのバージョン（現状1.6のみ）」「マスターノード（VM）数」「ワーカーノード（VM）数」
それらノードのFlavorなどを指定します。

Experiment作成完了後、このExperiment上にSwift上のデータをダウンロードして
SparkのMapReduceを実施して学習モデルを作成してきます。

学習したモデルにあるInputを渡すと、それを元に予測された
Outputを出力するという仕組みです。


5.サンプル集
-------------

以下にDevStackでのインストール方法と
サンプル集をまとめていますので
ぜひ試してみてください。

* `インストール手順 <https://wiki.openstack.org/wiki/Meteos/Devstack>`_

* `売上高を予測するサンプル <https://wiki.openstack.org/wiki/Meteos/ExampleLinear>`_

* `株を買うべきかどうかを予測するサンプル <https://wiki.openstack.org/wiki/Meteos/ExampleDecisionTree>`_

* `スキル毎にユーザをクラス分けするサンプル <https://wiki.openstack.org/wiki/Meteos/ExampleKmeans>`_

* `あるユーザへのお勧め映画を見つけるサンプル <https://wiki.openstack.org/wiki/Meteos/ExampleRecommend>`_

* `類義語を予測するサンプル <https://wiki.openstack.org/wiki/Meteos/ExampleWord2Vec>`_
