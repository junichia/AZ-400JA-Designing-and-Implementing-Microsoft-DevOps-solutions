---
lab:
    title: 'ラボ 12: LaunchDarkly と Azure DevOps を使用した機能フラグ管理'
    module: 'モジュール 12: 適切なデプロイ パターンの実装'
---

# ラボ 12: LaunchDarkly と Azure DevOps を使用した機能フラグ管理
# 受講生用ラボ マニュアル

## ラボの概要

[**LaunchDarkly**](https://launchdarkly.com/) は、サービスとして機能フラグを提供する継続的デリバリーのプラットフォームです。LaunchDarkly を使用すると、機能のロールアウトをコードのデプロイから分離し、機能フラグを大規模に管理できます。LaunchDarkly と Azure DevOps を統合すると、頻繁なリリースに関連した潜在的なリスクを最小限に抑えられます。リリースと開発プロセスをさらに統合するため、機能フラグロールアウトを Azure DevOps 作業項目にリンクすることができます。 

このラボでは、LaunchDarkly を活用して Azure DevOps で機能フラグの管理を最適化する方法を学びます。 

## 目標

このラボを完了すると、次のことができるようになります。

- LaunchDarkly で機能フラグを作成する
- LaunchDarkly と Web アプリケーションを統合する
- Azure DevOps リリース パイプラインで LaunchDarkly 機能フラグを自動的にロールアウトする

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
-   Visual Studio 2019 Community Edition ([Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/) から入手可能)Visual Studio 2019 のインストールには、**ASP.NET および Web 開発**、**Azure 開発**、**「.NET Core クロスプラットフォーム開発」** ワークロードを含める必要があります。これはすでにラボのコンピューターに事前インストールされています。

#### LaunchDarkly トライアル アカウントを設定する

Web ブラウザーを使用して [LaunchDarkly Web サイト](https://launchdarkly.com/) に移動し、トライアル アカウントを作成します。 

#### Azure DevOps 組織を設定します。

[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) の手順に従います。

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成されます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Parts Unlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」 ページで 「**新しいプロジェクト名**」 テキストボックスに「**LaunchDarkly**」と入力します。「**組織の選択**」 ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」 をクリックします。
1.  テンプレートのリストのツールバーで 「**DevOps Labs**」 をクリックし、「**LaunchDarkly**」 テンプレートを選択して 「**テンプレートの選択**」 をクリックします。
1.  再び 「**新しいプロジェクトの作成**」 ページで、欠落している拡張機能をインストールするよう指示されたら 「**LaunchDarkly Integration V2**」 ラベルの下にあるチェックボックスを選択し、「**プロジェクトの作成**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### 演習 1: LaunchDarkly を使用して Azure DevOps で機能フラグを構成する

この演習では、LaunchDarkly を使用して Azure DevOps で機能フラグを構成します。 

#### タスク 1: LaunchDarkly で機能フラグを作成する

このタスクでは、LaunchDarkly で機能フラグを作成します

1.  ラボのコンピューターで Web ブラウザーを起動して [LaunchDarkly Web サイト](https://app.launchdarkly.com/) に移動し、ご自分のマイクロソフトアカウントを使用してサインインします。アカウントが無い場合は、新たに　Signup してください。

1.  LaunchDarkly ポータルの左側にある垂直メニュー バーで、「**Feature Flags**」 をクリックします。

1.  「**Create Flag**」 をクリックします。

1.  「**Create Feature Flag**」 ペインで 「**Name**」 テキスト ボックスに「**Member portal**」と入力し、「**Sage Flag**」 ボタンをクリックします。

    > **注**: 「**Member portal**」という名前のフラグが作成されました。このフラグを使用して、ASP.NET MVC Web アプリで 「**Member portal**」 機能の可視性を判断するものと想定します。 

    > **注**: LaunchDarkly を使用中のアプリケーションに統合するには、SDK キーが必要です。 

1.  LaunchDarkly ポータルの左側の垂直メニュー バーで 「**Account Settings**」 をクリックします。 

    > **注**: 「**Account Settings**」 ペインの**Project**タブを開くと、事前に定義されたProjectが 2 つあります。**Test**と**Production**です。  このプロジェクトでは、Production 環境の SDK キーを使用します。 

1.  運用環境の SDK キーをコピーしてメモ帳に貼り付けます。これは、このラボで後ほど必要になります。

#### タスク 2: LaunchDarkly を使用中の アプリケーションに統合する

このタスクでは、LaunchDarkly を使用中の Web アプリケーションに統合します。

1.  ラボのコンピューターで、Azure DevOps ポータルが表示され、**LaunchDarkly** プロジェクトが開いている Web ブラウザー ウィンドウに切り替えます。プロジェクト ペインの左下コーナーで 「**プロジェクトの設定**」 をクリックします。

1.  「**プロジェクトの設定**」 という垂直メニュー バーの 「**リポジトリ**」 セクションで 「**リポジトリ**」 を選択します。

1.  「**すべてのリポジトリ**」 ペインで 「**LaunchDarkly**」 を選択します。

1.  「**Commit mention linking**」 と 「**Commit mention work item resolution**」 の設定を 「**On**」 にします。

1.  Azure DevOps ポータルの一番左にある垂直メニュー バーで 「**リポジトリ**」 をクリックし、「**ファイル**」 ペインで 「**Clone**」 をクリックします。

1.  「**Clone Repository**」 ペインで **IDE** の下向きキャレット記号をクリックし、ドロップダウン リストで 「**Visual Studio**」 をクリックします。これにより、Visual Studio が自動的に起動し、「**Azure DevOps**」 ダイアログ ボックスが開きます。 

1.  Visual Studio ウィンドウ内の 「**Azure DevOps**」 ダイアログ ボックスで 「**Clone**」 をクリックします。

1.  Visual Studio ウィンドウ内で最上部の 「**View**」 メニューをクリックし、ドロップダウン メニューで 「**Git changes**」 をクリックします。 グレーバックして選択できない場合は、画面の右側に表示されている **Solution Explorer** の下にある **Git changes** タブをクリックしてくだださい。

1.  Visual Studio ウィンドウ内の 「**Git changes**」 ペインの最上部にある**Master** ドロップダウン リストで、下向き矢印をクリックします。

1.  **master**ドロップダウン を展開し、「**Remotes**」 をクリックします。

1.  リモート ブランチのリストで 「**origin/launch-darkly**」 ブランチを右クリックして、「**Checkout**」 をクリックします。 

    > **注**: ローカル リポジトリのファイルが更新されるまで待ちます。

1. Visual Studio ウィンドウの下のバーの右下コーナーに表示されていることを確認し、「**Solution Explorer**」 ウィンドウに切り替えます。

1. 「**PartsUnlimited.sln**」 ファイルをダブルクリックしてソリューションを開きます。 

    > **注**: **LaunchDarkly** と .NET アプリケーションを統合するには、**LaunchDarkly クライアント**を使用して NuGet パッケージをインストールする必要があります。現在のプロジェクトでは、使いやすいようにこのパッケージはすでに追加されています。

1.  「**ソリューション エクスプローラー**」 ペインで **PartsUnlimitedWebsite** ノードを展開し、**Dependencies** サブノードを右クリックします。右クリック メニューで 「**Manage NuGet Packages**」 エントリを選択します。

1.  「**NuGet: PartsUnlimitedWebsite**」 ペインで、**LaunchDarkly.Client** がすでにインストールされていることを確認します。

1.  Visual Studio のツールバーに表示されている「**IIS Express**」 をクリックし、アプリケーションをローカルで起動します。 

1.  ローカル ブラウザー セッションでアプリケーションが起動し、「**Member portal**」 セクションが Web インターフェイスの右上コーナーに表示されていることを確認します。

    > **注**: この**メンバー ポータル** モジュールは新しい機能で、**LaunchDarkly 機能フラグ**を使用してこれをコントロールするものと想定します。これにより、LaunchDarkly をオンにすると、この機能がユーザーに表示されるはずです。

1.  Web アプリケーション インターフェイスの表示されている Web ブラウザー ウィンドウを閉じます。

1.  Visual Studio ウィンドウの**Solution Explorer**で、**PartsUnlimitedWebsite\\Controllers\\HomeController.cs** をクリック開きます。その内容を [the updated HomeController.cs code snippet](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/HomeController.cs) に表示されるコードに置き換え、変更を保存します。 

1.  Visual Studio ウィンドウの**Solution Explorer**で、**PartsUnlimitedWebsite\\Controllers\\AccountController.cs** に移動してこれを開きます。その内容を [the updated HomeController.cs code snippet](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/AccountController.cs) で入手できるコードに置き換え、変更を保存します。 
1.  Visual Studio ウィンドウの**ソリューション エクスプローラー**で、**PartsUnlimitedWebsite\\Views\Shared\\_Layout.chtml** に移動してこれを開き、ライン 55 の `@await Html.PartialAsync("_Login")` を以下のコードに置き換えます。

    ```cshtml
    @if (ViewBag.togglevalue == true)
    {
      @await Html.PartialAsync("_Login")
    }
    ```

1.  **HomeController.cs** ファイルの内容を表示しているタブに切り替え、`static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` ラインのプレースホルダ― `__YourLaunchDarklySDKKey__` を LaunchDarkly アカウントの SDK キー (この演習の以前のタスクでコピーしたもの) に置き換えます。 

    > **注**: これにより環境固有の SDK キーで LdClient オブジェクトが作成されます。

1.  前の手順を繰り返し、**AccountController.cs** ファイルのコードを更新します。 

    > **注**: 上記の変更により、静的 LaunchDarkly クライアントを初期化すると **HomeController** が起動するようになります。**MemberPortal** の表示方法は、機能フラグが LaunchDarkly がオンかオフかを確認できるよう修正されています。**_Layout.cshtml** ページはトグル値をチェックし、フラグがオンの場合は MemberPortal リンクをレンダリングします。 

1.  Visual Studio ウィンドウで、「**HomeController.cs**」 タブに戻り、内容を確認します。ライン 57 とライン 74 の間のコードに注目してください:

    ```
        //LaunchDarkly start
        User user = LaunchDarkly.Client.User.WithKey("administrator@test.com");
        bool value = client.BoolVariation("member-portal", user, false);
        if (value)
        {
          ViewBag.Message = "Your application description page.";
          ViewData["togglevalue"] = value;
          return View(viewModel);
        }
        else
        {
          return View(viewModel);
        }
        // return View(viewModel);

    }
    //LaunchDarkly End
    ```

    > **注**: 機能フラグを要求する場合は、ユーザー オブジェクトでパスする必要があります。このため、最初にユーザー オブジェクトを初期化します。これは、指定されたキーのユーザーが LaunchDarkly にいるかどうかを確認するために使われます。このサンプルでは、ユーザー値をハードコードしています。実際のシナリオでは、ログインしているユーザーを特定するか、データベース検索を行うと、この値を取得できます。

    > **注**: その後、**BoolVariation** メソッドを呼び出して LaunchDarkly で機能フラグの値をチェックします。フラグが true の場合は、**[ViewData["togglevalue"]** が true に設定されます。これは **_Layout.chtml** でメンバー ポータル モジュールを表示するために使われます。false の場合、メンバー ポータル モジュールは表示されません。

    > **注**: **AccountController.cs** の場合と同様に、LaunchDarkly コードはすでに **Login()** メソッドに追加されています。このため、**メンバー ポータル** アイコンをクリックすると、ログイン ページが表示されます。LaunchDarkly のフラグが false の場合は、ログイン ページで HttpNotFound エラーが返されます。

1.  Visual Studio ウィンドウで 「**すべて保存**」 をクリックし、「**IIS Express**」 をクリックしてアプリケーションをローカルで起動します。 

1.  Parts Unlimited Web サイトを表示している Web ブラウザーにもう**Member Portal** リンクが表示されていないことを確認します。この時点で **Member Portal** フラグはオフになっています。

    > **注**: これにより、LaunchDarkly を使用して機能フラグのコントロールが実装されます。これで LaunchDarkly ポータルからのトグルを手動で有効にできます。ただし、このラボでは、LaunchDarkly 拡張機能を使用して Azure DevOps リリース パイプラインでこれを構成します。リリース プロセスの一部として機能フラグを含めるには、この変更を Azure DevOps 作業項目に関連付けます。 

1.  Web アプリケーション インターフェイスの表示されている Web ブラウザー ウィンドウを閉じます。

1. Visual Studio で、View メニューから **Team Explorer** を選択します。

1.「**Team Explorer**」 ペインのツールバーにある 「**Home**」 アイコンをクリックし、「**Work Items**」 をクリックします。

1.「**Work Items**」 ページで 「**My Query**」 を右クリックし、右クリック メニューで 「**New Query**」 をクリックします。これにより、Visual Studio ウィンドウの中央のペインで 「**New Query**」 タブが自動的に開きます。

1.  「**新しいクエリ**」 ペインで既定の設定のまま、「**Run**」 をクリックします。このブランチに関連付けられている単一の作業項目が返されます。

1.  クエリ結果で**作業項目 ID** をメモし、その作業項目を示しているエントリをダブルクリックします。

1. Visual Studio ウィンドウの中央のペインに別のタブが自動的に開き、その作業項目に該当するユーザー ストーリーが表示されます。

1. ユーザー ストーリー タブの左上コーナーで 「**Assign To**」が「**未割り当て**」の場合は、Azure DevOps ユーザー名を入力して **Enter** キーを押し、作業項目を自分に割り当てます。その後、「**Save Work Item**」 をクリックして変更を保存します。

4.  Visual Studio ウィンドウで 「**Git Change**」 ペインに切り替え、「**Enter a message**」 テキストボックスに「**Integated LaunchDarkly #<作業項目ID>**」と入力します。「**Commit All**」 ボタンの隣にある下向き矢印をクリックし、ドロップダウン リストで 「**Commit All and Push**」 をクリックします。 

#### タスク 3: リリース中に自動的に LaunchDarkly 機能フラグをロールアウトする

このタスクでは、Azure DevOps リリース中の LaunchDarkly 機能フラグの自動ロールアウトを構成します。

> **注**: Azure DevOps サービスと統合するには、LaunchDarkly アクセス トークンが必要です。 

1.  LaunchDarkly アクセス トークンを取得するには、LaunchDarkly ポータルを表示しているブラウザー ウィンドウに戻ります。左側の垂直メニューで 「**Account Settings**」 をクリックし、「**Authorization**」 タブをクリックします。

1.  「**アクセス トークン**」 セクションで 「**Create tokens**」 をクリックします。

1.  「**アクセス トークンの作成**」 ペインで 「**名前**」 テキストボックスに「**member portal**」と入力します。「**Role**」 ドロップダウン リストで 「**writer**」 を選択し、「**Save token**」 をクリックします。

1.  トークンをコピーし、メモ帳に貼り付けます。 

    > **注**: 必ずトークンをコピーしてください。現在のページを閉じると、取得できなくなります。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに戻り、「**LaunchDarkly**」 プロジェクト ペインで 「**プロジェクトの設定**」 をクリックします。 

1.  「**プロジェクトの詳細**」 ペインの 「**パイプライン**」 セクションで 「**Service connections**」 をクリックしてから 「**New service connection**」 をクリックします。

1.  「**新しいサービス接続**」 ペインで 「**LaunchDarkly**」 をクリックしてから 「**次へ**」 をクリックします。

1.  「**新しい LaunchDarkly サービス接続**」 ペインで 「**アクセス トークン**」 テキスト ボックスに、このタスクで以前に取得しておいたアクセス トークンを貼り付けます。「**Service connection name**」 に「**az-400 m12l01 LaunchDarkly**」と入力して 「**保存**」 をクリックします。

1.  Azure DevOps ポータルの左側の垂直メニューで 「**Boards**」 をクリックします。「**Work items**」 ペインで、以前のタスクで自分に割り当てておいた作業項目をクリックします（規定で表示されます）。

1.  「**Implement FeatureFlag Management using LaunchDarkly**」をクリックし、右側コーナーの 「**LaunchDarkly**」 タブを選択します。 

1.  「**LauchDarkly**」 タブの 「**Select a feature flag**」 セクションで、「**member portal**」 機能フラグを選択します。

1.  Azure DevOps ポータルの左側にある垂直メニュー バーで 「**パイプライン**」 をクリックし、「**パイプライン**」 セクションで 「**リリース**」 を選択します。 

1.  「**パイプライン / リリース**」 ペインで 「**LaunchDarkly_CD**」 パイプラインを選択し、「**Edit**」 をクリックします。

1.  「**すべてのパイプライン / LaunchDarkly_CD**」 ペインで 「**Stage 1**」 段階の 「**1 job、3 tasks**」 リンクをクリックし、パイプラインのタスクを表示します。
1.  ステージに次のタスクが含まれていることを確認します。

    - **Azure Resource Group Deploy**: このタスクでは、ARM テンプレートを使用して PartsUnlimited Web サイト向けに Azure アプリ サービスをデプロイします。
    - **LaunchDarkly Rollout**: このタスクでは、LaunchDarkly サブスクリプションで機能フラグを有効にします。
    - **Azure App Service Deploy**: このタスクでは、最初のタスクで作成した Azure アプリ サービスに PartsUnlimited Web アプリをデプロイします。

1.  タスクのリストで 「**Azure Resource Group Deploy**」 タスクを選択し、「**Azure サブスクリプション**」 ドロップダウン リストで下向きキャレット記号をクリックします。エントリのリストで、このタスクで使用したい Azure サブスクリプションを選択し、「**Authorize**」 をクリックして Azure サービス接続を構成します。指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

1.  タスクのリストで 「**LaunchDarkly Rollout**」 タスクを選択します。「**Account**」 ドロップダウン リストで、このタスクで以前に作成した LaunchDarkly サービス接続を示しているエントリを選択します。

    > **注**: 「**フラグの状態**」 が 「**オン**」 になっていることを確認します。この設定により、リリース中のフラグの状態が管理されます。

1.  タスクのリストで 「**Azure App Service Deploy**」 タスクを選択します。「**Azure サブスクリプション**」 ドロップダウン リストで、以前に作成した Azure サブスクリプション サービス接続を選択します。 

1. Save をクリックして保存します。

1.  Azure DevOps ポータルの Azure DevOps ページの右上コーナーで 「**ユーザー設定**」 アイコンをクリックします。ドロップダウン メニューで 「**Personal Access Tokens**」 をクリックし、「**個人用アクセス トークン**」 ペインで 「**New Token**」 をクリックします。
4.  「**新しい個人用アクセス トークンの作成**」 ペインで以下の設定を指定し、「**Create**」 をクリックします (他はすべて規定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **LaunchDarkly および Azure DevOps ラボ** |
    | スコープ | **フル アクセス** |

1.  「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**: 必ずトークンをコピーしてください。このペインを閉じると、取得できなくなります。 

1.  「**成功**」ペインで「**閉じる**」をクリックします。

1.  「**All pipelines / LaunchDarkly_CD**」 ペインの編集画面に戻り、「**Valiables**」 タブをクリックします。 

1.  変数リストで **launchdarkly-pat** 変数の値を、新しく生成した個人用アクセス　トークンに設定します。

1.  変数リストで **launchdarkly-project-name** 変数の値を **LaunchDarkly** に設定します。

    > **注**: これによりリリース パイプラインの構成が完了します。 

1. **Save** をクリックして保存します。

1.  Azure DevOps ポータルの左側にある垂直メニュー バーの 「**パイプライン**」 セクションで、「**パイプライン**」 を選択します。

1.  「**パイプライン**」 ペインで、**LaunchDarkly-CI** ビルド パイプラインを示すエントリをクリックします。「**LaunchDarkly-CI**」 ペインで 「**パイプラインの実行**」 をクリックし、「**Run pipeline**」 ペインで 「**Run**」 をクリックします。

    > **注**: この CI パイプラインは .Net Core プロジェクトをコンパイルします。ビルドが完了すると、リリースがトリガーされ、アプリがデプロイされて LaunchDarkly で機能フラグがロールアウトされます。

1.  ビルド パイプライン実行ペインの 「**ジョブ**」 セクションで 「**Agent Job 1**」 エントリをクリックし、ビルド プロセスの進捗状況を監視します。

1.  ビルドの完了後、Azure DevOps ポータルの左側にある垂直メニュー バーの 「**パイプライン**」 セクションで、「**リリース**」 を選択します。

1.  「**パイプライン / リリース**」 ペインの 「**LaunchDarkly_CD**」 ペインで 「**Release-1**」 エントリをクリックし、デプロイ プロセスの進捗状況を監視します。

    > **注**: デプロイの完了後は、LaunchDarkly ダッシュボードで **MemberPortal** 機能フラグがオンになっていることを確認できるはずです。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで所有者のロールがあるユーザー アカウントを使ってサインインします。

1.  Azure portal で **App Services** のリソースを検索して選択します。「**App Services**」 ブレードから、Azure DevOps リリース パイプラインを介してアプリケーションをデプロイした Web アプリに移動します。

1.  Web アプリ ブレードで「**参照**」をクリックします。別の Web ブラウザー タブが開き、新しくデプロイされた Web アプリケーションが表示されます。

1.  Parts Unlimited の Web ページで、「**member portal**」 機能が有効になっていることを確認します。

> **注**: LaunchDarkly には、このほかにも以下のような多数の機能が含まれています。
    
- **ユーザーのターゲット**: LaunchDarkly ターゲットでは、各ユーザーやユーザー グループの機能をオンまたはオフにできます。これを使用すると、より広範なロールアウトを行う前に内部テストやプライベート ベータ、または使用可能性テストで機能をロールアウトできます。 
- **カスタム ターゲット規則**: LaunchDarkly では個々のユーザーをターゲットにできるだけでなく、カスタム規則を構築してユーザーのセグメントをターゲットにすることも可能です。つまり、カスタム規則を作成し、指定した属性に基づいてユーザーをターゲットにできます。 
- **開発プロセスを管理するためのプロジェクトと環境**: [プロジェクト](https://docs.launchdarkly.com/docs/projects) では、ひとつの LaunchDarkly アカウントで複数の様々なソフトウェア プロジェクトを管理できます。[環境](https://docs.launchdarkly.com/docs/environments) では、ローカル開発から QA、ステージング、運用にいたるまで開発サイクル全体で機能フラグを管理できます。 

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure portal を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、このラボで以前にデプロイした App Service のインスタンスまでナビゲートします。 
1.  App Service インスタンス ブレードで、App Service インスタンスを含むリソース グループの名前を示すリンクをクリックします。
1.  App Service インスタンスが含まれているリソース グループのブレードで、「**リソース グループの削除**」 をクリックします。指示されたら、リソース グループの名前を提供し、「**削除**」 をクリックします。

## レビュー

このラボでは、LaunchDarklyを活用して Azure DevOps で機能フラグの管理を最適化する方法を学びました。 
