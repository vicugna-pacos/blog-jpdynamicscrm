---
title: 低速フローのトラブル シューティング
date: 2023-02-03 00:00
tags:
  - Power Automate
  - Cloud flows
---

# 低速フローのトラブル シューティング

こんにちは、Power Platform サポートの瀬戸です。
今回は、「クラウド フローの処理がいつまで経っても終わらない」ときに試していただける改善策をご案内いたします。

<!-- more -->

## なぜ「いつまで経っても終わらない」のか
時間がかかるようなアクションは使っていないはずなのに、今まではすぐに処理が完了していたのに、クラウド フローが何日経っても完了しない。

そんなときは、フローが Power Automate のスループットの制限に抵触した可能性が考えられます。

フローが制限に抵触すると、そのフローは実行中でも低速の状態になります。制限を超えないような速度で実行されるようになるので、数日たってもフローが実行中…ということがよくあります。これでは「手作業でやった方が早いよ」となりかねないので、低速の状態は何としても避けたいところです。

そこで今回は、フローが低速状態となることを避けるために実施いただける改善策をご案内いたします。

■■ 目次 ■■
1. [Power Automate の制限](#power-automate-の制限)

## Power Automate の制限
改善策の前に、Power Automate の制限についてご案内いたします。
Power Automate には、サービスを皆さまへ安定的に提供するために、様々な制限が設けられています。

参考：[制限と構成 - Power Automate | Microsoft Learn](https://learn.microsoft.com/ja-jp/power-automate/limits-and-config)

中でも、抵触することが多いと感じるものに「24 時間あたりのアクション要求数」の制限が挙げられます。

「アクション要求数」とは、大まかに言うと「実行できるアクションの数」です。詳しい数え方については、下記公開情報をご参照ください。(アクション要求数と Power Platform 要求数は数え方が同じです)  

参考：[何が Power Platform 要求と見なされますか?](https://learn.microsoft.com/ja-jp/power-platform/admin/power-automate-licensing/types#what-counts-as-power-platform-request)

「24 時間あたりのアクション要求数」の制限値は、パフォーマンス プロファイルが「低」の場合「10,000」です。
一見それほど厳しい制限に見えませんが、大量の SharePoint リスト、Dataverse のデータ、CSV ファイルなどをフローで扱うとあっという間に抵触してしまいます。
例えば、1,000 行の CSV ファイルを読み込み、Apply to each で行ごとに処理をする場合、Apply to each の内側のアクションが 10 個あるだけで、実行されるアクション要求数が 10,000 を超えてしまいます。

### フローのアクション要求数を数えてみる
まずはじめに、低速状態となったフローのアクション要求数を数えてみることをお勧めいたします。

数え方は、先ほども記載した下記公開情報をご参照ください。
[「何が Power Platform 要求と見なされますか?」](https://learn.microsoft.com/ja-jp/power-platform/admin/power-automate-licensing/types#what-counts-as-power-platform-request)

これで、フローの実行 1 回あたりのアクション要求数が分かります。次に、1 日にこのフローが実行される回数を掛けてみましょう。これでおおよその 24 時間あたりのアクション要求数 が分かります。
実際のアクション要求数は再試行やページングで増えますので、この時点でフローの制限値を超えている場合は、対策が必要です。

## フローの実行回数を減らす
例えば 1 つのフローを 1 時間に 1 回実行している場合、これを 2 時間に 1 回へ減らすだけで、24 時間あたりのアクション要求数が半減します。
1 番簡単に実施できて、最も効果がありますので、実行頻度の削減はぜひご検討ください。

## 繰り返しの回数を減らす
Apply to each や Do until で大量のデータを処理している場合、そのデータ件数を減らすことで、単位時間あたりのアクション要求数を減らせます。
1 度に 1,000 行の データを処理させるのではなく、10 件の データを 1 時間に 1 回処理するよう設計することをお勧めします。

繰り返し処理を避けられない場合は、フローの作り方を工夫してできるだけ繰り返し回数を減らすことができます。
以下に繰り返し回数を減らすための改善案をご紹介いたします。

### フィルター クエリを使う
繰り返しを行う前に対象のデータ件数を減らしておけば、繰り返しの回数を減らせます。
SharePoint コネクタの「複数の項目の取得」アクションなど、フィルター クエリを指定できるアクションでは、できる限りフィルタークエリを使用して、不要なデータを取得しないようにしましょう。

※ フィルター クエリを使った例  
![](troubleshoot-throttling-flow/image06.png)

参考：

* SharePoint
  * [フィルター クエリの例](https://learn.microsoft.com/ja-jp/sharepoint/dev/business-apps/power-automate/guidance/working-with-get-items-and-get-files#filter-queries)
  * [SharePoint REST サービスでサポートされる OData クエリ演算子](https://learn.microsoft.com/ja-jp/sharepoint/dev/sp-add-ins/use-odata-query-operations-in-sharepoint-rest-requests#odata-query-operators-supported-in-the-sharepoint-rest-service)
* Excel
  * [「フィルタリング機能」の行に、サポートされる OData クエリの記載があります](https://learn.microsoft.com/ja-jp/connectors/excelonlinebusiness/#known-issues-and-limitations-with-actions)
* Dataverse
  * [行を一覧にする アクションの使用例](https://learn.microsoft.com/ja-jp/power-automate/dataverse/list-rows#filter-rows)
  * [Web API の観点での OData クエリの例](https://learn.microsoft.com/ja-jp/power-apps/developer/data-platform/webapi/query-data-web-api#%E7%B5%90%E6%9E%9C%E3%81%AE%E3%83%95%E3%82%A3%E3%83%AB%E3%82%BF%E3%83%BC)


### アレイのフィルター処理 を使う
フィルタークエリを使えない場合は、代わりに「データ操作」コネクタの「アレイのフィルター処理」アクションを使って繰り返しを行う前にデータ件数を減らすことができます。

※ アレイのフィルター処理を使った例  
![](troubleshoot-throttling-flow/image08.png)

その他にも、処理が異なるリストをあらかじめ 2 つに分けておく、といった活用もできます。繰り返し処理内の「条件」アクションが必要なくなるので、アクション要求数の削減になります。

※ 処理が異なるリストを分けておく例  
![](troubleshoot-throttling-flow/image09.png)

参考：[Power Automate でデータ操作を使用する (ビデオを含む) - Power Automate | Microsoft Learn](https://learn.microsoft.com/ja-jp/power-automate/data-operations#use-the-filter-array-action)

### 集計に XPath を活用する
例えば SharePoint リストの特定の列の合計を求めたい場合、Power Automate には配列を 1 度に集計するようなアクションや関数が無いため、繰り返し処理で集計する必要があります。

しかし、Power Automate の関数、xml と xpath を活用すると繰り返し処理無しで集計ができます。
つまりアクション要求数の削減になります。

詳しい実現方法は以下の記事で解説しておりますので、ご参照ください。

■■■記事へのリンク■■■

## CSV ファイルの利用を避ける
アクション要求数を削減する観点では、フローでの CSV ファイルの読み込みは避けることをお勧めいたします。
Power Automate には CSV ファイルを 1 つのアクションで読み込むような機能はご用意がございません。
そのため、CSV ファイルを読み込む処理を作りこむ必要があります。

参考：[Power Automate で CSV ファイルを取り込む | Japan Dynamics CRM & Power Platform Support Blog](/blog/powerautomate/Import-Csv-With-Standard-Connectors/)

上記記事の例からも分かる通り、変数を用意して…条件分岐して…とやっているだけでアクション要求数を消費してしまうので、アクション要求数を削減したい場合は、できるかぎり SharePoint リスト、Dataverse をはじめとしたデータベース、Excel ファイルなどの利用をお勧めします。



## 待機を入れて実行を遅くする。
　→スケジュールトリガー(1時間に1回など)や自動トリガーが高頻度で起動する場合は効果が薄そう）
　→コネクタの制限とか、短時間あたりの制限を避けるには効果がある。



## 最後に