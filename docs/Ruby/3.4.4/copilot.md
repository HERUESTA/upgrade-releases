# Ruby 3.4.4 コードリーディングノート

- 対象バージョン: 3.4.3 → 3.4.4
- リリース日: 2025年5月14日
- リリース種別: **前倒しリリース**（通常2ヶ月ごとだが、重要な修正のため早期リリース）

---

## 概要

| 分類 | 件数 | 自分のコードへの影響 |
|------|------|-------------------|
| セキュリティ修正（CVE） | 1件 | ⚠️ net-imapを使っている場合は要確認 |
| バグ修正（Ruby本体） | 4件 | Hashのtransform_values!を使っている場合は要確認 |
| YJIT修正 | 2件 | 通常は影響なし |
| CI/インフラ | 7件 | 影響なし |
| コード整理 | 3件 | 影響なし |

---

## ステップ1: リリースノート

- https://www.ruby-lang.org/ja/news/ → 日本語版には3.4.4の記載なし
- https://www.ruby-lang.org/en/news/2025/05/14/ruby-3-4-4-released/ → 英語版リリースノート
- https://github.com/ruby/ruby/releases/tag/v3_4_4 → GitHubリリースページ（チケット一覧あり）
- https://github.com/ruby/ruby/compare/v3_4_3...v3_4_4 → タグ間の全差分（23コミット）

---

## ステップ2: チケット詳細

### ⚠️ 自分のコードに影響する可能性があるもの

#### Bug #21331: transform_values!中のrehashでヒープ解放後アクセス（use-after-free）
- **影響**: `Hash#transform_values!`のブロック内で`rehash`を呼ぶとクラッシュする
- **修正後の動作**: `RuntimeError: rehash during iteration`が発生するようになった（安全に停止）
- **自分のコードへの影響**: `transform_values!`のブロック内で`rehash`を呼んでいる場合はRuntimeErrorになる。ただし通常このような書き方はしない
- https://bugs.ruby-lang.org/issues/21331
- 📝use-after-free: メモリが解放された後にそのメモリにアクセスしてしまうバグ

---

### ℹ️ 環境依存の修正（Windows環境の場合は要確認）

#### Bug #21286: Windows — GCC 15.1.0でビルドが失敗する
- **影響**: Windows + MSYS2環境でRubyやネイティブ拡張gemのビルドが失敗する
- **原因**: GCC 15のC23準拠により、`rb_define_method`等の関数で型チェックが厳しくなった
- **影響範囲**: nokogiri, prism, date, fiddleなど多くのネイティブ拡張gem
- **自分のコードへの影響**: Windowsでネイティブ拡張gemを使っている場合はアップデート推奨
- https://bugs.ruby-lang.org/issues/21286
- 📝MSYS2: Windows上でLinux風の開発環境を使えるようにするツール
- 📝GCC: C言語やC++のコンパイラ。Rubyの本体はC言語で書かれているのでビルドに必要

#### Bug #21327: Windows — clock_gettime変更後のビルド問題
- **影響**: Windows + MSYS2環境でRubyのビルドが失敗する
- **原因**: winpthreadsが`clock_gettime`と`clock_getres`をインライン関数として提供するようになり、Rubyの`win32.c`内の同名関数と再定義エラーが発生
- **自分のコードへの影響**: Bug #21286と同様、Windowsでのビルドに影響
- https://bugs.ruby-lang.org/issues/21327

---

### ℹ️ 通常は影響なしの修正

#### PR #13331 / PR #13282: YJIT — getlocal/setlocalの最適化でブロック分割
- **影響**: YJITのローカル変数最適化の内部修正
- **自分のコードへの影響**: なし（YJIT内部の改善）
- https://github.com/ruby/ruby/pull/13331
- 📝YJIT: Ruby 3.1から導入されたJITコンパイラで、Rubyの実行速度を早くする仕組み

#### Bug #21257: YJIT — OOM時に無限ループが発生する
- **影響**: YJITがメモリ不足の時、特定条件下で無限ループが生成される
- **発生条件**: TracePoint有効 + 定数キャッシュ無効化 + OOMの3条件が同時に揃った時
- **自分のコードへの影響**: 非常にまれなエッジケース。通常は影響なし
- https://bugs.ruby-lang.org/issues/21257

#### Bug #21289: ELF環境でのCレベルバックトレース修正
- **影響**: Linux環境でクラッシュ時のCレベルバックトレース情報が取得できなくなっていた
- **原因**: 以前のMach-O向け修正がELF環境に悪影響を与えていた
- **自分のコードへの影響**: なし（デバッグ情報の改善）
- https://bugs.ruby-lang.org/issues/21289
- 📝Mach-O: macOS・iOSなどApple系OSの実行ファイルフォーマット
- 📝ELF: LinuxなどUnix系OSの実行ファイルフォーマット

---

## ステップ3: リリースノートに載っていないコミット

### ⚠️ セキュリティ修正（リリースノートに未記載）

#### Bump net-imap to v0.5.8（CVE-2025-43857）
- **影響**: net-imap gemにDoS（サービス拒否）脆弱性がある
- **自分のコードへの影響**: net-imapを使っている場合はアップデート必須
- https://github.com/advisories/GHSA-j3g3-5qv5-52mj
- 📝net-imap: IMAP（メールサーバーからメールを読み取るプロトコル）クライアントを提供するgem

### ℹ️ YJIT関連

#### YJIT: End the block after OPTIMIZE_METHOD_TYPE_CALL
- YJIT関連の動作修正
- https://github.com/ruby/ruby/commit/7afa08ce8d245c7d756ddaeb78daddb61fd0e271

### ℹ️ CI/インフラ（影響なし）

| コミット | 内容 |
|---------|------|
| [4fc785b](https://github.com/ruby/ruby/commit/4fc785b91bec95cf4bb7a604fcb302db4deece90) | ubuntu-20.04 → ubuntu-22.04に置き換え |
| [c11836e](https://github.com/ruby/ruby/commit/c11836eceace1339d8b4388ed9625acac837241d) | windows-2019がEOLのため2022/2025に変更 |
| [321fb43](https://github.com/ruby/ruby/commit/321fb434ff74bac551eb0fa548bb55ccd895e2ba) | matrix.vs → matrix.osにリネーム（📝vs: Visual C++バージョン、os: CIランナーのOSバージョン） |
| [6e95400](https://github.com/ruby/ruby/commit/6e95400497b8cc6663fb417a53bf2d33b265bfb1) | ubuntu-20.04のretired対応 |
| [b04f399](https://github.com/ruby/ruby/commit/b04f3995ea520d723de5507203abc63537ad8fd3) | GitHub ActionsのcommitIDを更新 |
| [9e22d6f](https://github.com/ruby/ruby/commit/9e22d6fb1e546b4dde2a518cfc808e5d89044cff) | test-bundled-gemsでwindows-2022イメージを使用 |
| [1a0a169](https://github.com/ruby/ruby/commit/1a0a1690cdcda415879e28302d888a1dc1e3d74a) | 正しいバージョンのrakeを実行するよう修正 |

### ℹ️ コード整理（影響なし）

| コミット | 内容 |
|---------|------|
| [22f2047](https://github.com/ruby/ruby/commit/22f2047a3d8b02018269b49dbcd23b8101314009) | 未使用変数retvalの代入を削除 |
| [3557739](https://github.com/ruby/ruby/commit/3557739db95833454de34a4169212633dec8c76d) | rubocop: ブレース内スペースのフォーマット修正 |
| [4aac150](https://github.com/ruby/ruby/commit/4aac15063094d30c3ed0c3639ffbf54a595731a5) | 不安定なRactorテストをスキップ |

### ℹ️ ビルド環境（影響なし）

| コミット | 内容 |
|---------|------|
| Supply LIBCLANG_PATH | yjit-bindgen用にclang-14のパスを指定 |
| Use clang-14 | bindgenのデフォルトに合わせてclang-14を使用 |

---

## ステップ4・5: コード深掘り — Bug #21331

### テストコード（test/ruby/test_hash.rb）

今回追加されたテスト：

```ruby
x = (1..1337).to_h {|k| [k, k]}
assert_raise_with_message(RuntimeError, /rehash during iteration/) do
  x.transform_values! {|v|
    x.rehash if v == 1337
    v * 2
  }
end
```

- 1337個の要素を持つHashを作成（📝要素数が多いとst_tableという内部構造になり、rehashでメモリの解放・再確保が起きる）
- `transform_values!`のイテレーション中に`rehash`を呼ぶ
- **期待動作**: RuntimeError（"rehash during iteration"）が発生すること
- **修正前**: クラッシュ（use-after-free）
- **修正後**: RuntimeErrorで安全に停止

### 本体コード（hash.c）

修正のポイント：

```c
// 新しく追加された関数
static void
transform_values(VALUE hash)
{
    hash_iter_lev_inc(hash);    // 「今このHashはイテレーション中」というフラグを立てる
    rb_ensure(transform_values_call, hash,
              hash_foreach_ensure, hash);  // 正常終了でも例外でも必ずフラグを下げる
}
```

- **修正前**: `transform_values!`はイテレーション中であることをHashに記録していなかった → ブロック内で`rehash`を呼んでも止められず、内部テーブルが解放されてuse-after-freeが発生
- **修正後**: `hash_iter_lev_inc`でイテレーションレベルを上げ、`rehash`が呼ばれた時に「今イテレーション中だからダメ」と検知してRuntimeErrorを発生させる

また、`rb_hash_stlike_foreach`と`rb_hash_stlike_foreach_with_replace`に「This does not manage iteration level」というコメントが追加された。これは「この関数自体はイテレーションレベルを管理しないので、呼び出し側が管理する責任がある」という設計上の注意書き。

### Bug #21331の経緯（3コミット）

1. `862480a` — 最初の修正をバックポート（Prohibit modification during stlike loop）
2. `fbcb327` — **Revert**（最初の修正に問題があったため取り消し）
3. `a21b88a` — 別のアプローチで再修正（Prohibit hash modification during stlike loop）

最終的に採用されたのは3番目のコミット。