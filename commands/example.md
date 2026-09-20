---
description: サンプルのスラッシュコマンド。/example で呼び出せます。
argument-hint: "[任意の引数]"
---

これはカスタムスラッシュコマンドの雛形です。`/example` で実行されると、
このファイルの本文がプロンプトとして Claude に渡されます。

引数は $ARGUMENTS で受け取れます（例: `/example foo bar` → "foo bar"）。

## やってほしいこと
- ここに定型の指示を書く
- Skill より手軽。単発のプロンプトを使い回したいときに向く

引数: $ARGUMENTS
