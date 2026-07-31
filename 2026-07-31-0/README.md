---
title: Tailscale から Cloudflare Zero Trust へ移行する遊び
date: 2026-07-31
tags:
  - networking
  - tailscale
  - cloudflare
---
## TL;DR

全端末に Cloudflare One Client (or warp-cli) を入れて Tailscale の代わりにした。Tailscale の使用感を再現するために、ホスト名を解決するDNSレコードの管理を Cloudflare Workers で自動化し、ついでに Tailscale ライクなダッシュボード UI を作ったりして遊んだ。

## Cloudflare Zero Trust の導入

公式ドキュメントとか読みながらやる。

split tunneling の設定プロファイルはデフォルトで excluded routing だが、Parsec というリモートデスクトップアプリと干渉した。あくまで Tailscale の代替と考えれば、すべてのトラフィックを Zero Trust へ流す必要はないので、included routing へ変更、CIDR は公式ドキュメント通り `100.96.0.0/12` にした。 

## MagicDNS ライクなホスト名解決

Tailscale を導入すると `ssh <hostname>` といった DNS 解決がされるので便利。

Zero Trust 内部の DNS を設定できれば良かったのだが、どうやら個人プランではできなさそうだったので、 `<hostname>.internal.ojii3.dev` というパブリックレコードを作成して、search domains 的な奴(詳しくは知らない)で `internal.ojii3.dev` を省略できるようにしてしまえとなった。グローバル IP じゃないし DNS レコードからバレても困らない(はず、たぶん)。

Cloudflare Workers の cron job で 5分ごとに

1. Zero Trust に登録されているデバイス情報を取得
2. デバイス情報からホスト名と Zero Trust のIPアドレスを取得して `<hostname>.internal.ojii3.dev` に対して A レコードを作成
3. デバイスが削除されていればDNSレコードも削除

などを API を叩けばいい。実装は確か Qwen 3.7 Plus 君がやってくれた。

## ダッシュボード

Cloudflare Zero Trust のダッシュボードでは、Tailscale の様にデバイスのステータス、IP、OS などを一目で見られる画面が存在しない。

先ほどと同じ Worker に載せるよう AI に作らせたらペライチ index.html + vanilla js で作ってきたので、vite + svelte に手動で移行し、UI は Skeleton で作らせ直した。

svelte を選んだ理由は特にない。AI に Elysia.js に合うUIライブラリを聞いたら Svelte などと適当な答えが返ってきた。

## Cloudflare Access

サイトの認証に Cloudflare Access を使った。

初めて単にサイトを保護する用途以外で使ってみた。ちょこっと調べたら、Request に含まれる jwt を検証すればいいらしかった。公式ドキュメントに `jose` を用いたサンプルが載っていたのでそのまま使った。

Clouflare Access でページを保護する設定は、ダッシュボードで手動でやるかわりに terraform でやることにした。せっかくなので tfstate は AWS S3 互換の Cloudflare R2 を使うことにした。

## デプロイに関して

いつもの GitHub Actions を使う。Workers のデプロイに使用するトークンと Workers が DNS や Zero Trust にアクセスするためのトークンと、etc... と必要なシークレットが多くて大変だった。これどうにかならないもんですかね。

## おわりに

AIのおかげで新しい技術スタックに手をだしても短期間で完成した(代わりに知識は何も得られていない)。

Tailscale を使っているというだけで Cloidflare で遊べる幅が狭まっていたので、移行できてうれしーかも
