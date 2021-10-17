---
lab:
    title: 'ラボ 05: エージェント プールの構成とパイプライン スタイルの理解'
    module: 'モジュール 5: Azure Pipelines の構成'
---

# ラボ 05: エージェント プールの構成とパイプライン スタイルの理解
# 受講生用ラボ マニュアル

## ラボの概要

YAML ベースのパイプラインを使用すると、CI/CD をコードとして完全に実装できます。パイプライン定義は、Azure DevOps プロジェクトの一部であるコードと同じリポジトリにあります。YAML ベースのパイプラインは、プルリクエスト、コードレビュー、履歴、分岐、テンプレートなど、従来のパイプラインの一部である幅広い機能をサポートします。 

パイプライン スタイルの選択に関係なく、Azure Pipelines を使用してコードをビルドしたりソリューションをデプロイしたりするには、エージェントが必要です。エージェントは、一度に 1 つのジョブを実行するコンピューティング リソースをホストします。ジョブは、エージェントのホスト マシンで直接実行することも、コンテナーで実行することもできます。管理されている Microsoft がホストするエージェントを使用してジョブを実行するか、自分で設定および管理するセルフホステッド エージェントを実装するかを選択できます。 

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換し、最初に Microsoft がホストするエージェントを使用して実行し、次にセルフホステッド エージェントを使用して同等のタスクを実行するプロセスを順を追って説明します。

## 目標

このラボを完了すると、次のことができるようになります。

- YAML ベースのパイプラインを実装する
- セルフホステッド エージェントを実装する

## ラボの所要時間

-   推定時間: **45 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### インストールされているアプリケーションを確認します

Windows 10 デスクトップでタスク バーを探します。タスク バーには、この課題で使用するアプリケーションのアイコンが含まれています:
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/)。このラボでは前提条件の一部としてインストールされます。

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### 演習 0ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。

1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。

1.  「**新しいプロジェクトの作成**」 ページの 「**新しいプロジェクト名**」 テキスト ボックスに「**Configuring Agent Pools and Understanding Pipeline Styles**」と入力し、「**組織の選択**」 ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」 をクリックします。

1.  「**テンプレートの選択**」 ページで、**PartsUnlimited** テンプレートをクリックし、「**テンプレートの選択**」 をクリックします。

1.  「**プロジェクトの作成**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### 演習 1: YAML ベースの Azure DevOps パイプラインを作成する

この演習では、従来の Azure DevOps パイプラインを YAML ベースのパイプラインに変換します。 

#### タスク 1: Azure DevOps YAML パイプラインを作成する

このタスクでは、テンプレートベースの Azure DevOps YAML パイプラインを作成します。

1.  **Configuring Agent Pools and Understanding Pipeline Styles**プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ペインで 「**パイプライン**」 をクリックします。 

1.  「**パイプライン**」 ペインの 「**Recent - 最近**」 タブで、右端にある「**New pipeline - 新しいパイプライン**」 をクリックします。

1.  「**Whwew is your code - コードはどこにありますか?**」ペインで「**Azure Repos Git**」をクリックします。 

1.  「**リポジトリの選択**」ペインで「**PartsUnlimited**」をクリックします。

1.  「**パイプラインの構成**」 ペインで、「**Starter pipeline**」 をクリックします。

1.  「**Review your pipeline YAML - パイプライン YAML の確認**」 ペインでサンプル パイプラインを確認し、「**Save and run - 保存して実行**」 ボタンの横にある下向きのキャレット記号をクリックして 「**Save**」 をクリックし、「**Save**」 ペインで 「**Save**」 をクリックします。

    > **注**: これにより、プロジェクト コードをホストするリポジトリのルート ディレクトリに **azure-pipelines.yml** ファイルが作成されます。

    > **注**: サンプル YAML パイプラインのコンテンツをクラシック エディターによって生成されたパイプラインのコードに置き換え、クラシック パイプラインと YAML パイプラインの違いを考慮して変更します。

#### タスク 2: クラシック パイプラインを YAML パイプラインに変換する

このタスクでは、クラシック パイプラインを YAML パイプラインに変換します。

1.  **Configuring Agent Pools and Understanding Pipeline Styles**プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ペインで 「**パイプライン**」 をクリックします。 

1.  「**パイプライン**」 ペインの 「**最近**」 タブで、**PartsUnlimitedE2E** エントリを含むエントリの右端にマウス ポインターを合わせると、「**その他**」 メニューを示す縦の省略記号が表示され、省略記号をクリックして、ドロップダウン メニューで 「**編集**」 をクリックします。これにより、ラボの開始時に生成したプロジェクトの一部であるビルド パイプラインが表示されます。 

1.  **PartsUnlimitedE2E** 編集ペインの 「**タスク**」 タブで、「**Triggers - トリガー**」 をクリックし、**PartsUnlimited** ペインの右側で 「**
Enable continuous integration - 継続的インテグレーションを有効にする**」 チェックボックスをオフにし、「**Save and quiue**」 ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで 「**保存**」 をクリックし、「**ビルド パイプラインの保存**」 で 「**保存**」 をクリックします。

    > **注**: これにより、リポジトリの変更をトリガーにして自動ビルドが実行されるのを防ぐことができます。

1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 

1.  「**パイプライン**」 ペインの 「**最近**」 タブで、**PartsUnlimitedE2E** エントリをクリックします。 

1.  **PartsUnlimitedE2E** ペインの 「**Runs**」 タブの右上隅にある縦の省略記号 (垂直の 3 つの点) をクリックし、ドロップダウン メニューで 「**Export to YAML**」 をクリックします。これにより、**PartsUnlimitedE2E.yml** ファイルがローカルの**ダウンロード** フォルダーに自動的にダウンロードされます。

    > **注**: 「**YAML にエクスポート**」 機能は、Azure DevOp sポータル内のパイプライン エディター ペインから使用できる古い 「**YAML の表示**」 オプションに置き換わるものです。この機能は、YAML 解析ライブラリを含む既存のクラシックおよび YAML パイプライン インフラストラクチャを活用し、2 つの間のより正確な変換を実現します。次のパイプライン コンポーネントをサポートします。

    - 単一および複数のジョブ
    - チェックアウト オプション
    - 実行プランの並列処理
    - タイムアウトと名前のメタデータ
    - 需要
    - スケジュールとその他のトリガー
    - デフォルトとは異なるジョブを含むプールの選択
    - すべてのタスクとすべての入力 (デフォルトの入力の最適化を含む)
    - ジョブとステップの条件
    - タスク グループの展開

    > **注**: 新しい機能でカバーされていないコンポーネントは、変数とタイムゾーン変換のみです。YAML で定義された変数は、Azure DevOps ポータルのユーザー インターフェイスで設定された変数をオーバーライドします。**YAML へのエクスポート**機能がクラシック パイプライン変数の存在を検出した場合、それらは新しく生成された YAML パイプライン定義内のコメントに明示的に含まれます。同様に、YAML の cron スケジュールは UTC で表されるため、従来のスケジュールは組織のタイムゾーンに依存するため、その存在もコメントに含まれます。

    > **注**: この機能の詳細については、「[View YAML」の置き換え](https://devblogs.microsoft.com/devops/replacing-view-yaml/)を参照してください。

1.  ラボ コンピューターで、Visual Studio Code を起動し、それを使用してファイル **PartsUnlimitedE2E.yml** を開きます。ファイルには、次の内容が含まれています。

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  Visual Studio Code ウィンドウのトップレベル メニューで 「**選択**」 をクリックし、ドロップダウン メニューで 「**すべて選択**」 をクリックします。

1.  Visual Studio Code ウィンドウのトップレベルメニューで 「**編集**」 をクリックし、ドロップダウン メニューで 「**コピー**」 をクリックします。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替え、左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 

1.  「**パイプライン**」 ペインの 「**最近**」 タブで 「**PartsUnlimited**」 を選択し、「**PartsUnlimited**」 ペインで 「**Edit**」 を選択します。

1.  **PartsUnlimited** 編集ペインで、パイプラインの既存の YAML コンテンツを選択し、クリップボードのコンテンツに置き換えます。

1.  **PartsUnlimited** 編集ペインの右上隅にある 「**Save**」 をクリックし、「**Save**」 ペインの 「**Save**」 をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。 

1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。

1.  「**パイプライン**」 ペインの 「**最近**」 タブで、「**PartsUnlimited**」 エントリをクリックし、「**PartsUnlimited**」 ペインの 「**Runs**」 タブで最新の実行をクリックし、実行の 「**Summary**」 ペインで一番下までスクロールし、「**ジョブ**」 セクションで 「**Phase 1**」 をクリックし、正常に完了するまでジョブを監視します。 

### 演習 2: Azure DevOps エージェント プールを管理する

この演習では、セルフホスティド Azure DevOps エージェントを実装します。

#### タスク 1: Azure DevOps セルフホスティングエージェントを構成する

このタスクでは、ラボVM を Azure DevOps セルフホスティング エージェントとして構成し、それを使用してビルド パイプラインを実行します。

1.  ラボの仮想マシン (ラボ VM) またはお使いになっているコンピュータ内で、Web ブラウザーを起動し、[Azure DevOps ポータル](https://dev.azure.com)に移動し、Azure DevOps 組織に関連付けられている Microsoft アカウントを使用してサインインします。

1.  Azure DevOps ポータルの Azure DevOps ページの右上コーナーで 「**ユーザー設定**」 アイコン（人型のアイコン）をクリックします。<br><br> ドロップダウン メニューで 「**Personal Access Tokens**」 をクリックし、「**+ New Token**」 をクリックします。

1.  「**Create a new personal access token**」ペインで一番下にある「**Show all scopes**」リンクをクリックし、以下の設定を指定します。「**作成**」をクリックします (他はすべて、既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **Configuring Agent Pools and Understanding Pipeline Styles** |
    | スコープ (カスタム定義) | **Agent Pools**|
    | アクセス許可 | **Read & manage** |

1.  「**Success!**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**: 必ずトークンをコピーしてください。このペインを閉じると、取得できなくなります。 

1.  「**Success！**」ペインで「**Close**」をクリックします。

1. 左上隅にある 「**Azure DevOps**」 記号をクリックしてから、左下隅にある 「**Organization settings**」 ラベルをクリックします。

1.  「**Overview**」 ペインの左側の垂直メニューの 「**Pipelines**」 セクションで、「**Agent pool**」 をクリックします。

1.  「**Agent pools**」 ペインの右上隅にある 「**Add pool**」 をクリックします。 

1.  「**エージェント プールの追加**」 ペインの 「**Pool type**」 ドロップダウン リストで、「**Self-hosted**」 を選択し、「**名前**」 テキスト ボックスに「**az400m05l05a-pool**」と入力して、「**Create**」 をクリックします。

1.  「**Agent pools**」 ペインに戻り、新しく作成された **az400m05l05a-pool** を表すエントリをクリックします。 

1.  **az400m05l05a-pool** ペインの 「**Agents**」 タブで、「**New agent**」 ボタンをクリックします。

1.  「**エージェントの取得**」 ペインで、「**Windows**」 タブと 「**x64**」 タブが選択されていることを確認し、「**Download**」 をクリックしてエージェント バイナリを含む zip アーカイブをダウンロードし、ユーザー プロファイル内のローカルの**ダウンロード** フォルダーにダウンロードします。

    > **注**: この時点で、現在のシステム設定でファイルのダウンロードができないことを示すエラーメッセージが表示された場合は、「インターネット エクスプローラー」 ウィンドウの右上隅にある、ドロップダウン メニューの 「**設定**」 メニュー ヘッダーを示す歯車の記号をクリックします。「**インターネット オプション**」 を選択し、「**インターネット オプション**」 ダイアログ ボックスで 「**詳細設定**」 をクリックし、「**詳細設定**」 タブで 「**リセット**」 をクリックし、「**インターネット エクスプローラー設定のリセット**」 ダイアログ ボックスで 「**リセット**」 をクリックし、「**閉じる**」 をクリックして、ダウンロードを再試行します。 

1.  管理者として Windows PowerShell を起動し、「**管理者: Windows PowerShell**」 コンソールから、次のコマンドを実行して、**C:\\agent** ディレクトリを作成し、ダウンロードしたアーカイブのコンテンツをそこに抽出します： 

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1.  同じ 「**管理者: Windows PowerShell**」 コンソールで、以下を実行してエージェントを構成します。

    ```powershell
    .\config.cmd
    ```

1.  プロンプトが表示されたら、次の設定の値を指定します。

    | 設定 | 値 |
    | ------- | ----- |
    | サーバーの URL を入力する | **https://dev.azure.com/ `<organization_name>`** の形式でのAzure DevOps 組織の URL。ここで、`<organization_name>` は、Azure DevOps 組織の名前を表します |
    | 認証タイプを入力する (PAT の場合は Enter 押します) | **Enter** |
    | パーソナル アクセス トークンを入力する | このタスクの前半で記録したアクセス トークン |
    | エージェント・プールを入力する (デフォルトで Enter を押します) | **az400m05l05a-pool** |
    | エージェント名を入力する | **az400m05-VM0** |
    | 作業フォルダーを入力する (_work の場合は Enter 押します) | **Enter** |
    | 「各ステップのタスクの解凍を実行する」と入力します。(N の場合は Enter を押します) | **Enter** |
    | 「エージェントをサービスとして実行しますか?」入力します。(Y/N) (N の場合は Enter を押します) | **Y** |
    | サービスで使用するユーザー アカウントを入力します (NT AUTHORITY\NETWORK SERVICE で Enter を押します) | **Enter** |
    | whether to prevent service starting immediately after configuration is finished? (Y/N) | **Enter** |
    
    > **注**: セルフホスティド エージェントは、サービスまたは対話型プロセスとして実行できます。エージェントの機能の検証が簡単になるため、対話型モードから始めることをお勧めします。本番環境で使用する場合は、エージェントをサービスとして実行するか、自動ログオンを有効にした対話型プロセスとして実行することを検討する必要があります。どちらも実行状態を維持し、オペレーティング システムが再起動された場合にエージェントが自動的に起動するようにするためです。

    > **注**: エージェントが「**正常に開始されました**」ステータスを報告していることを確認します。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替えて、「**エージェントの取得**」 ペインを閉じます。

1.  **az400m05l05a-pool** ペインの 「**エージェント**」 タブに戻り、新しく構成されたエージェントが 「**オンライン**」 ステータスで一覧表示されていることに注意してください。

1.  Azure DevOps ポータルを表示している Web ブラウザー ウィンドウの左上隅にある、**Azure DevOps** ラベルをクリックします。

1.  プロジェクトのリストを表示しているブラウザー ウィンドウで、**Configuring Agent Pools and Understanding Pipeline Styles**プロジェクトを表すタイルをクリックします。 

1.  「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 

1.  「**パイプライン**」 ペインの 「**Recent**」 タブで 「**PartsUnlimited**」 を選択し、「**PartsUnlimited**」 ペインで 「**Edit**」 を選択します。

1.  **PartsUnlimited** 編集ペインの既存の YAML ベースのパイプラインで、**7** 行目の `vmImage: vs2017-win2016` を置き換えて、ターゲット エージェント プールを指定し、次のコンテンツを指定し、新しく作成されたセルフホスティド エージェント プールを指定します：

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-VM0
    ```
1. `Task: NugetInstaller@0` タスクの上で 「**Settings (グレーで表示されているリンク)**」 をクリックし、**「Advanced」 > 「NuGet バージョン」 > 「4.0.0」** に変更して 「**Add**」 をクリックします。 

1.  **PartsUnlimited** 編集ペインのペインの右上隅にある 「**Save**」 をクリックし、「**Save**」 ペインでもう一度 「**Save**」 をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。 

1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。

1.  「**パイプライン**」 ペインの 「**Recent**」 タブで、「**PartsUnlimited**」 エントリをクリックし、「**PartsUnlimited**」 ペインの 「**Run**」 タブで最新の実行を選択し、実行の 「**概要**」 ペインで一番下までスクロールし、「**ジョブ**」 セクションで 「**Phase 1**」 をクリックし、正常に完了するまでジョブを監視します。 

#### レビュー

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換する方法と、セルフホスティド エージェントを実装して使用する方法を学習しました。
