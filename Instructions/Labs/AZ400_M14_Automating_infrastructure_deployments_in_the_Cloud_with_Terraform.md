---
lab:
    title: 'ラボ 14b: Terraform および Azure Pipelines を使用してクラウドでのインフラストラクチャのデプロイを自動化'
    module: 'モジュール 14: Azure で利用可能なコード ツールとしてのサード パーティ インフラストラクチャ'
---

# ラボ 14b: Terraform および Azure Pipelines を使用してクラウドでのインフラストラクチャのデプロイを自動化
# 受講生用ラボ マニュアル

## ラボの概要

[Terraform](https://www.terraform.io/intro/index.html) は、インフラストラクチャを構築、変更、およびバージョン管理するためのツールです。Terraform は、既存および一般的なクラウド サービス プロバイダーだけでなく、カスタムの社内ソリューションも管理できます。

Terraform 構成ファイルは、単一のアプリケーションまたはデータセンター全体を実行するために必要なコンポーネントを記述します。Terraform は、目的の状態に到達するために何をするかを記述する実行プランを生成し、それを実行して説明されたインフラストラクチャを構築します。構成が変更されると、Terraform は変更内容を判断し、増分実行プランを作成できます。 

このラボでは、Terraform を Azure Pipeline に取り入れてインフラストラクチャをコードとして実装する方法を学びます。 

## 目標

このラボを完了すると、次のことができるようになります。

- Terraform を使用してインフラストラクチャをコードとして実装する
- Terraform および Azure Pipelines を使用して Azure でインフラストラクチャのデプロイを自動化する

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
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成されます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Terraform** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。

1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。

1.  「**新しいプロジェクトの作成**」ページで 「**新しいプロジェクト名**」テキストボックスに「**Automating infrastructure deployments with Terraform**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。

1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**Terraform**」テンプレートを選択して「**テンプレートの選択**」をクリックします。

1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**トークンの置換**」と「**Terraform**」の下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### 演習 1: Terraform および Azure Pipelines を使用してクラウドでのインフラストラクチャのデプロイを自動化

この演習では、Terraform と Azure Pipelines を使用して、インフラストラクチャをコードとして Azure にデプロイします

#### タスク 1: Terraform 構成ファイルを検証する

このタスクでは、PartsUnlimited Web サイトのデプロイに必要な Azure リソースのプロビジョニングにおける Terraform の使用を検証します。

1.  ラボのコンピューターを使い、Azure DevOps ポータルが表示され、「**Terraform を使用したインフラストラクチャ デプロイの自動化**」プロジェクトが開いている Web ブラウザーで、 Azure DevOps ポータルの一番左側にある垂直メニュー バーの「**Repos**」をクリックします。

1.  「**ファイル**」ペインで最上部にある**Master** エントリの隣にある下向きキャレット記号をクリックします。ブランチのドロップダウン リストで **terraform** ブランチを示しているエントリをクリックします。

    > **注**: **Terraform** ブランチが表示され、リポジトリのコンテンツ内で **Terraform** フォルダーが表示されていることを確認します。 

1.  **Terraform** リポジトリのフォルダー階層で **Terraform** フォルダーを拡張し、**webapp.tf** をクリックします。

1. **Edit** をクリックして、１１行目の下に、以下を挿入します。

    ```
    provider "azurerm" {
        features {} 
    }
    ```
1. **Commit** をクリックして保存します。

    > **注**: **webapp.tf** は terraform 構成ファイルです。Terraform は、HCL (Hashicorp Configuration Language) と呼ばれる、YAML に類似した独自のファイル形式を使用します。

    > **注**: この例では、Azure リソース グループ、アプリ サービス プラン、および Web サイトのデプロイで必要なアプリ サービスを作成します。Terraform ファイルは、Azure DevOps プロジェクトのソース コントロール リポジトリに追加されているので、これを使って必要な Azure リソースをデプロイできます。 
    
#### タスク 2: Azure CI Pipeline を使用してアプリケーションを構築する

このタスクでは、アプリケーションを構築し、ドロップと呼ばれる成果物として必要なファイルを公開します。

1.  Azure DevOps ポータルで、Azure DevOps ポータルの左側にある垂直メニュー バーで「**パイプライン**」をクリックします。その下で「**パイプライン**」を選択します。

1.  「**パイプライン**」ペインで「**Terraform-CI**」をクリックして選択します。「**Terraform-CI**」ペインで「**編集**」をクリックします。

    > **注**: この CI パイプラインには .NET Core プロジェクトをコンパイルするタスクがあります。パイプラインの DotNet タスクは依存関係を復元し、ビルド出力を構築してテストし、zip ファイル (パッケージ) に公開します。これを Web アプリケーションにデプロイできます。アプリケーション ビルドのほか、terraform ファイルを公開して成果物を構築し、CD パイプラインで利用できるようにする必要があります。**ファイルのコピー** タスクで Terraform ファイルを成果物ディレクトリにコピーするのはそのためです。

1.  **Terraform-CI** ペインで「**Tasks**」タブを確認した後、右側の「**Queue**」をクリックします。見つからない場合は「･･･」省略記号をクリックし、ドロップダウン メニューを開いて探してください。

1.  「**パイプラインの実行**」ペインで「**Run**」をクリックし、ビルドを開始します。 

1.  ビルド実行ペインの「**サマリー**」タブの「**ジョブ**」セクションで、「**Agent Job 1**」をクリックし、ビルド プロセスの進捗状況を監視します。

1.  ビルドに成功した後、ビルド実行ペインの「**Summary**」タブに戻り、「**Related**」セクションで「**Published 1、Consumed 1**」リンクをクリックします。「**Artifacts**」ペインが表示されます。

1.  「**Artifacts**」ペインの「**Published**」タブで**Drop** フォルダーを展開します。**PartsUnlimitedwebsite.zip** ファイルが含まれていることを確認してから、その **Terraform** サブフォルダーを拡張し、**webapp.tf** ファイルが含まれていることを確認します。 

#### タスク 3: Azure CD パイプラインで Terraform (IaC) を使用してリソースをデプロイする

このタスクでは、デプロイパイプラインの一環として Terraform を使用して Azure リソースを作成し、Terraform でプロビジョニングされた Azure アプリ サービス Web アプリに PartsUnlimited アプリケーションをデプロイします。

1.  Azure DevOps ポータルで、Azure DevOps ポータルの左側にある垂直メニューの「**パイプライン**」セクションで「**リリース**」をクリックします。**Terraform-CD** エントリが選択されていることを確認し、「**編集**」をクリックします。

1.  **「すべてのパイプライン」> 「Terraform-CD」** ペインの 「**Dev**」ステージを示す長方形で 「**1 job、8 tasks**」リンクをクリックします。

1.  「**Dev**」ステージのタスク リストで「**Azure CLI**」タスクを選択します。 

1.  「**Azure CLI**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、Azure サブスクリプションを示すエントリを選択してから「**承認**」をクリックし、該当するサービス接続を構成します。指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

    > **注**: 既定で、Terraform は「terraform.tfstate」という名前のファイルに状態をローカルで格納します。チームで Terraform を使用する際は、ローカル ファイルを使用すると Terraform の使用が複雑になります。Terraform は状態の共有を容易にするリモート データ ストアに対応しています。この場合は、**Azure CLI** タスクを使用して Azure ストレージ アカウントと BLOB コンテナーを作成して Terraform の状態を格納します。Terraform リモート状態の詳細については、[Terraform ドキュメント](https://www.terraform.io/docs/state/remote.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Azure PowerShell**」タスクを選択します。 

1.  「**Azure PowerShell**」ペインの 「**Azure Connection Type**」ドロップダウン リストで 「**Azure Resource Manager**」を選択します。「**Azure サブスクリプション**」ドロップダウン リストで、新しく作成された Azure サービス接続 (「利用可能な Azure サービス接続」 に含まれているもの) を選択します。

    > **注**: Terraform [バックエンド](https://www.terraform.io/docs/backends/)を構成するには、Terraform の状態をホストする Azure ストレージ アカウントへのアクセス キーが必要です。この場合、Azure PowerShell タスクを使用して、前のタスクでプロビジョニングされた Azure ストレージ アカウントのアクセス キーを取得します。`Write-Host "##vso「task.setvariable variable=storagekey]$key"`を使用して、後のタスクで使用できるパイプライン変数を作成しています。

1.  「**Dev**」ステージのタスク リストで「**Replace tokens in terraform file**」タスクを選択します。

    > **注**: **Webapp.tf** ファイルをよく確認すると、いくつかの値に「**__**」というサフィックスをプレフィックスが付いているのがわかります (例: `__terraformstorageaccount__`)。「**Replace Tokens**」タスクは、このような値を、リリース パイプラインで定義した変数値に置き換えます。
     
1.  **Install Terraform 0.12.3** タスクを使用して、Terraform の指定されたバージョンをインターネットまたはツール キャッシュからインストールし、Azure パイプライン エージェント (ホステッドまたはプライベート) の PATH の先頭に追加します。

1.  「**Dev**」ステージのタスク リストで「**Terraform のインストール**」タスクを選択して確認します。このタスクでは、指定したバージョンの Terraform がインストールされます。

    > **注**: Terraform を自動で実行する場合は通常、以下の 3 つのステージで構成されたコア プラン / 適用サイクルに焦点が当てられます:

    1.  Terraform 作業ディレクトリの初期化。
    1.  希望する構成に合わせるために現在の構成を変更する計画の策定。
    1.  計画ステージで判別された変更の適用。
    
    > **注**: リリース パイプラインで残りの Terraform タスクは、このワークフローを実装します。

1.  「**Dev**」ステージのタスク リストで「**Terraform: init**」タスクを選択します。 

1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 

1.  「**Terraform**」ペインで「**Container**」ドロップダウン リストに「**terraform**」と入力し、**Key** パラメーターが **terraform.tfstate** に設定されていることを確認します。 

    > **注**: `terraform init` コマンドは現在の作業ディレクトリで *.tf ファイルをすべて解析し、その処理にhちうようなプロバイダーを自動的にダウンロードします。この例では、Azure リソースをデプロイしているので、[Azure プロバイダー](https://www.terraform.io/docs/providers/azurerm/)をダウンロードします。`terraform init` コマンドの詳細については、[Terraform ドキュメント](https://www.terraform.io/docs/commands/init.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Terraform: plan**」タスクを選択します。

1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 

1. **Additional command arguments** テキストボックスrに `-out=tfplan.` を入力します。
    
    > **注**: `terraform plan` コマンドを使用して実行計画を作成します。Terraform は、構成ファイルで指定された希望する状態を実現するために必要なアクションを決定します。これにより、実際に変更を適用しなくても、どの変更が範囲内なのか確認できます。`terraform plan` コマンドの詳細については、[Terraform ドキュメント](https://www.terraform.io/docs/commands/plan.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Terraform: apply -auto-approve**」タスクを選択します。 

1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 

    > **注**: このタスクは `terraform apply` コマンドを実行してリソースをデプロイします。既定により、確認して続行するよう指示されます。ここではデプロイを自動化しているため、タスクには確認の必要がない `auto-approve` パラメーターが含まれています。 

1. **Additional command arguments** テキストボックスの内容を `-auto-approve tfplan` に書き換えます。

1.  「**Dev**」ステージのタスク リストで「**Azure App Service のデプロイ**」タスクを選択します。ドロップダウン リストから Azure サービス接続を選択します。

    > **注**: このタスクでは、前のステップの「**Terraform: apply -auto-approve**」タスクでプロビジョニングされた Azure App Service に PartsUnlimited パッケージをデプロイします。

1. 「**Agent job**」をクリックし、「Agent pool」ドロップダウンリストで、「**Azure Pipelines > windows-2019**」を選択します。

1.  **「すべてのパイプライン」>「Terraform-CD」** ペインで「**Save**」をクリックします。「**Save**」ダイアログ ボックスで「**OK**」をクリックし、右上コーナーで「**Create release**」をクリックします。 

1.  「**新しいリリースの作成**」ペインの「**Stages for a trigger change from automated to manual**」ドロップダウン リストで「**Dev**」をクリックします。「**Artifacts**」セクションの「**バージョン**」ドロップダウン リストでこのリリースの成果物のバージョンを示すエントリを選択して「**Create**」をクリックします。

1.  Azure DevOps ポータルで「**Terraform-CD**」ペインに戻り、新しく作成されたリリースを示す「**Release-1**」エントリをクリックします。 

1.  **「Terraform-CD」>「Release-1」** ブレードで、「**Dev**」ステージを示す長方形をクリックします。「**Dev**」ペインで「**デプロイ**」をクリックしてから再び「**デプロイ**」をクリックします。

1.  再び **「Terraform-CD」> 「Release-1」** ブレードで、「**Dev**」ステージを示す長方形をクリックし、デプロイ プロセスを監視します。 

1.  リリースの完了後、ラボのコンピューターで別の Web ブラウザー ウィンドウを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。 

1.  Azure portal で **App Services** のリソースを検索して選択します。「**App Services**」ブレードから、**pulterraformweb** で始まる名前の Web アプリに移動します。 

1.  Web アプリ ブレードで「**参照**」をクリックします。別の Web ブラウザー タブが開き、新しくデプロイされた Web アプリケーションが表示されます。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。


## レビュー

このラボでは、Azure Pipelines を使用して Azure 上で Terraform を使い、反復可能なデプロイを自動化する方法を学習しました。
