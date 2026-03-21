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
    - [バグ #21289: USE_ELF の C レベルバックトレースを修正 - Ruby - Ruby 問題追跡システム](https://bugs.ruby-lang.org/issues/21289)
- https://github.com/ruby/ruby/compare/v3_4_3...v3_4_4
  - 変更ログ
# 2. リリースノートのリンク先を読む
## YJIT: 最適化された getlocal/setlocal のブロックを分割する
- YJITのgetlocal/setlocal周りの最適化に修正が入った

## バグ #21257: YJIT は OOM 時に無限ループを生成する可能性がある - Ruby - Ruby 問題追跡システム
- メモリ不足のときに YJIT が無限ループ (同じアドレスへのジャンプ) を生成する可能性があるエッジケースを発見
  - 📝YJIT:Ruby 3.1から導入されたJITコンパイラで、Rubyの実行速度を早くする仕組み

## バグ #21286: Windows - MSYS2 が GCC 15.1.0 にアップデートされたばかりで、ビルドが失敗する - Ruby - Ruby 問題追跡システム
- MSYS2 が GCC 15.1.0 にアップデートされたばかりで、ruby-loco ビルドのコンパイルに失敗
  - 📝MSYS2：Windows上でLinux風の開発環境を使えるようにするツール（最近では、Linux/Unix向けに作られていて、Windowsだとそのまま動かないケースがあるから）
  - 📝GCC：C言語やC++のコンパイラ（rubyの本体はC言語で書かれているので、ビルドするために必要）
  - 📝ruby-loco：MSP-Gregさんが管理している、Windows向けにRubyをビルドするプロジェクト

## バグ #21327: clock_gettime の変更後、Windows ビルドが壊れているようです - Ruby - Ruby 問題追跡システム
- winpthreadsがclock_gettimeとclock_getresをインライン関数として提供するようになったことで、Rubyのwin32.c内の同名関数と再定義エラーが発生

## バグ #21331: transform_values 中の再ハッシュによりヒープの解放後使用が発生する - Ruby - Ruby 問題追跡システム
- `transform_values`中に再ハッシュによってヒープ開放後のアクセスが発生していることを発見
  - 📝ハッシュ：名前（キー）と値（データ）をペアにして管理するデータ構造

## バグ #21289: `USE_ELF`のCレベルバックトレースを修正 - Ruby - Ruby 問題追跡システム
- Ruby 3.4にアップグレードした際、クラッシュレポートに有用なCレベルのバックトレース情報が表示されなくなった
  - 以前の修正でMach-O向けに変更した箇所がELF環境に悪影響を与えていた
  - `addr2line.c`での実行ファイルのベースアドレス計算が壊れ、デバッグ情報が読めなくなっていた
    - 📝Mach-O（マッハオー）：macOS・iOS・iPadOSなどApple系OSで使われる実行ファイルのフォーマット
    - 📝ELF：LinuxをはじめとするUnix系OSで広く使われている実行ファイルのフォーマット

# 3. タグ間のコミットログを読む（ステップ2で出したものは省く）
## Remove unused retval assignments
https://github.com/ruby/ruby/commit/22f2047a3d8b02018269b49dbcd23b8101314009#diff-2894a90fd17590e4f7afd08dfd8505526d5283e09fea645cc7a7d4873936bdff
- retvalが削除された（未使用変数だったため）

## Skip an unstable Ractor test
https://github.com/ruby/ruby/commit/4aac15063094d30c3ed0c3639ffbf54a595731a5
- 不安定なテストをスキップするようにした

## YJIT: End the block after OPTIMIZE_METHOD_TYPE_CALL
https://github.com/ruby/ruby/commit/7afa08ce8d245c7d756ddaeb78daddb61fd0e271
- YJITのテスト修正

## [rubygems/rubygems] Fixed rubocop issue: Layout/SpaceInsideBlockBraces
https://github.com/ruby/ruby/commit/3557739db95833454de34a4169212633dec8c76d
- コードのフォーマット修正

## Run the proper version of rake
https://github.com/ruby/ruby/commit/1a0a1690cdcda415879e28302d888a1dc1e3d74a
- バージョン固定のため、書き方を修正
## Use windows-2022 image for test-bundled-gems
https://github.com/ruby/ruby/commit/9e22d6fb1e546b4dde2a518cfc808e5d89044cff
- windowsOSのバージョン変更

## Bump up the latest version of actions
https://github.com/ruby/ruby/commit/b04f3995ea520d723de5507203abc63537ad8fd3
- それぞれのgithubリポジトリのcommitIDを変更

## ubuntu-20.04 is retired
https://github.com/ruby/ruby/commit/6e95400497b8cc6663fb417a53bf2d33b265bfb1
- ubuntuバージョンアップ

## Rename matrix.vs to matrix.os. It's not Visual Studio version now
https://github.com/ruby/ruby/commit/321fb434ff74bac551eb0fa548bb55ccd895e2ba
- `vs`から`os`に変更（vsとosでバージョンの差異が出るようになったため）
  - 📝vs：Visual C++ のコンパイラバージョン（2019）
  - 📝os：CI ランナーの`OS`バージョン（Windows Server 2022/2025）

## Use windows-2022 and windows-2025 because windows-2019 is EOL at June 2025
https://github.com/ruby/ruby/commit/c11836eceace1339d8b4388ed9625acac837241d
- EOL終了に伴い、windowsのバージョンアップデート

## Supply LIBCLANG_PATH for clang-14 for yjit-bindgen Or else it gets confused from all the different versions of LLVM in the
- バージョン固定のため、パスを指定

## Use clang-14 to match the libclang version bindgen finds by default
- バージョンを一致させるため、clang-14を使用

## sd ubuntu-20.04 ubuntu-22.04 .github/workflows/*
https://github.com/ruby/ruby/commit/4fc785b91bec95cf4bb7a604fcb302db4deece90
- ubuntuのバージョンアップ

## Bump net-imap to v0.5.8 for Ruby 3.4
- net-imapを0.5.8にアップデート
  - 📝net-imap：IMAP（Internet Message Access Protocol）クライアントを提供する gem
  - 📝IMAP：メールサーバーからメールを読み取るためのプロトコル
    - net-imapを用いることで、rubyのプログラムでメールの取得、検索、フォルダ操作などができる
- ⚠️DOSの脆弱性のため、アップデート
  - https://github.com/advisories/GHSA-j3g3-5qv5-52mj
    - net-imap ruby​​gem はメモリ枯渇による DoS 攻撃の脆弱性がある

# 4. テストコードのdiffを読む
## Bug #21331 — transform_values!のuse-after-free
`test_hash.md`参照


# 5. 本体のdiffを読む（テストで期待される動作を理解した上で）

# 参考箇所


> ステップ1: リリースノートを読む
ruby3.4.4に関する記載はなかったです🥲

> ステップ2: リリースノートのリンクを辿る