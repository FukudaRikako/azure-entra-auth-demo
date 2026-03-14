# Azure Static Web Apps × Entra ID 認証デモ

## このデモで伝えたいこと

| | GitHub Pages（従来） | Azure Static Web Apps |
|---|---|---|
| ホスティング | ✅ 無料で簡単 | ✅ 無料で簡単 |
| アクセス制御 | ❌ **全世界に公開** | ✅ **Entra ID で認証された人だけ** |

> **ポイント：** HTMLファイルを置くだけなら GitHub でも Azure でも同じ。
> でも Azure なら **たった数行の設定ファイルを追加するだけ** で、Entra ID による認証・認可が実現できます。

---

## デモの流れ

### ① GitHub 上では全公開

GitHub リポジトリにHTMLファイルをアップロードすると、リポジトリがPublicなら**誰でもソースコードが見れる**状態になります。

📁 リポジトリ構成:
```
├── index.html                  ← プレゼン用のHTMLファイル
├── staticwebapp.config.json    ← 認証設定（これがキモ！）
└── og-image.png                ← OGP画像
```

### ② Azure Static Web Apps + Entra ID で保護

Azure Static Web Apps にデプロイすると、`staticwebapp.config.json` の設定により：

- **認証されていないユーザー** → Entra ID のログイン画面にリダイレクト
- **認証済みユーザー** → ページが表示される

```
ユーザー → サイトにアクセス
              ↓
        認証済み？ ──No──→ Entra ID ログイン画面
              ↓ Yes                    ↓
        ページ表示 ←── ログイン成功 ──┘
```

---

## 認証設定の中身（staticwebapp.config.json）

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

**たったこれだけ！** やっていることは：

1. **全ページ（`/*`）へのアクセスに `authenticated` ロールを要求** — ログイン必須
2. **未認証（401）の場合は Entra ID ログインにリダイレクト** — 自動でログイン画面へ

---

## なぜ Azure Static Web Apps？

| メリット | 説明 |
|---|---|
| **ノーコードで認証追加** | HTMLファイルの変更は不要。設定ファイル1つで完結 |
| **Entra ID 統合** | 社内のID基盤をそのまま利用。追加のユーザー管理不要 |
| **GitHub 連携** | pushするだけで自動デプロイ（GitHub Actions） |
| **無料プラン** | Free tier で認証機能が使える |
| **グローバル配信** | CDN による高速配信 |

---

## デモ用URL

- 🔗 **サイト:** https://gentle-pebble-053949610.1.azurestaticapps.net
- 📂 **GitHub:** https://github.com/FukudaRikako/doc

### 試してみてください

1. **シークレットブラウザ**で上記URLを開く → ログイン画面が表示される
2. **Entra ID アカウント**でログイン → ページが表示される
3. **ログインしていない状態**ではコンテンツにアクセスできない

---

## 構成図

```
┌─────────────┐     push      ┌─────────────────┐    自動デプロイ    ┌──────────────────────┐
│   VS Code   │ ───────────→  │  GitHub (リポ)   │ ──────────────→  │  Azure Static Web Apps │
│  HTML編集    │               │  ソースコード管理  │   GitHub Actions  │  + Entra ID 認証       │
└─────────────┘               └─────────────────┘                   └──────────────────────┘
                                                                              ↑
                                                                     Entra ID で保護された
                                                                     ユーザーだけがアクセス可能
```
