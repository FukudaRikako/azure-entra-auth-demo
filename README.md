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
