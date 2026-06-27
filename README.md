# お笑いライブ 投票システム（Supabase版）

ライブの順位投票をQRコードでサポートするWebアプリです。
GitHub Pagesで公開し、Supabase（無料）でデータを保存します。
RLS（行レベルセキュリティ）を有効化し、パスワード照合はすべてサーバー側の関数で行うため、
ブラウザのコードからお客さんのパスワード一覧が漏れません。

## 機能と仕様の対応
- ② ライブごとに管理者パスワード。認証後のみ順位閲覧（`admin_get_live` / `get_ranking`）
- ③ ライブ作成時に人数入力
- ④ 人数分のお客さんPWを自動発行、未入力では投票不可
- ⑤ QRを各ライブの管理画面に表示・PNG/PDF保存
- ⑥ 存在しないPWは投票画面でアラート（`cast_vote` が `bad_pw` を返す）
- ⑦ 同じPWで再投票すると「投票を修正しますか」→ はいで上書き（`votes` の主キー + upsert）
- ⑧ superadmin画面でPW閲覧・編集・ライブ削除
- ⑨ 香盤を貼り付けるとユニット自動登録、投票画面に反映（`set_units`）
- ⑩ 順位表にユニット名と得票数

## セットアップ（所要15〜20分）

### 1. Supabaseプロジェクトを作る
1. https://supabase.com/ にログイン →「New project」
2. 名前・データベースパスワード（DB管理用、覚えておく）・リージョン（Northeast Asia / Tokyo推奨）を入力して作成
3. 数分でプロビジョニング完了

### 2. テーブルと関数を作る
1. 左メニュー「SQL Editor」→「New query」
2. `schema.sql` の中身をすべて貼り付け
3. **貼り付けた中の `_super_ok` 関数にある `CHANGE_THIS_SUPER_PW` を、自分のsuperadminパスワードに変更**
4. 「Run」を実行（成功すればテーブル3つ・関数群が作成されます）

### 3. APIキーを index.html に貼る
1. 左メニュー「Project Settings > API」
2. `Project URL` と `anon public` キーをコピー
3. `index.html` の `SUPABASE_URL` と `SUPABASE_ANON_KEY` を置き換える
   （anonキーは公開しても問題ありません。RLSと関数で守られています）

### 4. Realtimeを有効化（順位表のリアルタイム更新用）
`schema.sql` の最後で `votes` テーブルをRealtimeに追加済みです。
もし反映されない場合は「Database > Replication」で `votes` がpublicationに含まれているか確認してください。

### 5. GitHubで公開（GitHub Pages）
1. 新しいリポジトリ（Public）を作成
2. `index.html` と `README.md` をアップロード（`schema.sql` は公開不要ですが置いてもOK）
3. Settings > Pages → Source「Deploy from a branch」、Branch `main` / `(root)` → Save
4. 数分後 `https://<ユーザー名>.github.io/<リポジトリ名>/` で公開

## 使い方
1. 公開URL →「管理者画面へ」→ ライブを作成（人数入力）
2. 出てきた管理者パスワードを控える
3. 「香盤・ユニット」で出演者を1行ずつ入力 → 登録
4. 「投票QR」を会場掲示（画像/PDF保存可）
5. 「お客さんPW」を1人1つ配布
6. お客さんはQR→PW入力→投票。「順位表」でリアルタイム集計

## セキュリティについて（この構成で守られていること）
- `lives`（管理者PW・お客さんPWを含む）と `votes` は直接アクセス**全拒否**
- 取得・投票・集計はすべて `SECURITY DEFINER` 関数経由で、関数内でパスワードを照合
- お客さんの投票画面に渡るのは「ライブ名」と「ユニット一覧」だけ（PWは渡らない）

### さらに固めたい場合の発展
- superadminパスワードを関数内ベタ書きでなく、Supabaseの Vault や別テーブル＋ハッシュにする
- 投票関数にレート制限（短時間の連打防止）を入れる
- 管理者ログインを本物の Supabase Auth に置き換える

必要なら追加実装します。
