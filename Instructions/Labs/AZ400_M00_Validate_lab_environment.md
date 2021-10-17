---
lab:
    title: 'ラボ 00: ラボ環境を検証する'
    module: 'モジュール 0: ようこそ'
---

# ラボ 00: ラボ環境を検証する
# 受講生用ラボ マニュアル

## 手順

#### Azure Pass の有効化
1. 講師またはその他のソースから、新しい **Azure Pass プロモーションコード** (30 日間有効) を入手します。
2. プライベート ブラウザー セッションを使用して、[Microsoft アカウント で](https://account.microsoft.com)新しい **Microsoft アカウント (MSA)** を取得するか、既存のアカウントを使用します。
3. 同じブラウザー セッションを使用して、[Microsoft Azure Pass](https://www.microsoftazurepass.com) にアクセスし、Microsoft アカウント (MSA) を使用して Azure Pass を利用します。詳細については、「[Microsoft　Azure　パスの引き換え](https://www.microsoftazurepass.com/Home/HowTo?Length=5)」を参照してください。引き換えの手順に従います。 

#### Azure DevOps の有効化
1. [Microsoft Azure](https://portal.azure.com) にアクセスします。

1. ポータル画面の右上で自分がログインしている**Azure AD ディレクトリの名前（**既定のディレクトリ、Default Directory 等**）を確認します

3. ポータル画面の上部で **Azure DevOps** を検索します。表示されたページで、「**Azure DevOps Organizations**」をクリックします。 

4. 次に、**My Azure DevOps Organizations** というリンクをクリックします（または [自分の情報](https://aex.dev.azure.com)に直接移動します)。

5. 左側のドロップダウン ボックスで、「Microsoft アカウント」 の代わりに先ほど確認した「**自分がログインしているディレクトリ（多くの場合、「既定のディレクトリ」のはずです）**」を選択します。

6. プロンプト (*「さらに詳細が必要です」*) が表示されたら、名前子メールアドレス、場所を入力して、「**Continue**」 をクリックします。

7. 「**既定のディレクトリ**」 を選択した状態で[自分の情報](https://aex.dev.azure.com)に戻り、青いボタン 「**Create new organization（新しい組織の作成）**」 をクリックします。

8. 「**Continue（続行）**」 をクリックして*利用規約*に同意します。

9. プロンプト (*「Almost done（ほぼ完了）」*) が表示されたら、Azure DevOps　組織の名前を既定のままにし (グローバルに一意の名前である必要があります)、一覧から最寄りのホスティング場所（East Asia）を 選択します。「**Continue**」をクリックします。

11. 組織が作成され、画面に「**Create a project to get started**」が表示されたら、左下隅にある 「**Organization settings（組織の設定）**」 をクリックします。

12. 「**Organization settings（組織の設定）**」 画面で、「**Billing（課金）**」 をクリックします (この画面を開くには数秒かかります)。

13. 「**Set up billing（課金の設定）**」 をクリックし、画面の右側で **「Azure Pass」 - 「スポンサーシップ** サブスクリプション」 を選択し、「**Save（保存）**」 をクリックしてサブスクリプションを組織にリンクします。

14. 画面の上部にリンクされた Azure サブスクリプション ID が表示されたら、**MS Hosted CI/CD** の**Paid parallel jobs(有料並列ジョブ)**の数を 0 から **1** に変更します。次に、下部にある 「**Save(保存)**」 をクリックします。 

15. 新しい設定がバックエンドに反映されるように、**CI/CD 機能を使用する前に少なくとも 3 時間待ちます**。それ以外の場合は、*「リクエストの最大数に達したため、このエージェントは実行されていません…」* というメッセージが表示されます。

17. オプション: ビルド パイプラインを作成してトリガーすることで、この新しい設定が正常に適用されたことを検証できます。これを行うには、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) を使用して、講師に相談するか、課金を有効にして新しく作成した組織にデモプロジェクトを作成します。
