---
lab:
    title: 'ラボ 20: SonarCloud および Azure DevOps による技術的負債の管理'
    module: 'モジュール 20: コンプライアンス向けのコード ベースの検証'
---

# ラボ 20: SonarCloud および Azure DevOps による技術的負債の管理
# 受講生用ラボ マニュアル

## ラボの概要

Azure DevOps では、「*技術的負債*」という言葉は戦術的な目標を達成する最適以下の手段を意味します。これは、ソフトウェアの開発・デプロイ分野で戦略的な目標能力に悪影響を与えます。技術的負債は、コードを理解しにくくなる、失敗しやすくなる、変更に時間がかかる、検証が難しいといった点で生産性に影響します。適切な監視と管理を行わなければ、技術的負債は時間とともに蓄積し、長期的にはソフトウェアの全体的な品質と開発チームの生産性に重大なインパクトを与えます。

[SonarCloud](https://sonarcloud.io/){:target="\_blank"} はクラウドベースのコード品質・セキュリティ サービスです。SonarCloud のおもな特徴は以下の通りです:

- 23 のプログラミング言語とスクリプト言語に対応 (Java、JS、C#、C/C++、Objective-C、TypeScript、Python、ABAP、PLSQL、T-SQL など)。
- 強力な静的コード アナライザーに基づき数千のルールで見つけにくいバグと品質の問題を検出。
- 人気の高い CI サービスを使用したクラウド ベースのインテグレーション （Travis、Azure DevOps、BitBucket、AppVeyor など）。
- ブランチとプル要求のあらゆるソース ファイルを調べる詳細なコード分析で、緑のクオリティ ゲートを達成してビルドを促進。
- スピードとスケーラビリティ。

このラボでは、Azure DevOps Services を SonarCloud に統合する方法を学びます。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps プロジェクトと CI ビルドを設定して SonarCloud と統合する
- SonarCloud レポートの分析
- 静的分析を Azure DevOps プル要求プロセスに統合する

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

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[Sonar Scanning Examples リポジトリ](https://github.com/SonarSource/sonar-scanning-examples.git)に基づくチーム プロジェクトで構成されています。

#### タスク 1: チーム プロジェクトを作成する

このタスクでは、[Sonar Scanning Examples リポジトリ](https://github.com/SonarSource/sonar-scanning-examples.git)に基づいて新しい Azure DevOps プロジェクトを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動して [**Azure DevOps ポータル**](https://dev.azure.com) に移動し、 Azure DevOps 組織にサインインします。

1.  **Azure DevOps ポータル**の右上コーナーで 「**+ New project**」をクリックします。 

1.  「**新しいプロジェクトの作成**」ペインで 「**プロジェクト名**」テキストボックスに「**SonarExamples**」と入力します。「**Visibility**」セクションで 「**Public**」をクリックしてから 「**Create**」をクリックします。 
   
    > **注**: SonarCloud の有料プランにサインアップするのでない限り、Azure DevOps プロジェクトがパブリックに設定されていることを確認してください。有料プランに*サインアップ*するのであれば、プライベート プロジェクトを作成できます。

1.  ［**SonarExamples**］ ペインで Azure DevOps ポータルの一番左にある垂直メニュー バーで 「**リポジトリ**」をクリックします。

1.  「**Import Repository**」セクションで 「**Import**」をクリックします。

1.  「**Git リポジトリのインポート**」ペインで、**Git** が 「**Repository Type**」ドロップダウン リストに表示されていることを確認します。

1.  「**クローン URL**」に **https://github.com/SonarSource/sonar-scanning-examples.git** と入力し、「**インポート**」をクリックします。 

    > **注**: スキャン例のリポジトリには、MSBuild を使用する C#、Maven、Java を使用した Gradle など、多数のビルド システムと言語のサンプル プロジェクトが含まれています。

#### タスク 2: Azure DevOps 個人用アクセス トークンを生成する

このタスクでは、Azure DevOps 個人用アクセス トークンを生成します。これは、この演習の次のタスクでインストールする Postman アプリからの認証に使用されます。

1.   ラボのコンピューターを使い、Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで Azure DevOps ページの右上コーナーにある 「**User Settings**」アイコンをクリックします。ドロップダウン メニューで 「**Personal Access Tokens**」をクリックし、「**+ New Token**」をクリックします。

1.  下にある「**Show all scopes**」リンクをクリックし、以下の設定を指定します。「**Create**」をクリックします (他はすべて、既定値のままにします):

     | 設定 | 値 |
     | --- | --- |
     | 名前 | **SonarCloud および Azure DevOps による技術的負債の管理ラボ** |
     | スコープ | **Custom Defined** |
     | スコープ | **コード** |
     | アクセス許可 | **Read & Write** |

1.   「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

     > **注**: 必ずトークンの値を記録してください。このペインを閉じると、取得できなくなります。 

1.   「**成功**」ペインで 「**Close**」をクリックします。

#### タスク 3: SonarCloud Azure DevOps 拡張機能をインストールして構成する

このタスクでは、SonarCloud Azure DevOps 拡張機能を Azure DevOps プロジェクトでインストールして構成します。

1.  ブラウザで [SonarCloud 拡張機能ページ](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud)に移動し、「**Get it free**」をクリックします。ご自分の Azure DevOps 組織の名前が「**Azure Devops 組織の選択**」ドロップダウン リストに表示されていることを確認し、「**インストール**」をクリックします。

1.  インストールが完了したら、「**Proceed to organization**」をクリックします。組織のホーム ページを表示している Azure DevOps ポータルにブラウザーがリダイレクトされます。 

    > **注**: マーケットプレースから拡張機能をインストールする適切な許可がない場合は、インストールをっ承認するよう求めるリクエストがアカウント管理者に送られます。

    > **注**: SonarCloud 拡張機能にはビルド タスク、ビルド テンプレート、カスタム ダッシュボード ウィジェットが含まれています。

1.  Web ブラウザー ウィンドウで **SonarCloud ホーム ページ** [https://sonarcloud.io/](https://sonarcloud.io/) に移動します。

1.  SonarCloud ホーム ページで「**ログイン**」をクリックします。

1.  「**SonarCloud にログインまたはサインアップ**」で「**With Azure DevOps**」をクリックします。 

1.  「**このアプリがあなたの情報にアクセスすることを許可しますか?**」と聞かれたら「**はい**」をクリックします。

    > **注**: SonarCloud で組織を作成し、その内部で新しいプロジェクトを作成します。SonarCloud で設定した組織とプロジェクトは、Azure DevOps で設定した組織とプロジェクトと同じになります。

1.  「**SonarCloud へようこそ**」ページで「**Azure からプロジェクトをインポート**」をクリックします。

1.  「**組織の作成**」ページの「**Azure DevOps 組織名**」テキストボックスにご自分の Azure DevOps 組織の名前を入力します。

1.  「**個人用アクセス トークン**」テキストボックスに、前の演習で記録しておいたトークンの値を貼り付け、「**続行**」をクリックします。 

1.  「**組織の設定をインポート**」セクションの「**キー**」テキストボックスに、ご自分の組織を示す文字列を入力し、「**続行**」をクリックします。

    > **注**: キーは SonarCloud システム内で一意のものでなくてはなりません。「**Key (キー)**」テキストボックスの右側に緑色のチェックマークが表示されていることを確認してください。これは、キーが一意性の前提条件を満たしていることを示唆します。

1.  「**Choose a plan (プランの選択)**」セクションで、このラボで使用したいプランを選択し、「**Create Organization (組織の作成)**」をクリックします。

    > **注**: これで Azure DevOps 組織と同一の SonarCloud 用の組織が作成されました。

    > **注**: 次に、新しく作成された組織内で、Azure DevOps プロジェクト「**SonarExamples**」と同一の SonarCloud プロジェクトを作成します。 

1.  「**All set! 設定完了! 組織を使用できます**」ページで「**新しいプロジェクトの分析**」をクリックします。

1.  「**Analyze projects - Select repositories (プロジェクトの分析 - リポジトリの選択)**」ページの Azure DevOps プロジェクトの一覧で、「**SonarExamples / SonarExamples**」エントリの横にあるチェックボックスを選択し、「**Set up (セットアップ)**」をクリックします。

1.  「**プロジェクトの分析**」ページで「**With Azure DevOps Pipeline**」タイルをクリックします。 

1.  「**Analyze with Azure Pipelines**」ページの「**Install our extension**」セクションで、「**Continue**」をクリックします。

    > **注**: すでにインストールしている場合は拡張機能の作成をスキップできます。 

1.  「**Configure Azure Pipeline**」セクションで「**.NET**」をクリックします。「**分析構成の準備**]、「**コード分析の実行**]、「**クオリティ ゲート結果の発行**」で必要な一連の手順が表示されます。 

    > **注**: 各目標を達成する手順の一覧をレビューします。これは、後続のタスクで実装します。

1.  SonarCloud サービス エンドポイントのセットアップに必要な **Token** の値と、「**Project Key**」および「**Project Name**」の値を記録しておきます。

### 演習 1: SonarCloud と統合する Azure DevOps パイプラインを設定する

この演習では、SonarCloud と統合する Azure DevOps パイプラインを設定します。

> **注**: SonarCloud と統合して **SonarExamples** コードを分析する新しいビルド パイプラインを設定します。 

#### タスク 1: プロジェクト ビルド パイプラインの作成を開始する

このタスクでは、プロジェクト用のビルド パイプラインの作成を開始します。

1.  Azure DevOps ポータルで 「**SonarExamples**」ペインが表示されている Web ブラウザー ウィンドウに切り替え、Azure DevOps ポータルの一番左にある垂直メニュー バーで 「**パイプライン**」をクリックし、「**Create Pipeline**」をクリックします。

1.  「**コードはどこにありますか?**」ペインで利用可能なオプションをレビューします。

    > **注**: この場合、2 つの選択肢があります。パイプラインは、**YAML エディター**または**クラシック エディター**を使用して構成できます。クラシック エディターを使う場合は、上の手順で SonarCloud Extension の一部としてインストールされていた事前定義テンプレートを利用できます。YAML エディターを使う場合は、別個に提供される YAML ファイルを使用する必要があります。この 2 つのオプションについてそれぞれ順番に説明します。 

    > **注**: クラシック エディターを使用する場合は次のタスクをスキップしてください。

#### タスク 2: YAML エディターを使用してパイプラインを作成する

このタスクでは、YAML エディターを使用してパイプラインを作成します。

> **注**: YAML パイプラインの構成を進める前に、まず、SonarCloud のサービス接続を作成します。

1.  別のブラウザー タブを開き、Azure DevOps ポータルで **SonarExamples** のホーム ページに移動します。

1.  Azure DevOps ポータルで「**SonarExamples**」ペインが表示されている Web ブラウザー ウィンドウの左下コーナーで「**Project Settings**」をクリックします。

1.  「**Pipelines**」セクションで「**Service connections**」をクリックしてから「**Create service connection**」をクリックします。

1.  「**New service connection**」ペインで「**SonarCloud**」オプションを選択してから「**Next**」をクリックします。

1.  「**New SonarCloud service connection**」ペインの「**SonarCloud Token**」テキストボックスに、前のタスクで記録しておいたトークンの値を貼り付け、**Verify** をクリックします。

1.  「**サービス接続名**」テキストボックスに「**SC**」と入力し、「**Verify and Save**」をクリックします。 

1.  パイプラインに戻って、**Create Pipeline** をクリックします。

1.  「**コードはどこにありますか?**」ペインで「**Azure Repos Git**」をクリックします。

6.  「**リポジトリの選択**」ペインで「**SonarExamples**」をクリックします。 

7.  「**パイプラインの構成**」ペインで「**.NET Desktop**」YAML テンプレートをクリックします。

    > **注**: テンプレート YAML ファイルが開いている状態で YAML エディターが自動的に表示されます。適正に構成するには、以下のファイルに一致するように調整 (または置換) が必要になります:

    ```yaml
    trigger:
    - master

    pool:
      vmImage: 'windows-latest'

    variables:
      buildConfiguration: 'Release'
      buildPlatform: 'any cpu'

    steps:
    - task: NuGetToolInstaller@0
      displayName: 'Use NuGet 4.4.1'
      inputs:
        versionSpec: 4.4.1

    - task: NuGetCommand@2
      displayName: 'NuGet restore'
      inputs:
        restoreSolution: 'SomeConsoleApplication.sln'

    - task: SonarCloudPrepare@1
      displayName: 'Prepare analysis configuration'
      inputs:
        SonarCloud: 'SC'
        organization: 'myorga'
        scannerMode: 'MSBuild'
        projectKey: 'dotnet-framework-on-azdo'
        projectName: 'Sample .NET Framework project with Azure DevOps'


    - task: VSBuild@1
      displayName: 'Build solution **\*.sln'
      inputs:
        solution: 'SomeConsoleApplication.sln'
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: VSTest@2
      displayName: 'VsTest - testAssemblies'
      inputs:
        testAssemblyVer2: |
         **\$(BuildConfiguration)\*Test*.dll
         !**\obj\**
        codeCoverageEnabled: true
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: SonarCloudAnalyze@1
      displayName: 'Run SonarCloud analysis'

    - task: SonarCloudPublish@1
      displayName: 'Publish results on build summary'
    ```

    > **注**: **Net-desktop-sonarcloud.yml** ファイルを [SonarSource GitHub リポジトリ](https://github.com/SonarSource/sonar-scanner-vsts/blob/master/yaml-pipeline-templates/net-desktop-sonarcloud.yml)からダウンロードできます。

    > **注**: YAML パイプラインは、このタスクの残りの手順に従って変更する必要があります。 

1.  **NuggetCommand@2** タスクで、`restoreSolution: 'SomeConsoleApplication.sln'` を `restoreSolution: '**\SomeConsoleApplication.sln'` に置き換え、このソリューションがリポジトリのルートにないことを考慮に入れます。

1.  「**VSBuild@1**」タスクで、`solution: 'SomeConsoleApplication.sln'` を `solution: '**\SomeConsoleApplication.sln'` に置き換え、このソリューションがリポジトリのルートにないことを考慮に入れます。

1.  「**SonarCloudPrepare@1**」タスクで `organization: 'myorga'` エントリの `myorga` プレースホルダーの値を SonarCloud 組織の名前に置き換えます。規定では、DevOps 組織の名前と同じです。

1.  「**SonarCloudPrepare@1**」タスクで、`projectKey: 'dotnet-framework-on-azdo'` エントリの `dotnet-framework-on-azdo` プレースホルダーの値を SonarCloud プロジェクト キーに置き換えます。

1.  「**SonarCloudPrepare@1**」タスクで、`projectName: 'Sample .NET Framework project with Azure DevOps'` エントリの `Sample .NET Framework project with Azure DevOps` プレースホルダーの値を SonarCloud プロジェクトの名前 (`SonarExamples`) に置き換えます。

1.  「**パイプライン YAML のレビュー**」ペインで「**保存して実行**」をクリックし、「**保存して実行**」ペインで「**保存して実行**」をクリックします。

1. 「プロジェクト設定」>「概要」に移動し、可視性フィールドで「**プライベート**」に変更します。
    > **注**: この最後のステップは、2021 年 2 月に公共プロジェクトに変更が発生したために必要です。パイプラインへのアクセスを要求する必要があります: https://devblogs.microsoft.com/devops/change-in-azure-pipelines-grant-for-public-projects/

1. パイプラインに移動し、**Sonarexample** パイプラインをクリックして、最新の実行を開きます。

1. **his pipeline needs permission to access a resource before this run can continue** が表示されていたら、**View** をクリックして **Permit** をクリックします。

    > **注**: 前のタスクで YAML エディターを使用した場合は、次のタスクをスキップしてください。

#### タスク 3: クラシック エディターを使用してパイプラインを作成する

このタスクでは、クラシック エディターを使用してパイプラインを作成します。

1.  「**コードはどこにありますか?**」ペインで「**Use the classic editor**」をクリックします。

1.  「**ソースの選択**」ペインで、「**Azure Repos Git**」オプションが選択されており、「**SonarExamples**」エントリが「**リポジトリ**」ドロップダウン リストに表示されており、「**master**」ブランチが表示されていることを確認してから、「**続行**」をクリックします。

    > **注**: 先ほどインストールされた SonarCloud 拡張機能には、Maven、Gradle、.NET Core、.NET Desktop アプリケーション向けの SonarCloud 有効カスタム ビルド テンプレートが含まれています。これらのテンプレートは標準的な Azure DevOps テンプレートに基づいていますが、さらに分析固有のタスクと事前に構成済みの設定が含まれています。

1.  「**テンプレートの選択**」ペインで「**Others**」セクションまでスクロールダウンし、「**.NET Desktop with SonarCloud**」テンプレート エントリをクリックしてから「**Appply**」をクリックします。

    > **注**: テンプレートには必要なタスクすべてと、必要な設定がほとんど含まれています。残りを提供する必要があります。

1.  ビルド パイプライン定義の「**タスク**」タブで、「**Pipeline**」エントリが選択されていることを確認します。右側の「**エージェント プール**」ドロップダウン リストで「**Azure Pipelines**」エントリを選択し、「**Agent Specification**」ドロップダウン リストで「**vs2017-win2016**」エントリを選択します。

1.  パイプライン タスクのリストで、「**Prepare analysis SonarCloud**」タスクを選択し、**SonarCloud Service Endpoint** で「**+ New**」をクリックします。

1.  「**新しいサービス接続**」ペインで「**SonarCloud トークン**」テキストボックスに、このラボで以前に記録しておいたトークンの値を貼り付けます。「**Verify**」をクリックしてこれを検証します。「**サービス接続名**」テキストボックスに「**SC**」と入力し、「**確認して保存**」をクリックします。 

1.  「**分析構成の準備**」ペインに戻り、「**Organization**」ドロップダウン リストで SonarCloud 組織の名前を選択します。 

1.  「**分析構成の準備**」ペインで「**Project Key**」テキストボックスに、このラボで記録しておいたプロジェクト キーを入力します。

1.  「**分析構成の準備**」ペインで「**Project Name**」テキストボックスに、このラボで記録しておいたプロジェクト名 (`SonarExamples`) を入力します。

1.  オプションで、「クオリティ ゲート結果の発行」を無効にすることもできます。パイプライン タスクのリストで「**Publish Quality Gate Result**」タスクを選択し、「**Control Option**」セクションを拡張して「**Enabled**」チェックボックスをクリアしてください。 

    > **注**: このタスクは、配置前ゲートとリリース パイプラインを使用するのでない限り、必須ではありません。

    > **注**: この手順を有効にすると、分析結果の概要が「**ビルドのSummary**」ページの「**Extensions**」タブに表示されます。ただし、これを行うと、SonarCloud で処理を終了するまでビルドの完了が遅れます。

1.  ビルド パイプライン エディター ペインで「**Save and Queue**」をクリックします。ドロップダウン メニューで「**Save and Queue**」をクリックし、「**パイプラインの実行**」ペインで「**Save and run**」をクリックし、ビルドが完了するまで待ちます。

#### タスク 4: パイプラインの結果を確認する

1.  ビルド実行ペイン (以前に作成された YAML または Classic のいずれか) で、「**Summary**」タブの内容をレビューしてから「**Extensions**」タブをクリックします。

    > **注**: 「**Publish Quality Gate Result**」タスクを有効なままにしておいた場合は、「**Extensions**」タブに SonarCloud 分析レポートの概要が含まれています。

1.  「**Extensions**」タブで「**Detailed SonarCloud report**」をクリックします。新しいブラウザー タブが自動的に開き、SonarCloud プロジェクト ページのレポートが表示されます。

    > **注**: SonarCloud プロジェクトを参照することもできます。 

1.  レポートにクオリティ ゲート結果が含まれていないことを確認し、その理由を確認します。

    > **注**: クオリティ ゲート結果を表示できるようにするには、最初のレポートの実行後、「**Set New Code Definition**」をクリックして設定する必要があります。これにより、その後のパイプライン実行にはクオリティ ゲート結果が含まれるようになります。

1.  SonarCloud プロジェクトの「**Overview**」タブで「**Set New Code Definition**」をクリックします。 

1.  SonarCloud プロジェクトの「**管理**」タブで「**Previous Version**」をクリックします。

1.  最も直近のビルド実行とともに Azure DevOps ポータルの「**SonarExamples**」プロジェクトが表示されている Web ブラウザー ウィンドウに切り替え、「**Run New**」をクリックし、「**パイプラインの実行**」ペインで「**Run**」をクリックします。完了するまで待ちます。

1.  ビルド実行ペインで、「**Summary**」タブの内容をレビューしてから「**Extensions**」タブのヘッダーをクリックします。

1.  「**Detailed SonarCloud report**」をクリックします。新しいブラウザー タブが自動的に開き、SonarCloud プロジェクト ページのレポートが表示されます。

1.  レポートにクオリティ ゲート結果が含まれていることを確認します。

    > **注**: これで SonarCloud で新しい組織が作成され、Azure DevOps ビルドを構成して、分析を行い、ビルドの結果を SonarCloud にプッシュできるようになりました。

### 演習 2: SonarCloud レポートの分析

この演習では、SonarCloud レポートを分析します。

#### タスク 1: このタスクでは、SonarCloud レポートを分析します。

1.  SonarCloud プロジェクトの「**Main Branch**」タブを開き、中央で **Overall Code** タブをクリックします。

    > **注**: このページには、**コードの臭い**や**カバレッジ**、**重複**、**サイズ** (コードのライン) といった他のメトリクスも含まれています。以下の表はそれぞれの用語について簡単に説明したものです。

    | 用語 | 説明 |
    | --- | --- |
    | **Bugs - バグ** | コードのエラーを示す問題。バグでまだ不具合が生じていない場合は、おそらく最悪のタイミングで発生するでしょう。これは修正しておく必要があります。 |
    | **Vulnerabilities - 脆弱性** | 攻撃者に潜在的なバックドアを提供するセキュリティ関連の問題 |
    | **Code Smells - コードの臭い** | コードのメンテナンス性関連の問題。そのままにしておくと、メンテナンス担当者が変更を加える際、通常よりも作業が困難になります。最悪の場合はコードの状態が混乱し、変更を加えるとさらにエラーが発生する可能性があります |
    | **Coverage** | 単体テストなどのテストで検証中のコードの割合を示します。バグから効果的に防御するには、このようなテストをコードの大部分で行うかカバーする必要があります |
    | **Duplications** | 重複修飾は、ソース コードのどの部分が重複しているのかを示します |

    > **注**: バグ カウントの隣に表示されている文字は**信頼性の評価**を示します。特に「**C**」という文字は、コードに少なくとも 1 つの重大なバグがあることを示唆します。信頼性評価の詳細については、[SonarQube ドキュメント](https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability)を参照してください。[ルールの種類](https://docs.sonarqube.org/latest/user-guide/rules/)に関する詳細も記載されています。[重要度](https://docs.sonarqube.org/latest/user-guide/issues/)を参照してください。

1.  **バグ**の数を示す数字をクリックします。「**Issues**」タブの内容が自動的に表示されます。 

1.  表示されているバグをクリックすると該当するコードが表示されます。 

    > **注**: **Program.cs** ファイルのライン番号 9 でエラーの詳細を確認します。「**常に 'true' にならないようこの条件を変更してください。後続のコードには実行されないものもあります**」という推奨も含まれます。

1.  コードとライン番号の間の垂直線上にマウスのポインターをホバーすると、コードのカバレッジのギャップを特定できます。（**Not covered by tests** と表示されます）

    > **注**: サンプル プロジェクトは非常に小規模なので履歴データは含まれていません。ただし、[SonarCloud のパブリック プロジェクト](https://sonarcloud.io/explore/projects)は多数あり、より興味深く現実的な結果を閲覧できます。

### 演習 3: Azure DevOps プル要求を SonarCloud と統合する

この演習では、Azure DevOps と SonarCloud 間のプル要求の統合を設定します。

> **注**: Azure DevOps プル要求に含まれているコードの分析を行うよう SonarCloud 分析を構成するには、以下のタスクを実行する必要があります:

- Azure DevOps 個人用アクセス トークンを SonarCloud プロジェクトに追加します (これによりプル要求へのアクセスが承認されます)。
- プル要求でトリガーされるビルドを管理する Azure DevOps ブランチ ポリシーを構成します。

#### タスク 1: SonarCloud とのプル要求統合向けに Azure DevOps 個人用アクセス トークンを作成する

このタスクでは、Azure DevOps プル要求を SonarCloud プロジェクトと統合するための個人用アクセス トークン要件をレビューします。

1.  Azure DevOps ポータルで **SonarExamples** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。

1.  このラボで前述されていた手順を繰り返し、**コード**のスコープと **SonarExamples** プロジェクトの**読み取りと書き込み**許可がある個人用アクセス トークンを生成します。 

    > **注**: このラボで以前に生成した個人用アクセス トークンを再使用することもできます。

    > **注**: プル要求への SonarCloud コメントは、個人用アクセス トークンを作成したユーザーのセキュリティを考慮して追加されます。この目的で別個の「ボット」Azure DevOps ユーザーを作成し、SonarCloud からのコメントを明確に識別できるようお勧めします。

#### タスク 2: SonarCloud でプル要求統合を構成する

このタスクでは、Azure DevOps 個人用アクセス トークンを SonarCloud プロジェクトに割り当てて、SonarCloud でプル要求統合を構成します。

1.  SonarCloud ポータルで **SonarExamples** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。 

1.  プロジェクトのダッシュボード ページで「**Administration**」タブのヘッダーをクリックし、ドロップダウン メニューで「**General Settings**」をクリックします。

1.  「**General Settings**」ページで「**Pull Requests**」タブをクリックします。

1.  「**Pull Requests**」設定の「**General**」セクションで「**Providor**」ドロップダウン リストから「**Azure DevOps Services**」を選択して「**Save**」をクリックします。

1.  「**Pull Requests**」設定の「**Integration with Azure DevOps Services**」セクションで、「**個人用アクセス トークン**」テキストボックスに貼り付けて、「**保存**」をクリックします。

#### タスク 3: SonarCloud との統合向けのブランチ ポリシーを構成する

このタスクでは、SonarCloudとの統合向けに Azure DevOps ブランチ ポリシーを構成します。

1.  Azure DevOps ポータルで **SonarExamples** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。

1.  Azure DevOps ポータルの一番左にある垂直メニュー バーで「**リポジトリ**」をクリックし、「**リポジトリ**」セクションで「**Branches**」をクリックします。 

1.  「**Branches**」ペインのブランチ一覧で、マウスのポインターを**master** ブランチ エントリの右端に配置すると、垂直の省略記号が現れるので、これをクリックし、ポップアップ メニューで「**Branch Policies**」をクリックします。

1.  「**master**」ペインの「**Build Validation**」セクションの右側で「**+**」をクリックします。

1.  「**Add build policy**」ペインの「**Build pipeline**」ドロップダウン リストで、このタスクで先ほど作成したパイプラインを選択します。

1.  一番下の「**Display name**」テキストボックスに「**SonarCloud 分析**」と入力し、「**Save**」をクリックします。

    > **注**: これで Azure DevOps は**master** ブランチを対象にしたプル要求が作成されると、 SonarCloud 分析をトリガーするよう構成されました。

#### タスク 4: プル要求の統合を検証する

このタスクでは、プル要求を作成し、その結果をレビューすることにより、Azure DevOps と SonarCloud 間のプル要求統合を検証します。

> **注**: リポジトリのファイルに変更を加え、SonarCloud 分析をトリガーする要求を作成します。

1.  Azure DevOps ポータルの左側にある垂直メニュー バーで「**リポジトリ**」をクリックします。「**ファイル**」ペインが表示されます。 

1.  中央のペインのフォルダー階層で **sonarqube-scanner-msbuild\\CSharpProject\\SomeConsoleApplication\\Program.cs** フォルダーの **Program.cs** ファイルに移動し、「**編集**」をクリックします。
1.  「**Program.cs**」ペインで、`public static bool AlwaysReturnsTrue()` ラインのすぐ上にあるコードに以下の空のメソッドを追加します 

    ```csharp
    public void Unused(){

    }
    ```

1.  「**Program.cs**」ペインで、「**Commit**」をクリックします。

1.  「**コミット**」ペインで「**ブランチ名**」テキストボックスに「**branch1**」と入力します。「**Create a pull request**」チェックボックスを選択し、「**コミット**」をクリックします。

1.   **New pull request** が自動的に表示されたら、**Create** をクリックします。

1.  「**Updated Program.cs**」ペイン「**Overview**」タブで、ビルド プロセスの完了まで進捗状況を監視します。 

1.  「**Updated Program.cs**」ペインの「**Overview**」タブで、必要なチェックを行う一方、オプションのチェックが失敗したことを確認し、「**View 3 checks**」リンクをクリックします。

1.  「**チェック**」ペインで SonarCloud チェックの結果をレビューし、ペインを閉じます。

    > **注**: 結果は、分析ビルドが完了したことを示していますが、PR の新しいコードはコード品質チェックに失敗しています。発見された新しい問題について、コメントが PR に投稿されています。

1.  「**Updated Program.cs**」ペインの「**Overview**」タブに戻り、コメントが含まれているセクションまでスクロールダウンし、新しく追加されたクラスに関する SonarCloud からのコメントをレビューします。 

    > **注**: 報告された問題には、プル要求に対応するコードの変更のみが含まれています。**Program.cs** と他のファイルの既存の問題は無視されます。

#### タスク 4: コード品質チェックの失敗に対応するプル要求をブロックする

このタスクでは、コード品質チェックの失敗に対応するプル要求のブロックを構成します。 

> **注**: この時点では、コード品質チェックが失敗しても、まだプル要求を完了して、該当する変更をコミットできます。適切なコード品質チェックに合格しなければ、Azure DevOps 構成を変更してコミットをブロックします。

1.  Azure DevOps ポータルの「**Updated Programs.cs**」ペインの左下コーナーで「**Project Settings**」をクリックします。

1.  「**プロジェクトの設定**」垂直メニューの「**リポジトリ**」セクションで「**リポジトリ**」をクリックします。

1.  「**すべてのリポジトリ**」ペインで「**SonarExamples**」をクリックします。

1.  「**SonarExamples**」ペインで「**Policies**」タブクリックします。

1.  「**Policies**」一覧でブランチのリストまでスクロールダウンし、**master** ブランチを示すエントリをクリックします。

1.  「**master**」ペインで「**Status Checks**」セクションまでスクロールダウンし、「**+**」をクリックします。

1.  「**Add status policy**」ペインの「**Status to check**」ドロップダウン リストで「**SonarCloud / quality gate**」エントリを選択し、「**Policy requirement**」オプションが「**Required**」に設定されていることを確認して「**Save**」をクリックします。

    > **注**: この時点でユーザーは、コード品質チェックに成功するまでプル要求をマージできなくなります。また、SonarCloud で識別された問題はすべて修正済みか、該当する SonarCloud プロジェクトで**確認済み**または**解決済み**としてマークされていなくてはなりません。

## レビュー

このラボでは、Azure DevOps Services を SonarCloud に統合する方法を学習しました。
