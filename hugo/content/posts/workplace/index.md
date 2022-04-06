---
title: "GitLab・Mattermost・Jitsiをセルフホスティングして君だけのConvDev環境を構築しよう"
subtitle: "GitLab/Mattermost/Jitsi hosting for your awesome workspace"
date: 2022-04-01T00:00:00+09:00
lastmod: 2022-04-01T00:00:00+09:00
publishDate: 2022-04-01T00:00:00+09:00
#author: "okmt"
#authorLink: "okmtLink"
#description: "okmtDesc"
license: "MIT"
draft: false

tags: [Onpremise, Hosting, GitLab, Mattermost, Jitsi, Git]
categories: [Documentation]
featuredImage: "/workplace/images/gitlab.png"
featuredImagePreview: "/workplace/images/gitlab.png"

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
mapbox:
  accessToken: ""
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---

## ConvDev

私が一年ほどソフトウェアエンジニアをやっていて、素晴らしいなと思う考え方の一つにConvDev (Conversational Development) というものがあるのでそちらから紹介したい

{{< admonition type=tip title="ConvDev" open=true >}}
Conversational Development (ConvDev) is a natural evolution of software development that carries a conversation across functional groups throughout the development process, enabling developers to track the full path of development in a cohesive and intuitive way. ConvDev accelerates the development lifecycle by fostering collaboration and knowledge sharing from idea to production.
{{< /admonition >}}

簡単にまとめると以下のようになる

{{< admonition type=tip title="ConvDev (日本語)" open=true >}}
ConvDevは開発プロセスフロー全体において、グループメンバー同士のコミュニケーションを重視してアプリケーション開発を行う開発スタイルのことで、アイデアからプロダクションまでのコラボレーションと知識共有を促進することで、開発ライフサイクルを加速することを目指している。
※ 「組織において一緒に協力しあい、作り上げるチーム開発体制」のことをコラボレーションと呼ぶ
{{< /admonition >}}

ただ、厳かに定義はしてみたもののここではConvDevが目的達成のために理にかなった自然な開発スタイルであることを強調したい

私たちが他人と同じ目的に向かって行動するときは達成目標や課題感を話し合い、
密にコミュニケーションを認識のズレを防ぐ行動を取るはずだし、
信頼がなければ仕事も任せられず、お互い尊重していないと納得して仕事を進めることができないはずである

なので、私が考えるにConvDevは、その性質から「多様性を受け入れ相手を思いやる信頼と尊敬の気持ちがあってこそ成り立つもの」でもあると考えている

## ConvDevを意識したお仕事で必要なもの

私がシステム開発の副業を進めていく上で必要になったツールは大きく分けると以下の3つである

- 音声通話ツール（リアルタイム通話機能、画面共有機能など）
- チャットツール（テキスト交換、簡単なファイル共有、通知機能、権限管理）
- ソースコード管理ツール（成果物共有機能、履歴管理、バージョン管理などいわゆるSCMというやつ）

クラウドサービスに課金してしまったりすると手っ取り早いが、その場合は主にデータの置き場所という点とセキュリティという点で問題があると思う

- データの置き場所: 顧客の同意が得られればどこでも良いが、大概の場合同意は得られないと思う。たとえば顧客データを取り扱う案件を受け入れるといったときに、許可なくクラウドサービスを利用して顧客が知らないサービスのログやデータベースにその情報が登録されてしまうことは企業秘密保持規定に反する場合がある
- セキュリティ: こちらも顧客の同意が得られればなんでも良いが、クラウドサービスの場合、セキュリティが担保されているかどうかは利用するサービスに委ねられる 

このようにクラウドサービスを利用してその影響を懸念を気にするより、コントロールできる範囲に持ってくるのが望ましいと思うので、（規模にもよるが）まるっと解決したい場合は自分のネットワーク内でOSSを使ってサクッと構築してしまうのが良い

今回、音声通話サービスはJitsi、チャットツールはMattermost、ソースコード管理はGitLabを構築してみたので、皆さんの仕事場に導入する際の検討の一助になればと思い、それぞれについてまとめてみた

### Jitsi

複数人での通話や映像共有などが可能な音声通話プラットフォーム

{{< figure src="/workplace/images/jitsi.png" >}}

[Jitsi](https://jitsi.org/)

必要スペック
- CPU: 1 Core
- Memory: 2 GB
- Disk: 20 GB

例えばZoomだと無料版には制限があり、3人以上で通話しようとすると上限40分で一度接続が切れてしまう

他に、日本人だとLINE通話も選択肢に入ってくる方もいらっしゃるかなと思うがビジネス用途としては個人的な連絡先とも紐づいてしまうLINEは相応しくない

Jitsiはユーザーごとの統計(ネットワークの状態、話している時間の長さ)なども表示してくれるのでそれも面白かった。

あともちろん画面共有や通話の動画収録も可能で、枝葉の機能として簡単なチャットを行うこともできる。

コロナ禍で急激に需要が増え無料で安心して利用できる通話サービスが少ない中、Jistiをホスティングして自由に使えるようにしておくと非常に便利だった

### Mattermost

大規模運用も可能な自由度の高いビジネスチャットツール

{{< figure src="/workplace/images/mattermost.png" >}}

[Mattermost](https://mattermost.com/)

必要スペック（規模によって必要になる傾向がある）
- CPU: 1 Core
- Memory: 2 GB
- Disk: 20 MB

例えばSlackだと無課金であれば古いデータが閲覧できなくなったり、枝葉の機能などで機能制限を受けている状態でしか使えない。

利用中に過去の議論が見えなくなったり、以前に貯めた知見が消えてしまうのは望ましくない。

mattermostはこの次で紹介するGitLabと連携できたり、WebHookやBotなどの対応も実装されていて機能面も豊富な印象

また、複数チーム運営、スマホアプリ版や権限付与が細かい粒度で行えるので中規模以上の会社にも導入できるようなポテンシャルを秘めていると思った

### GitLab

GitHubライクな企業向けDevOpsプラットフォーム

{{< figure src="/workplace/images/gitlab.png" >}}

[GitLab](https://about.gitlab.com/?utm_medium=cpc&utm_source=google&utm_campaign=brand_apac_rtg_rsa_br_exact&utm_content=homepage_digital_x-rtg_english_&_bt=414669289199&_bk=gitlab&_bm=e&_bn=g&_bg=92195234843&gclid=CjwKCAjwopWSBhB6EiwAjxmqDcxwlgWPqb2Aqt57LWxgRjbR0AzEFPBiEMSLnCVd7DrHMUo4sHJbfhoCU74QAvD_BwE)


必要スペック（規模によって必要になる傾向がある）
- CPU: 1 Core
- Memory: 4 GB
- Disk: 2.5 GB

[GitLab 日本語情報サイト](https://www.gitlab.jp/)

例えばGitHubも昔と比べればかなり無料で使えるものが増えたが、個人用、オーガナイゼーション用いずれも制限された機能しか使えない。

GitHubとGitLabはお互いに影響を受けながら開発を進められているそうなので、GitHubを使ったことある方なら自然とGitLabの使い方もわかると思う。

GitLabも適切な権限管理やソースコードレビュー機能やissue、マイルストーン、メモ機能なども豊富で使い勝手はGitHubと変わらない

昨今のDevOpsの潮流も味方して徐々に知名度を上げている印象

{{< figure src="/workplace/images/devops.png" >}}

GitLabはDevOpsを強力にサポートしているが、これについては概要レベルの内容を超えてくるのでここでは取り上げない

## 最後に

3つのOSSについて紹介した。

私は自分のネットワーク内やAWS EC2などにホスティングして活用している。

是非自分の仕事環境を作っていく必要があるときはおすすめされたい。

そしてConvDevという考えに共感していただき、広めていただけると嬉しいです。
