# 1. リリースノート（概要の把握）
- 予定より前倒しでリリースされた（通常2ヶ月ごとだが、重要な修正のため早めに出した）
- 主な修正: YJITのローカル変数に関するバグ修正、Windows環境でのGCC 15によるビルド問題の修正
- セキュリティ修正（CVE）はなし
- パッチの性質: バグ修正中心

- https://www.ruby-lang.org/ja/news/
  - newsでは3.4.4の内容は書かれておらず
- https://www.ruby-lang.org/en/news/2025/05/14/ruby-3-4-4-released/
  - ruby3.4.4のリリースノート
- https://github.com/ruby/ruby/releases/tag/v3_4_4
  - githubのリリースノート
    - [YJIT: 最適化された getlocal/setlocal のブロックを分割する (k0kubun 作) · Pull Request #13331](https://github.com/ruby/ruby/pull/13331/changes)
    - [バグ #21257: YJIT は OOM 時に無限ループを生成する可能性がある - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21257)
    - [バグ #21286: Windows - MSYS2 が GCC 15.1.0 にアップデートされたばかりで、ビルドが失敗する - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21286)
    - [バグ #21327: clock_gettime の変更後、Windows ビルドが壊れているようです - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21327)
    - [バグ #21331: transform_values 中の再ハッシュによりヒープの解放後使用が発生する - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21331)
    - [バグ #21289: USE_ELF の C レベルバックトレースを修正 - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21331)
- https://github.com/ruby/ruby/compare/v3_4_3...v3_4_4
  - 変更ログ
# 2. リリースノートのリンク先を読む
## YJIT: 最適化された getlocal/setlocal のブロックを分割する (k0kubun 作) · Pull Request #13331
- YJITのgetlocal/setlocal周りの最適化に修正が入った

## バグ #21257: YJIT は OOM 時に無限ループを生成する可能性がある - Ruby - Ruby 問題追跡システム
- メモリ不足のときに YJIT が無限ループ (同じアドレスへのジャンプ) を生成する可能性があるエッジケースを発見

# 3. タグ間のコミットログを読む

# 4. テストコードのdiffを読む

# 5. 本体のdiffを読む（テストで期待される動作を理解した上で）

# 参考箇所


> ステップ1: リリースノートを読む
ruby3.4.4に関する記載はなかったです🥲

> ステップ2: リリースノートのリンクを辿る