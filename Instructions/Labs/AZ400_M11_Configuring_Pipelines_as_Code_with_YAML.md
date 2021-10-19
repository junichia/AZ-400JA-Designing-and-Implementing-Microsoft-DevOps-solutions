---
lab:
    title: 'ラボ 11a: YAML を使用したコードとしてのパイプラインの構築'
    module: 'モジュール 11: Azure Pipelines を使用した継続的デプロイの実装'
---

# ラボ 11a: YAML を使用したコードとしてのパイプラインの構築
# 受講生用ラボ マニュアル

## ラボの概要

多くのチームは YAML を使用してビルドを定義し、パイプラインをリリースすることを好みます。これにより、ビジュアル デザイナーを使用した場合と同じパイプラインの機能にアクセスできます。ただし、マークアップ ファイルは他のソース ファイルと同様に管理できます。YAML ビルドの定義は、該当するファイルをリポジトリのルートに加えるだけでプロジェクトに追加できます。Azure DevOps は、人気の高いプロジェクトの種類の既定テンプレートのほか、YAML デザイナーを提供して、ビルドおよびリリース タスクの定義プロセスを簡素化します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

## ラボの所要時間

-   推定時間: **60 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成され、Azure Web アプリと Azure SQL データベースが含まれます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited-YAML** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: このサイトの詳細については、「[Azure DevOps Services Demo Generator とは](https://docs.microsoft.com/ja-jp/azure/devops/demo-gen)」を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**Configuring Pipelines as Code with YAML**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで 「**General**」 をクリックし、「**PartsUnlimited-YAML**」 テンプレートを選択して 「**テンプレートの選択**」 をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Azure リソースを作成する

このタスクでは、Azure portal を使用して Azure Web アプリを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure ADテナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。

1.  Azure portal で、ページの左上にある横線 3 本のアイコンをクリックし、「**+ リソースの作成**」をクリックします。

.  「**Web アプリ**」をクリックします。

1.  「**Web App + Database**」ブレードで以下の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m11l01-RG**」の名前 |
    | 名前 | 有効な、グローバルに一意の DNS ホスト名 |
    | ランタイムスタック | ASP.NET V4.8 |
    | オペレーティングシステム | Windows |
    | 地域 | EastUS などのリージョン |
    
1. App Service プランで **新規作成** をクリックします。

1. 新しい App Service プラン名として **az400l11plan** を入力します。

1. **サイズを変更します** をクリックします。

1. **開発/テスト** をクリックし、**D1** を選択したら、**適用**をクリックします。

1. **確認と作成** をクリックします。

1. **作成** をクリックし、リソースが作成されるまで待ちます。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。 

### 演習 1: Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### タスク 1: 既存のパイプラインを削除する

このタスクでは、既存のパイプラインを削除します。

1.  ラボのコンピューターで、Azure DevOps ポータルに「**Configuring Pipelines as Code with YAML**」プロジェクトが表示されているブラウザー ウィンドウに切り替え、垂直のナビゲーション ペインで「**パイプライン**」を選択します。

    > **注**: YAML パイプラインを構成する前に、既存のビルド パイプラインを無効にします。

1.  「**パイプライン**」ペインで「**PartsUnlimited**」エントリを選択します。 

1.  「**PartsUnlimited**」ブレードの右上コーナーで、垂直の省略記号をクリックし、ドロップダウン メニューで 「**削除**」を選択します。

1.  「**PartsUnlimited**」と書き込んで 「**削除**」 をクリックします。

1.  垂直のナビゲーション ペインで **「リポジトリ」 > 「ファイル」** を選択します。**マスター** ブランチ (「**ファイル**」 ウィンドウの最上部にあるドロップダウン) の **azure-pipelines.yml** ファイルで、垂直の省略記号をクリックします。ドロップダウン メニューで 「**削除**」 を選択します。「**コミット**」 をクリックして、マスター ブランチで変更をコミットします (既定のオプションのままにします)。

#### タスク 2: YAML ビルド定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1.  「**パイプライン**」に戻ります。 

1.  「**最初のパイプラインの作成**」ウィンドウで、「**パイプラインの作成**」をクリックします。 

    > **注**: ウィザードを使い、プロジェクトに基づいて YAML の定義を自動的に作成します。

1.  「**コードはどこにありますか?**」ペインで「**Azure Repos Git (YAML)**」オプションをクリックします。

1.  「**リポジトリの選択**」ペインで「**PartsUnlimited**」をクリックします。

1.  「**パイプラインの構成**」ペインで 「**ASP.NET**」をクリックし、このテンプレートをパイプラインの起点として利用します。これにより、「**パイプライン YAML のレビュー**」ペインが開きます。

    > **注**: パイプラインの定義は、リポジトリのルートで「**azure-pipelines.yml**」という名前のファイルとして保存されます。このファイルには、典型的な ASP.NET ソリューションを構築してテストするために必要なステップが含まれます。また、必要に応じてビルドをカスタマイズすることもできます。このシナリオでは、**プール**を更新し、Visual Studio 2017 を実行している VM を使用できるようにします。

1.  `trigger` が **master** であることを確認します。

    > **注**: リポジトリに**master**または**main** ブランチがあるかリポジトリでレビューします。組織は新しいリポジトリの既定のブランチ名を選択できます。[既定のブランチを変更](https://docs.microsoft.com/ja-jp/azure/devops/repos/git/change-default-branch?view=azure-devops#choosing-a-name)します。 

1.  「**パイプライン YAML のレビュー**」ペインの **10** 行目で、`vmImage: 'windows-latest'` を `vmImage: 'vs2017-win2016'` に置き換えます。

1.  「**パイプライン YAML のレビュー**」ペインで「**Save and run**」をクリックします。

1.  「**Save and run**」ペインで既定の設定を承諾し、「**Save and run**」をクリックします。

1.  パイプライン実行ペインの「**Jobs**」セクションで「**Job**」をクリックし、進捗状況を監視して、完了したことを確認します。 

    > **注**: 警告やエラーなど、YAML ファイルの各タスクをレビューできます。

1.  パイプライン実行ペインに戻り、「**Summary**」タブから「**Tests**」タブに切り替えてテスト統計情報をレビューします。

#### タスク 3: YAML 定義に継続的デリバリーを追加する

このタスクでは、以前のタスクで作成したパイプラインの YAML ベースの定義に継続的デリバリーを追加します。

> **注**: ビルドとテストのプロセスに成功すると、YAML 定義にデリバリーを追加できます。 

1.  パイプライン実行ペインで、右上コーナーにある省略記号をクリックし、ドロップダウン メニューで「**Edit pipeline**」をクリックします。

1.  **azure-pipelines.yaml** ファイルのコンテンツが表示されているペインで `trigger` セクションの後の **8** 行目に以下のコンテンツを追加し、YAML パイプラインの 「**ビルド**」ステージを定義します。 

    > **注**: パイプラインの進捗状況をよりよく整理して追跡するために必要なステージを定義できます。

    ```yaml
    stages:
    - stage: Build
      jobs:
      - job: Build
    ```

1.  YAML ファイルの残りのコンテンツを選択し（- job: Build より下の行の全て）   、**Tab** キーを 3 回押してスペース 6 つ分をインデントします (```job: Build``` と同じインデントに置き換えます)。 

    > **注**: これにより、`pool` セクションで始まるものはすべて `job: Build` の一部になります。 

1.  ファイルの最下部で、以下の項性を追加して 2 番目のステージを定義します。

    ```yaml
    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
          vmImage: 'vs2017-win2016'
        steps:
    ```

1.  YAML 定義の最後で新しいライン上にカーソルを配置します。 

    > **注**: これは、新しいタスクが追加される場所になります。

1.  コード ペインの右側のタスク一覧で、「**Azure App Service Deploy**」タスクを検索して選択します。

1.  「**Azure App Service のデプロイ**」ペインで以下の設定を指定し、「**Add**」をクリックします。

    - 「**Azure サブスクリプション**」ドロップダウン リストで、ラボで以前に Azure リソースをデプロイした Azure サブスクリプションを選択し、「**Authorize**」をクリックします。指示されたら、Azure リソース デプロイの際に使用したものと同じユーザー アカウントを使用して認証します。
    - 「**App Service の名前**」ドロップダウン リストで、ラボで以前にデプロイした Web アプリの名前を選択します。 
    - 「**Package or folder**」テキストボックスで `$(System.ArtifactsDirectory)/drop/*.zip` と入力します。 

    > **注**: これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

1.  エディターで追加されたタスクを選択したまま   の状態にして、**Tab** キーを 3 回押し、スペース 6 つ分をインデントします。これにより、**steps** の子タスクとしてリストされます。

    > **注**: 一番下の **packageForLinux** パラメーターは、このラボの状況では不適切に見えますが、Windows または Linux の両方で有効なパラメタです。 

    > **注**: 既定により、2 つのステージは独立して実行されます。このため、最初のステージのビルド出力は、さらに変更を加えなければ 2 番目のステージで利用できない可能性があります。このような変更を実装するには、ひとつのタスクを使用してビルド出力をビルド ステージの最後に公開し、別のタスクを使ってデプロイ ステージの最初にこれをダウンロードします。 

1.  Build ステージの最後に空白のライン上にカーソルを配置して、もうひとつタスクを追加します。(右下の `task: VSTest@2` )

1.  「**Tasks**」ペインで「**Publish build artifacts**」タスクを検索して選択します。 

1.  「**Publish build artifacts**」ペインで既定の設定を承諾し、「**追加**」をクリックします。 

    > **注**: これにより、「**drop**」というエイリアスでダウンロードできる場所にビルド成果物が公開されます。

1.  追加されたタスクをエディターで選択したままの状態にして、**Tab** キーを 3 回押し、スペース 6 つ分をインデントします (またはタスクが 1 つ上にインデントされるまで **Tab** を押します)。

    > **注**: また、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  **Deploy** ステージの **steps** ノードの下に１行追加し、最初のラインにカーソルを配置します。

1.  「**Tasks**」ペインで「**Download build artifacts**」タスクを検索して選択します。

1.  **「追加」** をクリックします。

1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 3 回押し、スペース 6 つ分をインデントします。 

    > **注**: ここでも、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  Download タスクにプロパティを追加し、`drop` の `artifactName` を指定します (必ずスペースを一致させてください):

    ```
    ArtifactName: 'drop'
    ```

1.  「**Save**」をクリックし、「**Save**」ペインで再び「**Save**」をクリックして変更を直接、マスター ブランチにコミットします。

    > **注**: これにより、新しいビルドが自動的にトリガーされます。

1.  パイプラインはこの例のようになります (**自分のサブスクリプションと前回のタスクの webapp を参照します**):

    ```
    trigger:
    - master

    stages:
    - stage: Build
      jobs:
      - job: Build
        pool:
            vmImage: 'vs2017-win2016'

        variables:
            solution: '**/*.sln'
            buildPlatform: 'Any CPU'
            buildConfiguration: 'Release'

        steps:
        - task: NuGetToolInstaller@1

        - task: NuGetCommand@2
          inputs:
            restoreSolution: '$(solution)'

        - task: VSBuild@1
          inputs:
            solution: '$(solution)'
            msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.artifactStagingDirectory)"'
            platform: '$(buildPlatform)'
            configuration: '$(buildConfiguration)'

        - task: VSTest@2
          inputs:
            platform: '$(buildPlatform)'
            configuration: '$(buildConfiguration)'

        - task: PublishBuildArtifacts@1
          inputs:
            PathtoPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: 'drop'
            publishLocation: 'Container'

    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
            vmImage: 'vs2017-win2016'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            downloadPath: '$(System.ArtifactsDirectory)'
            artifactName: 'drop'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'YOUR-AZURE-SUBSCRIPTION'
            appType: 'webApp'
            WebAppName: 'YOUR-WEBAPP-NAME'
            packageForLinux: '$(System.ArtifactsDirectory)/drop/*.zip'
    ```

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、「**パイプライン**」を選択します。

1.  「**パイプライン**」ペインで、新しく構成されたパイプラインを示すエントリをクリックします。

1. 最も直近の実行をクリックします (自動的に開始されています)。

1.  「**Summary**」ペインで、パイプライン実行の進捗状況を監視します。

1.  「**This pipeline needs permission to access a resource before this run can continue to Deploy**」というメッセージが表示されたら、「**View**」をクリックします。「**Waiting for review**」ダイアログ ボックスで「**Permit**」をクリックします。「**アクセス許可?**」ペインで再び「**Permit**」をクリックします。

1.  「**Summary**」ペインの最下部で「**Deploy**」ステージをクリックすると、デプロイの詳細が表示されます。

    > **注**: タスクが完了すると、アプリが Azure Web アプリにデプロイされます。

#### タスク 4: デプロイされたサイトをレビューする

1.  Azure portal で Azure Web アプリの画面に移動します。

1.  Azure Web アプリ ブレードで「**概要**」をクリックします。概要ブレードで「**参照**」をクリックし、新しい Web ブラウザー タブでサイトを開きます。

1.  デプロイされたサイトが新しいブラウザー タブで期待されているように読み込まれていることを確認します。

    >WEBアプリが正常に表示されない可能性がありますが、これは気にしなくて結構です。デプロイが正常に完了していればこの演習は成功です。 

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成しました。
