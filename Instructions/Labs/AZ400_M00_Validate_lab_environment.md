---
lab:
    title: 'ラボ: ラボ環境の確認'
    module: 'モジュール 0: ようこそ'
---

# ラボ: ラボ環境を検証する
# 受講生用ラボ マニュアル

## 手順

1. 講師または他のソースから新しい Azure Pass プロモーションコード (30 日間有効) を入手します。
2. プライベート ブラザー セッションを使用して、account.microsoft.com で新しい Microsoft アカウント (MSA) を取得するか、既存のアカウントを使用します。
3. 同じブラザー セッションを使用して、Microsoftazurepass.com に移動して、Microsoft アカウント (MSA) を使用して Azure Pass を引き換えます。[Microsoft Azure Pass の引き換え](https://www.microsoftazurepass.com/Home/HowTo?Length=5) 引き換えの手順に従います。 
4. 同じブラザー セッションを使用して、portal.azure.com に移動して、「Azure DevOps」 を検索します。結果ページで、「Azure DevOps Organizations」 をクリックします。 
5. 次に、「My Azure DevOps Organizations」 と呼ばれるリンクをクリックします (または、https://aex.dev.azure.com/ に移動します)。
6. 左側のドロップダウン ボックスで、「Microsoft アカウント」 の代わりに、「既定値」 を選択します。
7. 既定のディレクトリを使用して、新しい組織を作成します (画面右上隅の青色ボックスで確認できます)。 
8. 新しく作成した組織を選択し、画面の左側にある 「組織設定」 を選択します。
9. 「組織設定」 -> 「請求」 -> 「請求の設定」 の順に移動し、「Azure サブスクリプション」 を選択します。次に、「Azure Pass サブスクリプション」 を選択してから、「MS Hosted CI/CD」 を選択して、フィールド 「Paid parallel jobs」 を 1 に設定します。次に、下部の青色ボックスの 「保存」 をクリックします。 
10. 新しい設定がバック エンドに反映されるまで、少なとも 3 時間待ってから CI/CD の機能を使用します。それ以外の場合、「要求が最大数に達したため、エージェントは動作していません…」というメッセージが表示されます。
11. 任意の手順として、請求を有効な状態の新しく作成した組織を使用し、https://azuredevopsdemogenerator.azurewebsites.net/ を使用して、新しい事前定義されたプロジェクトを作成して、これを検証します。しばらく待ってから試行し、テスト ビルドを実行します。
