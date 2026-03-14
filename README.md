# 同じHTMLファイルを「全公開」にも「認証付き」にもできるデモ

## 🔓 認証なし vs 🔐 認証あり — 違いは設定ファイル1つだけ

```
┌──────────────────────────────────────────────────────────────────┐
│                     同じ HTML ファイル                             │
├────────────────────────────┬─────────────────────────────────────┤
│                            │                                     │
│  � Azure（認証なし）       │  🔐 Azure（認証あり）                │
│                            │                                     │
│  設定ファイルなし            │  staticwebapp.config.json あり       │
│                            │                                     │
│  → URLを知っていれば        │  → Entra ID でログインした人だけ      │
│    世界中の誰でも閲覧可能    │    ページを閲覧できる                 │
│                            │                                     │
│  🌍 全公開                  │  🏢 社内限定公開                     │
│                            │                                     │
└────────────────────────────┴─────────────────────────────────────┘
```

> **ポイント：** 両方とも Azure Static Web Apps にデプロイされた同じHTMLファイル。
> 違いは **設定ファイル（`staticwebapp.config.json`）を1つ追加するかしないか** だけ。それだけで Entra ID による認証が実現します。

---

## 🎯 実際に見比べてみてください

| | 🔓 Azure（認証なし・全公開） | 🔐 Azure（認証付き） |
|---|---|---|
| **URL** | [認証なしサイト](https://mango-tree-0ee71ed10.6.azurestaticapps.net) | [認証ありサイト](https://gentle-pebble-053949610.1.azurestaticapps.net) |
| **アクセス** | 誰でも見れる | Entra ID ログインが必要 |
| **シークレットブラウザで開くと** | そのまま表示される | ログイン画面が表示される ✋ |
| **設定ファイル** | なし | `staticwebapp.config.json` あり |
| **GitHub** | [azure-no-auth-demo](https://github.com/FukudaRikako/azure-no-auth-demo) | [azure-entra-auth-demo](https://github.com/FukudaRikako/azure-entra-auth-demo) |

> **HTMLファイルは完全に同じ。** 違いは `staticwebapp.config.json` が **あるかないか** だけ！

---

## 🛡️ どうやって保護しているの？

HTMLファイルと一緒に、この設定ファイル（`staticwebapp.config.json`）を **1つ置くだけ**：

```json
{
  "routes": [
    {
      "route": "/*",
      "allowedRoles": ["authenticated"]
    }
  ],
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad",
      "statusCode": 302
    }
  }
}
```

**たったこれだけで：**
- ✅ 全ページにログインを要求
- ✅ 未認証ユーザーを Entra ID ログインに自動リダイレクト
- ✅ HTMLファイルの変更は一切不要

---

## 🔄 仕組み

```
                    ┌─ GitHub（全公開）─── 🔓 誰でもソースが見れる
                    │
同じ HTML ファイル ──┤
                    │
                    └─ Azure Static Web Apps ── 🔐 Entra ID 認証
                                                    │
                                          ┌─────────┴──────────┐
                                          │ ログイン済み？        │
                                          │  Yes → ページ表示    │
                                          │  No  → ログイン画面  │
                                          └────────────────────┘
```

---

## 💡 Azure Static Web Apps のメリット

| メリット | 説明 |
|---|---|
| **設定ファイル1つで認証** | HTMLの変更不要。JSONファイルを追加するだけ |
| **Entra ID 統合** | 社内のID基盤をそのまま利用できる |
| **GitHubから自動デプロイ** | pushするだけでサイトが更新される |
| **無料** | Free プランで認証機能が使える |

---

## ❓ なぜGitHubを使うの？直接Azureにアップロードじゃダメ？

GitHub Copilot で作ったHTMLを **Azureに直接アップロードすることも可能**です。
GitHubは必須ではありませんが、間に挟むことでこんなメリットがあります：

| GitHubを使うメリット | 説明 |
|---|---|
| **変更履歴が残る** | いつ・誰が・何を変えたか全部記録される。前のバージョンに戻せる |
| **自動デプロイ** | GitHubにファイルを置くだけでAzureに自動反映。手動アップロード不要 |
| **チームで共同編集** | 複数人で同じファイルを安全に編集できる |
| **レビューできる** | 変更を公開前に誰かにチェックしてもらえる |

> 💡 今回のデモでは、GitHub を使うことで **ファイルを変更するたびにAzureへ自動反映される仕組み** も一緒に実現しています。
> 個人で使うだけ・ファイルが少ないなど、シンプルな用途なら GitHub なしで直接デプロイでもOKです。

---

## 📖 やり方ガイド（Step by Step）

### ① Azure上でHTMLを全体公開する方法

HTMLファイルをインターネットに公開するまでの手順です。プログラミングの知識は不要です。

#### Step 1 ― GitHubアカウントを作る

1. https://github.com にアクセス
2. 「Sign up」からアカウントを作成（メールアドレスが必要）

#### Step 2 ― GitHubにリポジトリ（ファイル置き場）を作る

1. GitHub にログインした状態で https://github.com/new を開く
2. 「Repository name」に好きな名前を入力（例：`my-website`）
3. 「**Public**」を選択（公開設定）
4. 「**Create repository**」をクリック

#### Step 3 ― HTMLファイルをアップロードする

1. 作成したリポジトリのページで「**uploading an existing file**」のリンクをクリック
2. 公開したいHTMLファイルをドラッグ＆ドロップ
   - ⚠️ ファイル名は **`index.html`** にしてください
3. 「**Commit changes**」をクリック

#### Step 4 ― Azure PortalでStatic Web Appを作る

1. https://portal.azure.com にサインイン
2. 上部の検索バーに「**Static Web Apps**」と入力して選択
3. 「**＋ 作成**」をクリック
4. 以下の項目を入力：
   - **サブスクリプション**：使用するサブスクリプションを選択
   - **リソースグループ**：「新規作成」→ 好きな名前（例：`rg-my-website`）
   - **名前**：好きな名前（例：`my-website`）
   - **プランの種類**：Free
   - **ソース**：GitHub
5. 「**GitHubでサインイン**」をクリックして認証
6. 以下を選択：
   - **組織**：自分のGitHubユーザー名
   - **リポジトリ**：Step 2 で作ったリポジトリ
   - **分岐（ブランチ）**：main
7. ビルドの詳細：
   - **ビルドのプリセット**：Custom
   - **アプリの場所**：`/`
8. 「**確認および作成**」→「**作成**」をクリック
9. 数分待つと自動的にデプロイされます🎉

#### Step 5 ― サイトのURLを確認する

1. 作成された Static Web App のリソースページを開く
2. 「**URL**」の欄に表示されているリンクが、あなたのサイトのアドレスです
3. クリックして開くと、HTMLが表示されます！

> ✅ **これで完了！** 誰でもアクセスできるWebページが公開されました。

---

### ② Azure上でEntra IDアクセス制限をかける方法

① の手順で作ったサイトに、Entra ID ログインを追加する手順です。
**やることは「設定ファイルを1つ追加する」だけ**です。

#### Step 1 ― 設定ファイルを作る

テキストエディタ（メモ帳でOK）で、以下の内容のファイルを作ります。

**ファイル名：`staticwebapp.config.json`**

```json
{
  "routes": [
    {
      "route": "/*",
      "allowedRoles": ["authenticated"]
    }
  ],
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad",
      "statusCode": 302
    }
  }
}
```

> 💡 この設定の意味：
> - `"route": "/*"` → すべてのページに対して
> - `"allowedRoles": ["authenticated"]` → ログインしている人だけアクセスOK
> - `"redirect": "/.auth/login/aad"` → ログインしていない人はEntra IDのログイン画面に飛ばす

#### Step 2 ― GitHubにアップロードする

1. GitHubのリポジトリページを開く
2. 「**Add file**」→「**Upload files**」をクリック
3. Step 1 で作った `staticwebapp.config.json` をドラッグ＆ドロップ
4. 「**Commit changes**」をクリック

#### Step 3 ― 自動でデプロイされるのを待つ

- GitHubにファイルを追加すると、**自動的にAzure側に反映**されます（1〜2分）
- 特に追加の操作は不要です

#### Step 4 ― 動作確認

1. シークレットブラウザ（InPrivate / シークレットモード）でサイトのURLを開く
2. → **Entra ID のログイン画面が表示されれば成功！** 🎉
3. ログインすると、HTMLページが表示されます

> ✅ **これで完了！** Entra ID でログインした人だけがページを見れるようになりました。

---

### 💬 よくある質問

**Q. 認証を外して元に戻したい場合は？**
→ GitHubから `staticwebapp.config.json` を削除するだけでOKです。自動的に全公開に戻ります。

**Q. 特定の人だけに見せたい場合は？**
→ Azure Static Web Apps の「ロールの管理」機能から、招待メールでユーザーを追加できます。Azure Portal → Static Web App → 「ロールの管理」から設定できます。

**Q. HTMLファイル以外（画像やPDFなど）も保護される？**
→ はい。`"route": "/*"` の設定で、サイト内のすべてのファイルが保護されます。

**Q. 費用はかかる？**
→ Azure Static Web Apps の **Free プラン** で認証機能が使えます。無料です。
