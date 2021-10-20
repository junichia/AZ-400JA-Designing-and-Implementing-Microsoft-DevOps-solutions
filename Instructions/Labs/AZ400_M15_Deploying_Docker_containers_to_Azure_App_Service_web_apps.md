---
lab:
    title: 'ラボ 15: Docker コンテナーを Azure App Service Web アプリにデプロイする'
    module: 'モジュール 15: Docker を使用したコンテナーの管理'
---

# ラボ 15: Docker コンテナーを Azure App Service Web アプリにデプロイする
# 受講生用ラボ マニュアル

## ラボの概要

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習します。 

## 目標

このラボを完了すると、次のことができるようになります。

- Microsoft がホストする Linux エージェントを使用してカスタム Docker イメージを構築する
- Azure Container Registry にイメージをプッシュする
- Azure DevOps を使用して Docker イメージをコンテナーとして、Azure App Service にデプロイする

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

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### 演習 1: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートと Azure リソース (Azure App Service Web アプリ、Azure Container Registry インスタンス、Azure SQL データベースなど) に基づくチーム プロジェクトで構成されています。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、Docker テンプレートに基づいて新しいプロジェクトを生成します。

> **注**: Docker テンプレートベースのプロジェクトは、コンテナー化された ASP.NET Core アプリをビルドして Azure App Service にデプロイします

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、[https://docs.microsoft.com/ja-jp/azure/devops/demo-gen ](https://docs.microsoft.com/ja-jp/azure/devops/demo-gen)をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページの「**新しいプロジェクト名**」テキストボックスに「**Deploying Docker containers to Azure App Service web apps**」と入力し、「**組織の選択**」ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで、「**DevOps Labs**」をクリックし、「**Docker**」テンプレートをクリックして、「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**Docker Integration**」ラベルの下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Azure リソースを作成する

このタスクでは、Azure Cloud Shell を使用して、このラボで必要な Azure リソースを作成します。

- Azure Container Registry
- Azure Web App for Containers
- Azure SQL Database

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure ADテナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal を表示している Web ブラウザーのツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。 

    >**注**: **Cloud Shell** を起動するのが初めてであり、「**ストレージがマウントされていません**」のメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージを作成**」 を選択します。 

1.  Cloud Shell ペインの Bash セッションから、以下を実行して、このラボでデプロイする Azure リソースをホストするリソースグループを作成します ( `<Azure_region>` プレースホルダーを、これらのリソースをデプロイする予定の 'eastus' などの Azure リージョンの名前に置き換えます)。

    ```bash
    LOCATION=<Azure_region>
    ```

    >**注**: `az account list-locations -o table` を実行すると、Azure リージョン名を識別できます。

1.  次のコマンドを実行して、Azure Container Registry インスタンス、Azure App Servic eプラン名、Azure Web アプリ名、Azure SQL データベース論理サーバー名、Azure SQL Database 名などの Azure リソースの名前を表す変数を作成します。

    ```bash
    RG_NAME='az400m1501a-RG'
    ACR_NAME=az400m151acr$RANDOM$RANDOM
    APP_SVC_PLAN='az400m1501a-app-svc-plan'
    WEB_APP_NAME=az400m151web$RANDOM$RANDOM
    SQLDB_SRV_NAME=az400m15sqlsrv$RANDOM$RANDOM
    SQLDB_NAME='az400m15sqldb'
    ```

    >**注**: `az account list-locations -o table` を実行すると、Azure リージョン名を識別できます。

1.  以下を実行して、このラボに必要なすべてのリソースに必要な Azure リソースを作成します。

    ```bash
    az group create --name $RG_NAME --location $LOCATION
    az acr create --name $ACR_NAME --resource-group $RG_NAME --location $LOCATION --sku Standard --admin-enabled true
    az appservice plan create --name 'az400m1501a-app-svc-plan' --location $LOCATION --resource-group $RG_NAME --is-linux
    az webapp create --name $WEB_APP_NAME --resource-group $RG_NAME --plan $APP_SVC_PLAN --deployment-container-image-name elnably/dockerimagetest
    IMAGE_NAME=myhealth.web
    az webapp config container set --name $WEB_APP_NAME --resource-group $RG_NAME --docker-custom-image-name $IMAGE_NAME --docker-registry-server-url $ACR_NAME.azurecr.io/$IMAGE_NAME:latest --docker-registry-server-url https://$ACR_NAME.azurecr.io
    az sql server create --name $SQLDB_SRV_NAME --resource-group $RG_NAME --location $LOCATION --admin-user sqladmin --admin-password Pa55w.rd1234
    az sql db create --name $SQLDB_NAME --resource-group $RG_NAME --server $SQLDB_SRV_NAME --service-objective S0 --no-wait 
    az sql server firewall-rule create --name AllowAllAzure --resource-group $RG_NAME --server $SQLDB_SRV_NAME --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

    >**注**: プロビジョニングプロセスが完了するまで待ちます。これにはおよそ 5 分かかる場合があります。

1.  以下を実行して、新しく作成された Azure Web アプリの接続文字列を構成します ($SQLDB_SRV_NAME および $SQLDB_NAME プレースホルダーは、それぞれ Azure SQL atabase 論理サーバーとそのデータベース インスタンスの名前の値に置き換わります)。
   
    ```bash
    CONNECTION_STRING="Data Source=tcp:$SQLDB_SRV_NAME.database.windows.net,1433;Initial Catalog=$SQLDB_NAME;User Id=sqladmin;Password=Pa55w.rd1234;"
    az webapp config connection-string set --name $WEB_APP_NAME --resource-group $RG_NAME --connection-string-type SQLAzure --settings defaultConnection="$CONNECTION_STRING"
    ```

1.  Azure portal を表示している Web ブラウザーで、「Cloud Shell」ペインを閉じ、「**リソース グループ**」ブレードに移動し、「**リソース グループ**」ブレードで **az400m1501a-RG** エントリを選択します。

1.  「**az400m1501a-RG**」リソース グループ ブレードで、リソースのリストを確認します。 

    >**注**: 論理 Azure SQL Database サーバーの名前を記録します。これは、このラボで後ほど必要になります。

1.  **az400m1501a** リソース グループ ブレードのリソース リストで、コンテナー レジストリ インスタンスを表すエントリをクリックします。

1.  「コンテナー レジストリ」ブレードの左側の垂直メニューの「**設定**」セクションで、「**アクセス キー**」をクリックします。

1.  コンテナー レジストリ インスタンスの「**アクセス キー**」ブレードで、**レジストリ名**、**ログイン サーバー**、**管理者ユーザー**、および**パスワード** エントリの値を特定します。

    >**注**: **レジストリ名**、**ログイン サーバー**、および**パスワード** エントリを記録します (レジストリ名と管理者ユーザー名は一致している必要があります)。これは、このラボで後ほど必要になります。


### 演習 2: Azure DevOps を使用して、Docker コンテナーを Azure App Service Web アプリにデプロイする

この演習では、Azure DevOps を使用して、Docker コンテナーを Azure App Service Web アプリにデプロイします。

#### タスク 1: 継続的インテグレーション (CI) と継続的デリバリー (CD) を構成する

このタスクでは、前の演習で生成した Azure DevOps プロジェクトを使用して、Docker コンテナーをビルドして Azure App Service Web アプリにデプロイする CI/CD パイプラインを実装します。

1.  ラボのコンピューターで、Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、**Deploying Docker containers to Azure App Service web apps**。Azure DevOps ポータルの左端にある垂直メニューバーで、「**パイプライン**」をクリックします。

    >**注**: まず、ビルド パイプラインを変更します。

1.  「**パイプライン**」ペインで、**MHCDocker.build** パイプラインを示しているエントリをクリックし、「**MHCDocker.Build**」ペインで「**編集**」をクリックします。

    >**注**: ビルド パイプラインは以下のタスクで構成されます

    | タスク | 使用法 |
    | ----- | ----- |
    | **Run Services** | 必要なパッケージを復元してビルド環境を準備します |
    | **Build services** | **myhealth.web** イメージを構築します |
    | **Push services** | **$(Build.BuildId)** でタグ付けされ **myhealth.web** イメージをコンテナー レジストリにプッシュします |
    | **Publish artifacts** | Azure DevOps の成果物を介してデータベース デプロイ用の dacpac を共有できます |

1. **MHCDocker.build** パイプライン の **Pipeline** セクションを選択し、**Agent Specification** を **Ubuntu-18.04** に変更します。

1.  **MHCDocker.build** パイプライン のタスクのリストで、「**Run Services**」タスクをクリックし、右側の「**Docker Compose**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、使用している Azure サブスクリプションを表すエントリを選択します。このラボで、「**Authorize**」をクリックして、対応するサービス接続を作成します。指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

    >**注**: この手順では、Azure サービス接続が作成され、サービス プリンシパル認証 (SPA) を使用してターゲット Azure サブスクリプションへの接続が定義・確立されます。 

1.  「**Azure Container Registry**」ドロップダウン リストから、このラボで先ほど作成した ACR インタンスを示しているエントリを選択します (**必要に応じてリストを更新する**か、ログイン サーバーの名前を入力してください)。

1.  前の 2 つの手順を繰り返して、**Build Services** タスクと**Push Services** タスクで **Azure サブスクリプション**と **Azure Container Registry** の設定を構成します。

1.  同じ「**MHCDocker.build** パイプライン」ペインのペインの上部で、「**Save & Queue**」ボタンの横にある下向きのキャレットをクリックし、「**Save**」をクリックして変更を保存し、もう一度プロンプトが表示されたら「**Save**」をクリックします。

    >**注**: 次に、リリース パイプラインを変更します。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。 

1.  「**パイプライン/リリース**」ペインで、**MHCDocker.release** エントリが選択されていることを確認し、「**Edit**」をクリックします。

1.  デプロイの **Dev** ステージを示す長方形の中にある「**2 Jobs、2 Tasks**」リンクをクリックします。

    >**注**: リリース パイプラインは以下のタスクで構成されます

    | タスク | 使用法 |
    | ----- | ----- |
    | **Azure SQL: DacpacTask の実行** | ターゲット スキーマとデータを含む dacpac 成果物を Azure SQL データベースにデプロイします |
    | **Azure App Service のデプロイ** | ビルド段階で生成された Docker イメージを、指定されたコンテナー レジストリからプルし、そのイメージを Azure App Service Web アプリにデプロイします |

1. パイプラインのタスクリストで、**DB Deployment** をクリックし、**Agent Pool** で **Azure Pipelines**、**Agent Specification** で **VS2017-win2016** を選択します。

1.  パイプラインのタスクのリスト で、「**Execute Azure SQL: DacpacTask**」タスクを選択します。

1.  右側の「**Azure SQL Database deployment**」ペインで「**Azure サブスクリプション**」ドロップダウン リストから、このタスクで先ほど作成した Azure サービス接続を示すエントリを選択します。

1. パイプラインのタスクリストで、**Web App deployment** をクリックし、**Agent Pool** で **Azure Pipelines**、**Agent Specification** で **Ubuntu-18.04** を選択します。

1.  パイプラインのタスクリストで、「**Azure App Service Deploy**」タスクをクリックし、**Azure サブスクリプション** ドロップダウン リストで、このタスクの前半で作成した Azure サービス接続を表すエントリを選択します。また、「**App Service 名**」ドロップダウンリストで、このラボの前半でデプロイした Azure App Service Web アプリを表すエントリを選択します。

    >**注**: 次に、デプロイに必要なエージェント プール情報を構成する必要があります。

1.  ペインの上部にある **「Valiables」** をクリックします。
1.  パイプライン変数のリストで、次の変数の値を設定します。

    | 変数 | 値 |
    | -------- | ----- |
    | ACR | このラボの前回の演習で記録した Azure Container Registry のログイン名 (**azurecr.io** サフィックスを含む) |
    | DatabaseName | **az400m15sqldb** |
    | パスワード | **Pa55w.rd1234** |
    | SQLadmin | **sqladmin** |
    | SQLServer | このラボの前回の演習で記録した Azure SQL Database 論理サーバー名 (**database.windows.net** サフィックスを含む) |

1.  ペインの右上隅にある「**Save**」ボタンをクリックして変更を保存し、再度プロンプトが表示されたら「**OK**」をクリックします。

#### タスク 2: コード コミットを使用して、ビルド パイプラインとリリース パイプラインをトリガーする 

この演習では、コード コミットを使用してビルド パイプラインとリリース パイプラインをトリガーします。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**リポジトリ**」をクリックします。 

    >**注**: 「**ファイル**」ペインが自動的に表示されます。 

1.  「**ファイル**」ペインで、**src/MyHealth.Web/Views/Home** フォルダーに移動し、**Index.cshtml** ファイルを表すエントリをクリックしてから、「**Edit**」をクリックして編集用に開きます。

1.  「**Index.cshtml**」ペインの **28** 行目で、「**JOIN US**」を「**CONTACT US**」に変更し、ペインの右上隅にある「**コミット**」をクリックし、確認を求められたら、もう一度「**コミット**」をクリックします。

    >**注**: このアクションにより、ソース コードの自動ビルドが開始されます。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」をクリックします。 
1.  「**パイプライン**」ペインで、コミットによってトリガーされたパイプライン実行を表すエントリをクリックします。
1.  「**MHCDocker.build**」ペインで、パイプラインの実行を表すエントリをクリックします。
1.  パイプライン実行の「**概要**」タブの「**ジョブ**」セクションで、**Docker** エントリをクリックし、表示されるペインで、ジョブが正常に完了するまで個々のタスクの進行状況を監視します。 

    >**注**: ビルドは Docker イメージを生成し、それを Azure Container Registry にプッシュします。ビルドが完了すると、その概要を確認できるようになります。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。 

1.  「**リリース**」ペインで、ビルドの成功によってトリガーされた最新のリリースを表すエントリをクリックします。

1.  「**MHCDocker.release > Release-1**」ペインで、**Dev** ステージを表す長方形を選択します。

1.  「**MHCDocker.release > Release-1 > Dev**」ペインで、正常に完了するまでリリース タスクの進行状況を監視します。 

    >**注**: このリリースでは、ビルド プロセスによって生成された Docker イメージがApp Service Web アプリにデプロイされます。リリースが完了すると、その概要とログを確認できます。 

1.  リリース パイプラインが完了したら、[Azure portal](https://portal.azure.com) を表示している Web ブラウザー ウィンドウに切り替えて、このラボで以前にプロビジョニングした Azure App Service Web アプリのブレードに移動します。
1.  App Service Web アプリで、ターゲット Web アプリを表す **URL** リンク エントリをクリックします。

    >**注**: これにより、ターゲット Web サイトを表示する新しい Web ブラウザー タブが自動的に開きます。

1.  CI/CD パイプラインをトリガーするために適用した変更を含め、ターゲット Web アプリに HealthClinic.biz Web サイトが表示されることを確認します。

### 演習 3: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュし、Azure DevOps を使用して Azure App Service にコンテナーとしてデプロイしました。
