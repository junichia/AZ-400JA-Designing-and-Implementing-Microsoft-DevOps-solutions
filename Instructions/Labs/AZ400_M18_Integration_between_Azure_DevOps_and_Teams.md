---
lab:
    title: 'ラボ 18: Azure DevOps とチーム間の統合'
    module: 'モジュール 18: システム フィードバックのメカニズムの実装'
---

# ラボ 18: Azure DevOps とチーム間の統合
# 受講生用ラボ マニュアル

## ラボの概要

**[Microsoft Teams](https://teams.microsoft.com/start)** は、Office 365 でのチームワークのためのハブです。チームのあらゆるチャットやミーティング、ファイル、アプリを 1 箇所で管理して使用できます。Office 365 と Azure DevOps 全体でチーム、会話、コンテンツ、ツールのハブをソフトウェア開発チームに提供します。

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装します。

> **注**: **Azure DevOps Services** を Microsoft Teams と統合すると、開発サイクルを通して包括的なチャットとコラボレーションを体験できます。チームは Azure DevOps チーム プロジェクトで、作業項目、プル要求、コードのコミット、ビルドおよびリリース イベントに関する通知とアラートにより、重要なアクティビティについて容易に情報を得られます。

## 目標

このラボを完了すると、次のことができるようになります。

- Microsoft Teams と Azure DevOps を統合する
- Azure DevOps Kanban ボードと Teams のダッシュボードを統合する
- Azure Pipelines と Microsoft Teams を統合する
- Azure Pipelines アプリを Microsoft Teams にインストールする
- Azure Pipelines 通知をサブスクライブする

## ラボの所要時間

-   推定時間: **60 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Microsoft Teams。このラボでは前提条件の一部としてインストールされます。

### 演習 0: ラボの前提条件の構成

#### Office 365 サブスクリプションを設定 

現在使用しているマイクロソフトアカウントを使用して、[Microsoft Teams サインアップ ページ](https://teams.microsoft.com/start) から無料のトライアル サブスクリプションを作成します。 

    >**For work and organization** を選択してください。
    >**企業名は架空の企業名で大丈夫です**
    
既にこのマイクロソフトアカウントで無料トライアルを使用済みの場合は、新たにマイクロソフトアカウントを作成してください。

#### Azure DevOps 組織を設定します。 

新たなマイクロソフトアカウントを作成した場合は、この作業を行ってください。それ以外の方は、次のステップに進んでください。

[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) の手順に従います。Azure DevOps 組織を作成する際は、Office 365 サブスクリプションの設定で使用したものと同じユーザー アカウントを使ってサインインします。

> **注**: Office 365 サブスクリプションと Azure DevOps 組織は、同じ Azure Active Directory (Azure AD) テナントを共有する必要があります。


この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートおよび Microsoft Teams で作成したチームに基づいて事前に構成された **Tailwind Traders** チーム プロジェクトから成っています。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Tailwind Traders** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。

1.  必要な場合は、「**Azure DevOps Demo Generator**」 ページで 「**承諾する**」 をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。

1. **Azure DevOps Demo Generator now supports GitHub forking!** でチェックボックスをチェックして、**Authorize** をクリックし、要求されたら **GitHub** アカウントで認証します。

1. 「**新しいプロジェクトの作成**」 ページで、欠落している拡張機能をインストールするよう指示されたら 「**ARM 出力**」 の下にあるチェックボックスを選択し、「**プロジェクトの作成**」 をクリックします。。

    > **注**: プロジェクトの作成途中でエラーが発生します（右側に**View Diagnostics**と赤く表示されます）になることがありますが、これは無視してください。必要なぷろじぇくとやリポジトリ、パイプラインは全て作成されています。
    
    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Microsoft Teams でチームを作成する

このタスクでは、Microsoft Teams でチームを作成します。

1.  ラボコンピューターに Teams がインストールされていない場合は、ラボのコンピューターで Web ブラウザーを起動して [Microsoft Teams ダウンロード ページ](https://www.microsoft.com/ja-jp/microsoft-365/microsoft-teams/download-app) に移動します。そこから既定の設定で Microsoft Teams をダウンロードしてインストールします。 

1.  ラボのコンピューターでデスクトップ アプリを使い、**Microsoft Teams** を起動します。

    > **注**: Web ブラウザーを使用して [Microsoft Teams 起動ページ](https://teams.microsoft.com/dl/launcher/launcher.html?url=/_%23/l/home/0/0&type=home) に移動することもできます

1.  サインインするよう指示されたら、Teams のライセンスを持ち、Azure DevOps 組織へのアクセス権限がある Microsoft Account 使ってサインインします。

1.  Microsoft Teams のページ左側のツールバーで 「**Teams**」 をクリックし、チーム一覧の最下部で 「**チームに参加、またはチームを作成**」 をクリックします。

    > **注**: チームは、共通の目標を掲げて集まる人々のグループです。 

1.  「**チームに参加、またはチームを作成**」 ペインで 「**チームを作成**」 をクリックします。

1.  「**チームを作成**」 パネルで 「**最初から**」 をクリックします。「**チームの種類は何でしょうか?**」 パネルで 「**プライベート**」 をクリックします。

1.  「**プライベート チームに関する簡単な詳細**」 パネルで 「**チームに名前を付ける**」 を 「**Tailwind Traders**」 に置き換えて 「**作成**」 をクリックします。

1.  「**Tailwind Traders にメンバーを追加**」 パネルで 「**スキップ**」 をクリックします。

### 演習 1: Azure Boards に Microsoft Teams を統合する

この演習では、Azure Boards と Microsoft Teams 間で統合を実装します。

#### タスク 1: Microsoft Teams で Azure Boards アプリをインストールして構成する

このタスクでは、Microsoft Teams で新しく作成されたチームで Azure Boards アプリをインストールして構成します。

1.  Microsoft Teams ウィンドウの左下コーナーで 「**アプリ**」 アイコンをクリックします。「**アプリ**」 ペインが開きます。

1.  「**アプリ**」 ペインの 「**すべてのアプリを検索**」 テキストボックスで「**Azure Boards**」と入力し、アプリ一覧で 「**Azure Boards**」 をクリックします。

1.  「**Azure Boards**」 パネルで 「**追加**」 をクリックします。

1.  「Microsoft Teams」 ウィンドウの最上部にある 「**Azure Boards**」 ドロップダウン リストで 「**Work Items**」 をクリックします。「**Azure Boards**」 ペインが表示されます。

1.  「**Azure Boards**」 パネルで 「**サインイン**」 リンクをクリックします。その後、**承諾** します。

1. **Azure Boards** で **設定** リンクをクリックします。

1. **Organization** を選択し **Continue** をクリックします。

1. **Project** で **Tailwind Traders** を選択し、**Continue** をクリックします。

    > **注**: **Tailwind Traders** Azure DevOps プロジェクトで既存の作業項目のリストが表示されます。

1.  作業項目のリストをスクロールして、いずれかを選択し、作業項目名をクリックします。

1.  作業項目のリンクをクリックすると新しい Web ブラウザー ウィンドウが自動的に開き、Azure DevOps ポータルに作業項目の詳細が表示されます。

1. 右側のメニューから「･･･」をクリックし、アプリの一覧から Azure Boards を検索して選択します。

1. ポップアップが開いたら、「開く」の横の下向きのキャレットをクリックし、**チームに追加** を選択します。

1. ＊＊Tailwind Traiders** チームを選択し、「**ボットを設定**」をクリックします。

1.  **Tailwind Traders** チームの 「**一般**」 チャネルの投稿リストで、以下のようなボットに投稿されたメッセージを確認します:

    ```
    Sign in to your Azure Boards account with: @Azure Boards signin
    To see what else you can do, type @Azure Boards help
    ```
1.  **Tailwind Traders** チームの 「**一般**」 チャネルの投稿リストで、「**Azure Boards**」というメッセージを投稿し、**Enter** キーを押します。ボットから投稿された追加メッセージをレビューします:

    ```
    link [project url] - Link to a project to create work items and receive notifications
    subscriptions - Add or remove subscriptions for this channel
    addAreapath [area path] - Add an area path from your project to this channel
    signin - Sign in to your Azure Boards account
    signout - Sign out from your Azure Boards account
    unlink - Unlink a project from this channel
    feedback - Report a problem or suggest a feature
    ```
1. **一般** チャネルで、新しい投稿をクリックすると、メッセージウィンドウの下に表示されるアイコンの右側に **Azure Boards** アイコンが表示されます。この画面で、Boards の内容を検索し、メッセージとともに投稿することができます。

#### タスク 2: Azure Boards Kanban ボードを Microsoft Teams に追加する

このタスクでは、Azure Boards Kanban ボードを Microsoft Teams のタブに追加します。

> **注**: かんばんボードは、バックログをインタラクティブな看板に変え、作業の視覚的なフローを提供します。作業が構想から完了まで進むにつれて、ボード上の項目を更新します。各列は作業段階を示し、各カードはその作業段階のユーザーのストーリー (青いカード) またはバグ (赤いカード) を表します。タブを使用することで、チームのかんばんボードまたはお気に入りのダッシュボードを直接、Microsoft Teams に取り込むことができます。**タブ** を使うと、チーム メンバーは専用のキャンバスで、チャネル内で、またはユーザーの個人的なアプリ スペースでサービスにアクセスできます。既存の Web アプリを利用して、Teams 内ですばらしいタブ経験を創造できます。

1.  ラボのコンピューターで、Azure DevOps ポータルで **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替えます。Azure DevOps ポータルの一番左にある垂直メニュー バーで 「**ボード**」 をクリックし、「**ボード**」 セクションで 「**ボード**」 をクリックします。

1.  「**ボード**」 パネルの 「**My team boards (1)**」 セクションで、「**Tailwind Traders Team boards**」 エントリをクリックします。 

1.  「**Tailwind Traders Team**」 ペインが表示されている URL をクリップボードにコピーします。

1.  Microsoft Teams ウィンドウに切り替え、新しく作成されたチーム **Tailwind Traders** の 「**一般**」 チャネルが選択されていることを確認します。

1. 「**一般**」 ペインの上側のセクションでプラス記号をクリックします。「**タブの追加**」 パネルが表示されます。

1.  「**タブの追加**」 パネルで 「**Web サイト**」 をクリックします。「**Web サイト**」 パネルで 「**タブ名**」 を 「**Tailwind Traders Team Board**」 に設定し、「**URL**」 を URLフィールドに貼り付けて「**保存**」 をクリックします。

1.  「Microsoft Teams」 ウィンドウで **Tailwind Traders** チームの 「**一般**」 チャネルを選択し、トップ メニューのタブ一覧で新しく追加された 「**Tailwind Traders Team ボード**」 タブをクリックします。Azure DevOps ポータルで利用できる **Tailwind Traders Team** ボードに一致する内容がここに含まれていることを確認します。

> **注**: 毎日の立ち上げ中にすべての作業を監視できます。該当する作業項目の状態が変化した場合、更新はリアルタイムで反映されます。かんばんボードを Microsoft Teams から変更するオプションもあります。

### 演習 2: Azure Pipelines と Microsoft Teams を統合する

この演習では、Azure Pipelines と Microsoft Teams の統合を実装します。

#### タスク 1: Microsoft Teams で Azure Pipelines アプリをインストールして構成する

このタスクでは、Microsoft Teams で指定されたチームで Azure Pipelines アプリをインストールして構成します。

> **注**: Microsoft Teams の Azure Pipelines アプリを使うと、パイプラインのイベントを監視できます。通知を直接、Teams チャネルに投稿することで、リリースや保留中の承認、完了したビルドといったイベントのサブスクリプションを設定して管理できます。また、Teams チャネル内からリリースを承認することも可能です。

1.  Microsoft Teams ウィンドウの左下コーナーで 「**アプリ**」 アイコンをクリックします。「**アプリ**」 ペインが開きます。

1.  「**アプリ**」 ペインの 「**すべてのアプリを検索**」 テキストボックスで「**Azure Pipelines**」と入力し、アプリ一覧で 「**Azure Pipelines**」 をクリックします。

1.  「**Azure Pipelines**」 パネルで、「**追加**」 ボタンのすぐ右にある下向き矢印をクリックします。ドロップダウン リストで 「**チームに追加**」 エントリをクリックします。

1.  「**チームのために Azure Pipelines を設定**」 パネルの 「**検索**」 テキスト ボックスに「**Tailwind Traders**」と入力します。結果リストで **「Tailwind Traders」 > 「一般」** エントリを選択し、「**ボットを設定**」 をクリックします。自動的に **Tailwind Traders** チームの **一般** チャネルの投稿ビューにリダイレクトされます。

1.  **Tailwind Traders** チームの 「**全般**」 チャネルの投稿リストで、以下のようなボットに投稿されたメッセージを確認します:

    ```
    Subscribe to one or more pipelines or all pipelines in a project with: @Azure Pipelines subscribe [pipeline url/ project url]
    To see what else you can do, type @Azure Pipelines help
    ```

1.  **Tailwind Traders** チームの 「**一般**」 チャネルの投稿リストで、「**Azure Pipelines**」を入力して投稿すると、ボットから返信が投稿されます。

    ```
    subscribe [pipeline url/ project url] - Subscribe to a pipeline or all pipelines in a project to receive notifications
    subscriptions - Add or remove subscriptions for this channel
    feedback - Report a problem or suggest a feature
    signin - Sign in to your Azure Pipelines account
    signout - Sign out from your Azure Pipelines account
    ```
   
#### タスク 2: Microsoft Teams で Azure Pipelines 通知にサブスクライブする

このタスクで、Microsoft Teams で Azure Pipelines 通知にサブスクライブします

> **注**: `@Azure Pipelines` ハンドルを使用して、アプリの操作を開始できます。

1.  「**Tailwind Traders**」 タブの 「**一般**」 チャネルで、**新しい投稿** をクリックすると、Pipeline アイコンが表示されていることがわかります。これをクリックします。

1.  **サインイン** をクリックして、サインインし、**承認** をクリックします。

1. **設定** リンクをクリックし、Organization と Project **Tailwind Traiders** を選択し **Continue** をクリックします。


    > **注**: これで `@azure pipelines subscribe [pipeline url]` コマンドを実行して Azure DevOps パイプラインにサブスクライブできます。

1.  ラボのコンピューターで、Azure DevOps ポータルの **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで 「**パイプライン**」 をクリックします。「**パイプライン**」 ペインで 「**Website-CI**」 エントリをクリックします。

1.  「**Website-CI**」 が表示されている URL をクリップボードにコピーします。

    > **注**: URL は、`https://dev.azure.com/<organization_name>liveid2530565/Tailwind%20Traders/_build?definitionId=6` の形式になります。`<organization_name>` は、DevOps 組織の名前を示すプレースホルダーです。

    > **注**: パイプライン URL は、*definitionId* または *buildId/releaseId* が URL に含まれているパイプライン内のどのページでも構いません。

1.  Teams に戻り、**Tailwind Traders** チームの 「**一般**」 チャネルで `@Azure Pipelines subscribe コピーしたURL` を投稿し、ビルド パイプラインにサブスクライブします (必ず `<organization_name>` プレースホルダーは DevOps 組織の名前に置き換えてください)。

    >**注** **URL 自動的にページ名に変換されてしまう場合は、一旦メモ帳等に貼り付け、それをコピーしてから Teams に貼り付けてください。**

1.  サブスクリプションが作成されたという通知を待ちます。
 
    > **注**: ビルド パイプラインで、チャネルは 「**Run stage state changed**」 と 「**Run stage waiting for approval**」 という通知にサブクライブします。

1.  ラボのコンピューターで、Azure DevOps ポータルの **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで 「**パイプライン**」 をクリックします。「**パイプライン**」 セクションで 「**リリース**」 をクリックします。リリース一覧で 「**Website-CD**」 エントリをクリックします。「**Website-CD**」 エントリを選択し、Web ブラウザー ウィンドウで URL をクリップボードにコピーします。

    > **注**: URL は、`https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` の形式になります。`<organization_name>` は、DevOps 組織の名前を示すプレースホルダーです。

1.  「**投稿**」 タブを選択し、**Tailwind Traders** チームの 「**一般**」 チャネルで `@Azure Pipelines subscribe コピーしたURL` を投稿し、リリース パイプラインにサブスクライブします (必ず `<organization_name>` プレースホルダーは DevOps 組織の名前に置き換えてください)。

    > **注**: リリース パイプラインで、チャネルは 「**Deployment started**」、「**Deployment completed**」、「**Deployment approval pending**」 通知にサブスクライブします。
 
 ### タスク 3: フィルターを使用し、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズする

このタスクでは、フィルターを使い、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズします。

> **注**: ユーザーがいずれかのパイプラインにサブスクライブすると、既定によりいくつかのサブスクリプションが作成され、フィルターは適用されません。ユーザーは頻繁にサブスクリプションをカスタマイズする必要があります。たとえば、ユーザーはビルドが失敗した場合のみ、またはデプロイが運用環境にプッシュされる場合にのみ通知を希望する可能性があります。Azure Pipelines アプリは、チャネルの表示内容をカスタマイズするフィルターをサポートします。

> **注**: `@Azure Pipelines subscriptions` コマンドを使用すると、サブスクリプションを一覧表示した管理できます。このコマンドでは、当該チャネルの現在のサブスクリプションすべてを一覧表示します。

1.  **Tailwind Traders** チームの 「**一般**」 チャネルで `@Azure Pipelines subscriptions` コマンドを投稿します。**Azure Pipelines** ボットからの応答で 「**Add Subscriptions**」 を選択します。

1.  「**Azure Pipelines** **Add Subscription**」 パネルの 「**Choose event**」 ドロップダウン リストで 「**Build completed**」 が選択されていることを確認し、「**Next**」 をクリックします。

1.  「**Azure Pipelines** **Add subscription**」 パネルの 「**Choose pipeline**」 ドロップダウン リストで 「**Website-CI**」 が選択されていることを確認し、「**Next**」 をクリックします。

1.  「**Azure Pipelines** **Add subscription**」 パネルの 「**ビルドの状態**」 ドロップダウン リストで 「**任意**」 が選択されていることを確認し、「**送信**」 をクリックします。
1.  「**Azure Pipelines** **Add subscription**」 パネルで 「**OK**」 をクリックし、確定メッセージを認識します。

1.  「**Azure Pipelines** **View all subscriptions**」 パネルで、サブスクリプションのリストをレビューし、パネルを閉じます。

### 演習 3: DevOps シナリオで Microsoft Teams のコラボレーション機能をレビューする

この演習では、DevOps シナリオにさらなる価値を提供する Microsoft Teams のコラボレーション機能をレビューします。

#### タスク 1: Microsoft Teams 会話機能をレビューする

このタスクでは、基本的な Microsoft Teams の会話機能をレビューします。

## コラボレーション エクスペリエンス

> **注**: Microsoft Teams の投稿機能は、会話に接続して履歴を維持する簡単な手段になります。インタラクティビティを増進する絵文字、ステッカー、GIF のサポートが含まれています。

1.  会話を始めるには、Microsoft Teams ウィンドウで 「**投稿**」 タブをクリックします。

    > **注**: Teams では Azure DevOps アイテムを容易に見つけて参照でき、会話とコラボレーションは Teams アプリ内に維持されます。たとえば、ユーザー ストーリーについて話し合いたい場合は、それを検索して会話に追加し、コメントを入力できます。

1.  投稿エントリのすぐ下にあるアイコンのリストで 「**Boards**」 アイコンをクリックします。「**Azure Boards**」 ポップアップ ウィンドウが自動的に表示されます。

1.  投稿エントリに戻り、作業項目の参照を投稿に追加します。

#### タスク 2: Microsoft Teams でチャネルを作成する

このタスクでは、Microsoft Teams でのチャネル作成プロセスを順番に実行します

> **注**: **チャネル**はチーム内の専用セクションで、好みに応じて特定のトピック、プロジェクト、分野別に会話を整理します。チーム チャネルは、チームの全員があけすけに会話できる場所です。プライベート チャットは、そのチャットの参加者にのみ表示されます。チャネルは、タブやコネクタ、ボットなどのアプリで保管すると最も効果的になります。

1.  「Microsoft Teams」 ウィンドウで、このラボで先ほど作成した **Tailwind Traders** チームを見つけ、右側の省略記号をクリックします。ドロップダウン メニューで 「**チャネルの追加**」 をクリックします。

1.  「**"Tailwind Traders」 チームのチャネルを作成**」 ポップアップ ウィンドウで、「**チャネル名**」 テキストボックスに「**DevOps 投稿**」と入力します。「**説明 (オプション)**」 テキストボックスは空白のままにして、「**プライバシー**」 ドロップダウン リストで 「**標準 - チームの全員がアクセスできます**」 を選択し、「**追加**」 をクリックします。

#### タスク 3: Microsoft Teams でコンテンツを共有する

このタスクでは、Microsoft Teams で Azure DevOps wikis を共有するプロセスを順番に実行します。

> **注**: チームが協力する際、共有して共同作業を行いたいファイルが出てくるはずです。Microsoft Teams では、チャネル内でファイルを容易に共有できます。ファイルが標準的な Microsoft Office アプリケーション (Word、Excel、PowerPoint、Visio など) で作成していっる場合は、Teams 内で直接、ファイルを表示、編集してコラボできます。 

1.  「Microsoft Teams」 ウィンドウで 「**DevOps 投稿**」 チャネルを選択し、「**ファイル**」 タブをクリックします。ツールバーに 「**アップロード**」、「**同期**」、「**ダウンロード**」 項目が含まれていることを確認します。 

    > **注**: 「**ドラッグ アンド ドロップ**」 を使用してファイルをアップロードすることもできます。

    > **注**: さらに、Teams 内で Azure DevOps 常駐コンテンツをタブとして共有するオプションもあります。この中には、プロジェクトの目標やエピック、仕様、リリース ノート、ベスト プラクティスを文書化する Azure DevOps Wikis も含まれます。

1.  ラボのコンピューターで、Azure DevOps ポータルが表示されているブラウザー ウィンドウに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで 「**概要**」 をクリックします。「**概要**」 セクションで 「**Wiki**」 をクリックします。

1.  「**Publish code as wiki**」 をクリックします。

1.  ドロップダウン メニューに **TailwindTraders-Website** が含まれていることを確認します。

1.  「**ブランチ**」 ドロップダウン リストで 「**main**」 を選択し、「**フォルダー**」 の値を「**/Documents**」に設定します。

1.  「**Wiki 名**」 で「**TailWindTraders-Website wiki**」と入力し、「**Publish**」 をクリックします。

1.  「**TailWindTraders-Website wiki**」 ページが表示されている Web ブラウザー ウィンドウで、その URL をクリップボードにコピーします。

1.  Microsoft Teams ウィンドウに切り替え、新しく作成されたチーム **Tailwind Traders** の 「**DevOps 投稿**」 チャネルが選択されていることを確認します。

1.  「**DevOps 投稿**」 ペインの上側のセクションでプラス記号をクリックします。「**タブの追加**」 パネルが表示されます。

1.  「**タブの追加**」 パネルで 「**Web サイト**」 をクリックします。「**Web サイト**」 パネルで 「**タブ名**」 を 「**Tailwind Traders DevOps wikis**」 に設定し、「**URL**」 をクリップボードにコピーしたばかりの URL に設定してから 「**保存**」 をクリックします。

7.  「Microsoft Teams」 ウィンドウで **Tailwind Traders** チームの 「**DevOps 投稿**」 チャネルを選択し、トップ メニューのタブ一覧で新しく追加された 「**Tailwind Traders DevOps wikis**」 タブをクリックします。Azure DevOps ポータルで利用できる **TailWindTraders-Website wiki** wikis に一致する内容がここに含まれていることを確認します。

    > **注**: Microsoft Teams と Azure DevOps の接続が完了したので、以下のような Microsoft Teams で開示できる他の種類の情報について検討します: 

    - [OneNote ノートブックを Teams に追加](https://support.office.com/ja-jp/article/Add-a-OneNote-notebook-to-Teams-0ec78cc3-ba3b-4279-a88e-aa40af9865c2)すると、「**スプリント企画会議**」や「**反省会**」といったミーティングのメモを維持できます。 
    - [Azure DevOps を Power BI に接続](https://docs.microsoft.com/ja-jp/azure/devops/report/powerbi/?view=azure-devops) と [Power BI タブを追加](https://support.office.com/ja-jp/article/add-a-powerbi-tab-to-teams-708ce6fe-0318-40fa-80f5-e9174f841918)では、Azure DevOps または他のプロジェクト関連データからの詳細なレポートが表示されます。

## レビュー

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装しました。
