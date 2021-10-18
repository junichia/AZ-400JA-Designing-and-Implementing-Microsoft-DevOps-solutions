---
lab:
    title: 'ラボ 09: Azure Artifacts によるパッケージ管理'
    module: 'モジュール 9: 依存関係管理戦略の設計と実装'
---

# ラボ 09: Azure Artifacts によるパッケージ管理
# 受講生用ラボ マニュアル

## ラボの概要

Azure Artifacts は、Azure DevOps での NuGet、npm、Maven パッケージの検出、インストール、公開を容易にします。ビルドなどの他の Azure DevOps 機能と緊密に統合されているため、パッケージ管理は既存のワークフローのシームレスな部分になります。

## 目標

このラボを完了すると、次のことができるようになります。

-  フィードを作成して接続する。
-  NuGet パッケージを作成して公開する。
-  NuGet パッケージをインポートする。
-  NuGet パッケージを更新する。

## ラボの所要時間

-   推定時間: **40 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Visual Studio 2019 Community Edition ([Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/) から入手可能)Visual Studio 2019 のインストールには、**ASP<nolink>.NET および Web 開発**、**Azure 開発**、**.NET Core クロスプラットフォーム開発** ワークロードを含める必要があります。これはすでにラボのコンピューターに事前インストールされています。

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### 演習 0ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これには、Azure DevOps Demo Generator テンプレートと Visual Studio 構成に基づいて事前構成された Parts Unlimited チーム プロジェクトが含まれます。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。

1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。

1.  「**新しいプロジェクトの作成**」 ページの 「**新しいプロジェクト名**」 テキストボックスに「**Package Management with Azure Artifacts**」と入力し、「**組織の選択**」 ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」 をクリックします。

1.  「**テンプレートの選択**」 ページのテンプレートのリストで、「**PartsUnlimited**」 テンプレートをクリックし、「**テンプレートの選択**」 をクリックします。

1.  「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Visual Studio で Parts Unlimited ソリューションを構成する

このタスクでは、ラボの準備をするために Visual Studio を構成します。

1.  Azure DevOps ポータルで **Package Management with Azure Artifacts** チーム プロジェクトを表示していることを確認します。 

    > **注**: [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts) URL に移動して、プロジェクト ページに直接アクセスすることもできます。ここで、`<your-Azure-DevOps-account-name>` プレースホルダーは、アカウント名を表します。 

1.  「**Package Management with Azure Artifacts**」 ペインの左側にある垂直メニューで、「**リポジトリ**」 をクリックします。

1.  「**ファイル**」 ペインで、「**Clone**」 をクリックし、URL をクリップボードにコピーします。

1. Windows 10 で、Visual Studio **2017** を起動します。

1. **Clone a Repository** をクリックします。
    
1. **Browse a repository** の下にある **Azure DevOps** をクリックします。
    
1. **Showing hosted repositories for** プルダウンに表示されているアカウントが Azure DevOps で使用しているアカウントであることを確認します。異なる場合は、プルダウンから「**Microsoft Work, School or personal account**」を選択して認証します。
    
1. リストボックスの **dev.azure.com** を展開し、その配下から自身の組織を展開する。
    
1. 表示されているプロジェクトの一覧から **Package Management with Azure Artifacts** 配下の **PartsUnlimited** 選択して **Clone** をクリックする。

1.  画面右側の **Solution Explorer** に複製されたリポジトリが表示されていることを確認してください。

1.  ラボで使用するために、Visual Studio ウィンドウを開いたままにします。

### 演習 1  Azure Artifacts の操作

この演習では、次の手順を使用して Azure Artifacts を操作する方法を学習します。

- フィードを作成して接続する。
- NuGet パッケージを作成して公開する。
- NuGet パッケージをインポートする。
- NuGet パッケージを更新する。

#### タスク 1: フィードを作成して接続する

このタスクでは、フィードを作成して接続します。

1.  Azure DevOps ポータルでプロジェクト設定を表示している Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、「**Artifacts**」 を選択します。

1.  **アーティファクト** ハブが表示された状態で、ペインの上部にある 「**+ Create Feed**」 をクリックします。 

    > **注**: このフィードは、組織内のユーザーが利用できる NuGet パッケージのコレクションであり、パブリック NuGet フィードと並んで配置されます。このラボのシナリオでは、Azure Artifacts を使用するためのワークフローに焦点を当てているため、実際のアーキテクチャと開発の決定は純粋に例示的なものです。  このフィードには、この組織のプロジェクト間で共有できる共通の機能が含まれます。 

1.  「**Create new feed **」 ペインの 「**Name**」 テキストボックスに「**PartsUnlimitedShared**」と入力し、「**Scope**」 セクションで 「**Organization**」 オプションを選択し、他の設定をデフォルト値のままにして、「**Create**」 をクリックします。 

    > **注**: このNuGet フィードに接続するユーザーは、環境を構成する必要があります。 

1.  **アーティファクト** ハブに戻り、「**Connect to feed**」 をクリックします。

1.  「**Connect to feed**」 ペインの 「**NuGet**」 セクションで 「**Visual Studio**」 を選択し、「**Visual Studio**」 ペインで**Source** に表示されている URL をコピーします。 

1.  **Visual Studio** ウィンドウに戻ります。

1.  Visual Studio ウィンドウで、「**Tools**」 メニュー をクリックし、ドロップダウン メニューで 「**NuGet Package Manager**」 を選択し、カスケード メニューで 「**Package manager settings**」 を選択します。

1.  「**Options**」 ダイアログ ボックスで、「**Package Sources**」 をクリックし、プラス記号をクリックして新しいパッケージ ソースを追加します。

1.  ダイアログ ボックスの下部にある 「**Name**」 テキストボックスで、「**パッケージ ソース**」 を **PartsUnlimitedShared** に置き換え、「**Source**」 テキストボックスで、コピーした URL を Azure DevOps ポータルに貼り付けます。 

1.  「**Update**」 をクリックし、「**OK**」 をクリックして追加を完了します。

    > **注**: Visual Studio が新しいフィードに接続されました。

1.  Visual Studio インスタンスを一旦閉じて、再度起動し、**PartsUnlimited** ソリューションを開きます。この演習の 3 番目のタスクで必要になります。

#### タスク 2: NuGet パッケージの作成と公開

このタスクでは、NuGet パッケージを作成して公開します。

1.  新しいパッケージソースの構成に使用した Visual Studio ウィンドウで、メイン メニューの 「**File**」 をクリックし、ドロップダウン メニューの 「**New**」 をクリックしてから、カスケード メニューの 「**Project**」 をクリックします。 

    > **注**: NuGet パッケージとして公開される共有アセンブリを作成して、他のチームがプロジェクト ソースを直接操作しなくても、アセンブリを統合して最新の状態に保つことができるようにします。

1.  「**Create a new project**」の検索テキスト ボックスを使用して**Class Library (.NET Framework)** テンプレートを見つけて選択し、「**Next**」 をクリックします。 

1.  「**新しいプロジェクトの作成**」 ペインの 「**クラス ライブラリ (.NET Framework)**」 ページで、次の設定を指定し、「**作成**」 をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | プロジェクト名 | **PartsUnlimited.Shared** |
    | 場所 | 既定値を受け入れる |
    | ソリューション | **Create new solution** |
    | ソリューション名 | **PartsUnlimited.Shared** |
    | フレームワーク | **「.NET Framework 4.5.1」** |

1.  Visual Studio インターフェイス内の 「**Solution Explorer**」 ウィンドウで、「**Class1.cs**」 を右クリックし、右クリック メニューで 「**Delete**」 を選択し、確認を求められたら 「**OK**」 をクリックします。

1.  Visual Studio インターフェイス内の 「**ソリューション エクスプローラー**」 ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、「**Properties**」 を選択します。

1.  **PartsUnlimited.Shared** プロパティ ペイン内で、**Target Framework** が **「.NET Framework 4.5.1」** に設定されていることを確認します。

1.  **Ctrl+Shift+B** を押してプロジェクトをビルドします。 

    > **注**: 次のタスクでは、**NuGet.exe** を使用して、ビルドされたプロジェクトから直接 NuGet パッケージを生成しますが、最初にプロジェクトをビルドする必要があります。

1.  Azure DevOps ポータルを表示している Web ブラウザーに切り替えます。 

1.  「**フィードに接続**」 ペインの 「**NuGet**」 セクションで、「**NuGet.exe**」 を選択します。これにより、「**NuGet.exe**」 ペインが表示されます。

1.  「**NuGet.exe**」 ペインで、「**Get a tools**」 をクリックします。

1.  「**Get a tools**」 ペインで、「**Step1 Download the latest Nuget**」 リンクをクリックします。これにより、「**Available NuGet Distribution Versions**」 ページを表示する別のブラウザー タブが自動的に開きます。

1.  「**Available NuGet Distribution Versions**」 ページで、nuget.exe のバージョン **v5.5.1** を選択し、実行可能ファイルをローカルの**ダウンロード** フォルダーにダウンロードします。

1.  **Visual Studio** ウィンドウに切り替えます。「**Solution Explorer**」 ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、右クリック メニューで 「**Open Folder in File Explorer**」 を選択します。

1.  ダウンロードした **nuget.exe** ファイルをこのフォルダに移動します。

1.  同じファイル エクスプローラー ウィンドウで、「**File**」 メニューを選択し、ドロップダウン メニューで 「**Windows PowerShell を開く**」 のカスケード メニューで 「**管理者として Windows PowerShell を開く**」 をクリックします。 

1.  「**管理者: Windows PowerShell**」 ウィンドウで、次のコマンドを実行して、プロジェクトから **「.nupkg」** ファイルを作成します。 

    > **注**: これは、デプロイ用に NuGet ビットをパッケージ化するためのショートカットです。NuGet は高度にカスタマイズ可能です。詳細については、[NuGet パッケージの作成ページ](https://docs.microsoft.com/ja-jp/nuget/create-packages/overview-and-workflowhttps:/docs.microsoft.com/ja-jp/nuget/create-packages/overview-and-workflow)を参照してください。

    ```
    ./nuget.exe pack ./PartsUnlimited.Shared.csproj
    ```

    > **注**: **管理者: Windows PowerShell** ウィンドウに表示される警告は無視してください。

    > **注**: NuGet は、プロジェクトから識別できる情報に基づいてケージを構築します。たとえば、名前が **PartsUnlimited.Shared.1.0.0.nupkg** であることに注意してください。そのバージョン番号は、アセンブリから取得されました。

1. **ls** コマンドで、**PartsUnlimited.Shared.1.0.0.nupkg** ファイルが作成されていることを確認します。
    
1.  **Visual Studio** ウィンドウに戻り、「**Solution Explorer**」 ウィンドウで 「**PartsUnlimited.Shared \ Properties**」 ノードを展開し、「**AssemblyInfo.cs**」 をクリックしてウィンドウの中央ペインで開き、コンテンツを確認します。

    > **注**: **AssemblyVersion** 属性は、アセンブリに組み込むバージョン番号を指定します。NuGet の各リリースには一意のバージョン番号が必要なため、パッケージの作成にこの手順を引き続き使用する場合は、ビルドの前にこれをインクリメントする必要があります。

1.  **管理者: Windows PowerShell** ウィンドウに切り替え、次のコマンドを実行して、パッケージを **PartsUnlimitedShared** フィードに公開します。

    > **注**: **API キー**を指定する必要があります。これは、空でない文字列にすることができます。ここでは **AzDO** を使用しています。プロンプトが表示されたら、Azure DevOps 組織にサインインします。 

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.0.0.nupkg
    ```
1.  Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、垂直ナビゲーションペインで 「**アーティファクト**」 を選択します。

1.  **アーティファクト** ハブ ペインで、左上隅のドロップダウンリストをクリックし、フィードのリストで、**PartsUnlimitedShared** エントリを選択します。

    > **注**: **PartsUnlimitedShared** フィードには、新しく公開された NuGet パッケージが含まれています。 

1.  NuGet パッケージをクリックして、詳細を表示します。

#### タスク 3: NuGet パッケージのインポート

このタスクでは、NuGet パッケージをインポートします。

1.  **Parts Unlimited** ソリューションを表示している **Visual Studio** ウィンドウに切り替えます。既に閉じてkる場合は、もう一つの Visual Studio を起動して、Parts Unlimited ソリューションを開いてください。

1. **PartsUnlimitedWebsite** 配下にある 「**References**」 ノードを右クリックし、右クリックメニューで 「**Manage NuGet Packages**」 を選択します。これにより、ウィンドウの中央ペインに 「**NuGet: PartsUnlimitedWebsite**」 タブが開きます。もし **Reference** ノードが見つからない場合は、「**Solution Explorer**」 ペインで、**PartsUnlimited.sln** ファイルを探してダブルクリックして開きます。

1.  「**NuGet: PartsUnlimitedWebsite**」 ペインで、「**Browse**」 タブをクリックし、ペインの右上隅にある 「**Package sourtce:**」 ドロップダウン リストで、「**PartsUnlimitedShared**」 を選択します。 

    > **注**: パッケージのリストは、追加した単一のパッケージのみで構成されます。

1.  パッケージを選択し、**PartsUnlimited.Shared** ペインで、「**Install**」 をクリックしてプロジェクトに追加します。

1.  プロンプトが表示されたら、「**Preview changes**」 ダイアログ ボックスで 「**OK**」 をクリックします。

1.  **Ctrl+Shift+B** を押してプロジェクトをビルドし、ビルドが正常に完了したことを確認します。 

    > **注**: NuGet パッケージはまだ値を追加していませんが、ワークフローが意図したとおりに機能することを確認できました。 

#### タスク 4: NuGet パッケージの更新

このタスクでは、NuGet パッケージを更新します。

1.  **PartsUnlimited.Shared** プロジェクト (NuGet ソース プロジェクトを含む) が開いている **Visual Studio** ウィンドウに切り替えます。

1.  「**ソリューション エクスプローラー**」 ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、右クリック メニューで 「**Add**」 を選択し、カスケード メニューで 「**New Item**」 を選択します。

1.  「**Add New Item - PartsUnlimitedShared**」 ダイアログ ボックスの **Visual C# Items** のリストで、「**Class**」 テンプレートが選択されていることを確認し、ダイアログ ボックスの下部にある 「**Name**」 テキストボックスに「**TaxService.cs**」と入力し、「**Add**」をクリックしてクラスを追加します。 

    > **注**: 他のチームが NuGet パッケージを簡単に操作できるように、税計算がこの共有クラスに統合され、一元管理されることを想定しています。

1.  中央ペインの **TaxService.cs** クラスのコードで、クラスの既存の定義を次のコードに置き換え、ファイルを保存します。

    ```c#
    namespace PartsUnlimited.Shared
    {
        public class TaxService
        {
            static public decimal CalculateTax(decimal taxable, string postalCode)
            {
                return taxable * (decimal).1;
            }
        }
    }
    ```

    > **注**: アセンブリ (およびパッケージ) を更新しているため、アセンブリのバージョンを更新する必要があります。

1.  Visual Studio ウィンドウの中央のペインで、「**AssemblyInfo.cs**」 タブをクリックして、対応するファイルのコンテンツを表示します。 

1.  **AssemblyInfo.cs** ファイルで、`[assembly: AssemblyVersion("1.0.0.0")]` を `[assembly: AssemblyVersion("1.1.0.0")]` に変更し、ファイルを保存します。
1.  **Ctrl+Shift+B** を押してプロジェクトをビルドします。

1.  **管理者: Windows PowerShell** ウィンドウに切り替え、次のコマンドを実行して NuGet パッケージを再パッケージ化します。 

    > **注**: 新しいパッケージのバージョン番号は更新されます。

    ```
    ./nuget.exe pack PartsUnlimited.Shared.csproj
    ```
1. **ls** コマンドで PartsUnlimited.Shared.1.1.0.nupkg が作成されていることを確認します。

1.  **管理者: Windows PowerShell** ウィンドウで、次のコマンドを実行して、更新されたパッケージを公開します。 

    > **注**: 公開されたアーティファクトのバージョン番号は、パッケージのバージョン更新を反映するように変更されています。

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.1.0.nupkg
    ```

1.  **PartsUnlimitedShared1.0.0** アーティファクトを表示している Azure DevOps ポータルに切り替えます。

1.  **PartsUnlimitedShared 1.0.0** アーティファクトペインで、「**Versions**」 タブをクリックし、バージョン **1.0.0** および **1.1.0** が含まれていることを確認します。 

1.  **PartsUnlimited** プロジェクトを表示している **Visual Studio** ウィンドウに戻ります。

1.  「**ソリューション エクスプローラー**」 ペインで、**PartsUnlimitedWebsite\Utils\DefaultShippingTaxCalculator.cs** に移動して選択します。これにより、ウィンドウの中央ペインでファイルが自動的に開きます。

1.  **DefaultShippingTaxCalculator.cs** ファイルのコードで、**20** 行目 の **CalculateTax** の呼び出しを見つけ、 `tax = CalculateTax(subTotal + shipping, postalCode);` を `tax = PartsUnlimited.Shared.TaxService.CalculateTax(subTotal + shipping, postalCode);` に置き換えます。

    > **注**: 元のコードはこのクラスの内部のメソッドを呼び出したため、行の先頭に追加するコードは、NuGet アセンブリからのコードにリダイレクトしています。ただし、このプロジェクトはまだ NuGet パッケージを更新していないため、1.0.0.0 を参照しており、これらの新しい変更を利用できないため、コードは正しくビルドされません。

1.  「**ソリューション エクスプローラー**」 ペインで、「**References**」 ノードを右クリックし、右クリック メニューで 「**Manage NuGet Packages**」 を選択します。

    > **注**: 「**Update**」 タブの内容で示されているように、NuGet は更新を認識しています。 

1.  「**NuGet: PartsUnlimitedWebsite**」 ペインで、「**Update**」 タブをクリックし、検索テキストボックスに「**PartsUnlimited.Shared**」と入力し、ペインの右側の 「**Version: Latest stable 1.1.0**」 ドロップダウン リストの横にある 「**Update**」 をクリックして、新しいバージョンをインストールします。 

    > **注**: 利用可能な NuGet の更新が多数ある場合がありますが、更新する必要があるのは **PartsUnlimited.Shared** のみです。パッケージが完全に更新できるようになるまで、少し時間がかかる場合があることに注意してください。エラーが発生した場合は、しばらく待ってから再試行してください。

1.  プロンプトが表示されたら、「**変更のプレビュー**」 ダイアログ ボックスで 「**OK**」 をクリックします。

1.  **F5** キーを押して、サイトを構築して実行します。期待どおりに機能することを確認します。

#### レビュー

このラボでは、次の手順を使用して Azure Artifacts を操作する方法を学習しました：

- 作成してフィードに接続しました。
- NuGet パッケージを作成して公開しました。
- NuGet パッケージをインポートしました。
- NuGet パッケージを更新しました。
