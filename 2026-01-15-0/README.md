---
title: codex-cli の symlink のつらみを nix で乗り越える
date: 2026-01-15
tags:
  - dev
  - nix
---
## Codex CLI の

Codex CLI の設定は、`~/.codex/config.toml` に書きます。

また、一般的に、こういったファイルを dotfiles で管理するには `~/.codex/config.toml` へのシンボリックリンクを張ります。(Nix & home-manager でも同様)

しかし、Codex CLI はプロジェクト毎の設定などを全てこの `~/.codex/config.toml` に書きこんでくるのです。(Nix だと read-only になるのですが、それでも上書きしてくる)

## リンクを張らないという選択

シンボリックリンクを張るのではなく、コピーして配置する方法をとります。

`home-manager` であれば、 `home.activation` に任意のシェルスクリプトを書くことで実現できます。

```nix title="modules/dev/ai/codex/default.nix"
{ pkgs, lib, ... }
let
    codexConfig = ./config.toml;
in
{
    home.activation.codexSeedMerge = lib.hm.dag.entryAfter [ "writeBoundary" ] ''
        cp '${codexConfig}' '${config.home.homeDirectory}/.codex/config.toml'
    '';
}
```

上記のコードで、dotfiles 内の同じ階層にある `config.toml` を `~/.codex/config.toml` にコピーできます。

関数型言語なのに副作用たっぷりで不思議な感じがしますね。

## コピーだけでは不十分

ここで満足してはいけません。ここまでの設定では、`home-manger switch` で更新するたびに `~/.codex/config.toml` が上書きされてしまいます。

Codex CLI が自動で書きこんだプロジェクト毎の設定、例えば以下のような項目が `~/.codex/config.toml` から消されてしまいます。

```toml
[projects."/Users/ojii3/src/github.com/OJII3/dotfiles"]
trust_level = "trusted"
```

そうすると、このディレクトリで Codex CLI を使うたびに再度ポップアップが表示されて面倒です。

## tomlkit でマージする

Python の `tomlkit` ライブラリを使って、`config.toml` と `~/.codex/config.toml` をマージするスクリプトを書きます。

```python title="modules/dev/ai/codex/merge_codex_config.py"
# ここにコードを貼ってもいいのですが、どうせ AI に書かせただけなので、省略....
```

ぼちぼち賢いAIに書かせれば問題ないでしょう。

この Python コードを `home.activation` で実行すれば良く、最終的に以下のような Nix モジュールになりました。


```nix title="modules/dev/ai/codex/default.nix"
{
  config,
  lib,
  pkgs,
  ...
}:
let
  cfg = config.dot.home.dev.ai;
  seedToml = ./config.toml;
  pythonWithTomlkit = pkgs.python3.withPackages (ps: [ ps.tomlkit ]);
  codexConfigPath = "${config.home.homeDirectory}/.codex/config.toml";
in
{
  config = lib.mkIf cfg.codex.enable {
    home.packages = with pkgs; [
      bun
      pythonWithTomlkit
    ];
    programs.zsh = {
      shellAliases = {
        codex = "bun x @openai/codex@latest";
      };
    };

    home.file.".codex/AGENTS.md".source = ./AGENTS.md;
    home.file.".codex/scripts" = {
      source = ./scripts;
      recursive = true;
    };

    home.activation.codexSeedMerge = lib.hm.dag.entryAfter [ "writeBoundary" ] ''
      # seed 優先でマージ
      ${pythonWithTomlkit}/bin/python ${./merge_codex_config.py} \
        --seed '${seedToml}' \
        --config '${codexConfigPath}'
    '';
  };
}
```

たとえば項目の削除などが反映されないので、完全に快適とはいきませんが、まあまあ実用的にはなりました。

## まとめ

Claude Code の方が良いんじゃない (weeekly limit中...)
