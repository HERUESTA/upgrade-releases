# Ruby 3.4.5 コードリーディングノート

- 対象バージョン: 3.4.4 → 3.4.5
- リリース日: 2025年7月15日
- リリース種別: **通常リリース**（バグ修正とGCC 15サポートが中心）

---

## 概要

| 分類 | 件数 | 自分のコードへの影響 |
|------|------|-------------------|
| セキュリティ修正（CVE） | 0件 | なし |
| バグ修正（Ruby本体） | 18件 | 影響なし（詳細は各チケット参照） |
| YJIT修正 | 0件 | なし |
| CI/インフラ | 20件（ステップ2: 5件 + ステップ3: 15件） | なし |
| コード整理 | 8件（ステップ3） | なし |

---

## ステップ1: リリースノート

- https://www.ruby-lang.org/en/news/2025/07/15/ruby-3-4-5-released/ → 英語版リリースノート（「バグ修正とGCC 15サポートを含む定期アップデート」とだけ記載。詳細はGitHubへ）
- https://github.com/ruby/ruby/releases/tag/v3_4_5 → GitHubリリースページ（チケット一覧あり、26件）
- https://github.com/ruby/ruby/compare/v3_4_4...v3_4_5 → タグ間の全差分

---

## ステップ2: チケット詳細

### ⚠️ 自分のコードに影響する可能性があるもの

なし

---

### ℹ️ 環境依存の修正

#### PR #13472: Sync lockfile from rubygems/rubygems
- **影響**: rakeリリースのたびにBundler CIが動作しなかったバグの修正
- **原因**: Rubyの安定ブランチがロックファイル（Gemfile.lock）を使用しないため
- **自分のコードへの影響**: なし（CI環境の問題）
- https://github.com/ruby/ruby/pull/13472

#### Bug #21255: Can't build Ruby with Windows SDK 10.0.26100
- **影響**: Windows SDK 10.0.26100でRubyをインストールできない
- **修正**: SDKバージョンのダウングレードで対応
- **自分のコードへの影響**: Windows環境のみ
- https://bugs.ruby-lang.org/issues/21255

#### PR #13757: Backport GH-13617 for s390x
- **影響**: s390xアーキテクチャで失敗していたテストをスキップ
- **自分のコードへの影響**: なし（s390x環境のみ）
- https://github.com/ruby/ruby/pull/13757
- 📝s390x: IBMメインフレーム向けアーキテクチャ。ビッグエンディアンかつ独自のメモリアライメント要件を持つ

#### PR #13818: Bump up resolv-0.6.2 for Ruby 3.4
- **影響**: resolvライブラリをバージョンアップ（Windows向けの機能変更が含まれる）
- **自分のコードへの影響**: なし（Windows環境のみ）
- https://github.com/ruby/ruby/pull/13818
- 📝resolv: Rubyの標準ライブラリに含まれるDNS名前解決のためのライブラリ

#### Bug #21500: Backport gcc 15 support
- **影響**: GCC 15でRubyをビルドできるようにサポート追加
- **自分のコードへの影響**: GCC 15を使う場合は恩恵あり
- https://bugs.ruby-lang.org/issues/21500

---

### ℹ️ 通常は影響なしの修正

#### Bug #21340: Bump autoconf version to properly handle C23 bool/stdbool defines
- **影響**: C23のbool/stdbool定義を適切に処理するためにautoconfバージョンを更新
- **自分のコードへの影響**: なし（ビルドシステムの内部修正）
- https://bugs.ruby-lang.org/issues/21340
- 📝C23: C言語の最新の国際標準規格（ISO/IEC 9899:2024）

#### Bug #21438: use-after-free when resizing exivars
- **影響**: 汎用インスタンス変数（exivars）をリサイズする際にメモリ解放後アクセスが発生
- **自分のコードへの影響**: なし（Cレベルの内部修正）
- https://bugs.ruby-lang.org/issues/21438

#### PR #12661: Ensure that memory is not freed before calling free_fast_fallback_getaddrinfo_*
- **影響**: getaddrinfo関連の内部メモリ管理バグの修正
- **自分のコードへの影響**: なし
- https://github.com/ruby/ruby/pull/12661

#### PR #13231: Fix heap-use-after-free in free_fast_fallback_getaddrinfo_entry
- **影響**: getaddrinfo関連のheap-use-after-freeの修正
- **自分のコードへの影響**: なし
- https://github.com/ruby/ruby/pull/13231

#### Bug #21441: SEGV during thread cleanup if profiler calls thread_profiles_frames at wrong time
- **影響**: プロファイラがスレッドクリーンアップ中の不正なタイミングで`thread_profiles_frames`を呼ぶとSEGVが発生
- **自分のコードへの影響**: なし（プロファイラ使用時のエッジケース）
- https://bugs.ruby-lang.org/issues/21441
- 📝SEGV: アクセスを許可されていないメモリ領域に読み書きしようとしたときにOSが発するシグナル

#### Bug #21197: Prism does not accept newline after defined? keyword
- **影響**: `defined?`キーワードの後に改行があるとPrismパーサーが受け付けなかった
- **修正後**: 改行が入っていても正しくパースされるように修正
- **自分のコードへの影響**: なし（通常このような書き方はしない）
- https://bugs.ruby-lang.org/issues/21197

#### Bug #21333: heap-use-after-free caused by rehash during update
- **影響**: `to_h`でハッシュ変換後に`rehash`すると、ヒープメモリの解放後アクセスが発生
- **自分のコードへの影響**: なし（3.4.4のBug #21331と同系統の修正。Hash操作中のrehashによるuse-after-free）
- https://bugs.ruby-lang.org/issues/21333

#### Bug #21357: Crash in Hash#merge! with ruby-dev in rubocop-rspec test suite
- **影響**: `Hash#merge!`等のメソッドでクラッシュ
- **自分のコードへの影響**: なし（Bug #21333と関連するHash内部の修正）
- https://bugs.ruby-lang.org/issues/21357

#### Bug #21383: Prism leaks memory with invalid yield
- **影響**: 無効な`yield`によるPrismパーサーのメモリリーク
- **自分のコードへの影響**: なし（不正なコードをパースした場合のみ）
- https://bugs.ruby-lang.org/issues/21383

#### Bug #21394: Memory leak in Prism's RubyVM::InstructionSequence.new
- **影響**: `RubyVM::InstructionSequence.new`でメモリリーク
- **自分のコードへの影響**: なし（ISeqを直接操作しない限り影響なし）
- https://bugs.ruby-lang.org/issues/21394

#### Bug #21099: TestGc#test_gc_stress_at_startup assertion failure
- **影響**: 起動時にGCが実行されると`newobj_cache`がクリアされずクラッシュ
- **原因**: Ractorリストが初期化されていない状態でGCが走るケース
- **自分のコードへの影響**: なし（起動時のエッジケース）
- https://bugs.ruby-lang.org/issues/21099

#### Bug #21395: debug.gemのrescue/ensureフレームがバックトレースから除外される
- **影響**: debug.gem使用時に`rescue`と`ensure`のフレームがバックトレースに表示されなくなっていた
- **自分のコードへの影響**: debug.gemを使用している場合、デバッグ体験が改善される
- https://bugs.ruby-lang.org/issues/21395

#### Bug #21439: Crash with PM_SPLAT_NODE compiler error (Prism)
- **影響**: forループの左側にスプラット(`*`)を使用するとクラッシュ
- **自分のコードへの影響**: なし（`for *x in ...`のような書き方は通常しない）
- https://bugs.ruby-lang.org/issues/21439

#### Bug #21354: Symbol#to_proc is not ractor safe
- **影響**: `Symbol#to_proc`が複数のRactorから同時に使うと安全でなかった
- **自分のコードへの影響**: なし（Ractorを使用していなければ影響なし）
- https://bugs.ruby-lang.org/issues/21354

#### Bug #20009: Marshal.load raises exception when load dumped class include non-ASCII
- **影響**: クラス名に非ASCII文字（日本語など）を使うと、`Marshal.dump` → `Marshal.load`で例外が発生
- **原因**: Marshalがクラス名をバイト列として保存する際にエンコーディング情報を含めていなかった
- **修正**: デフォルトのエンコーディングをUTF-8と想定するように変更
- **自分のコードへの影響**: なし（日本語クラス名を使用していない場合は影響なし）
- https://bugs.ruby-lang.org/issues/20009

#### Bug #21380: Use-After-Free in String#split with In-Block String Modification
- **影響**: `split`のブロック内で元の文字列を変更すると、use-after-freeが発生しクラッシュ
- **修正後**: `RuntimeError`で安全に停止するようになった
- **自分のコードへの影響**: なし（`split`のブロック内で元の文字列を書き換える操作は通常しない）
- https://bugs.ruby-lang.org/issues/21380

#### Bug #21447: Fix handling of PM_CONSTANT_PATH_NODE in keyword arguments with ARGS_SPLAT
- **影響**: Prismコンパイラの最適化不可能なケースの修正
- **自分のコードへの影響**: なし
- https://bugs.ruby-lang.org/issues/21447

#### Bug #21448: Random.urandom may fail to fall back to reading /dev/urandom on Linux < 3.17
- **影響**: Linux 3.17未満で`Random.urandom`が`/dev/urandom`にフォールバックできない
- **自分のコードへの影響**: なし（Linux 3.17以上であれば影響なし）
- https://bugs.ruby-lang.org/issues/21448

#### Bug #21440: Cannot create instances of frozen Data subclasses
- **影響**: freezeされたDataサブクラスのインスタンスを作成できない
- **原因**: Ruby 3.2で追加された`Data`クラス関連のバグ
- **自分のコードへの影響**: なし（`Data`クラスをfreezeして使っている場合は恩恵あり）
- https://bugs.ruby-lang.org/issues/21440

#### Bug #21437: Date#hash may return different values for equal dates with large years
- **影響**: 年数が大きい場合に`Date#hash`が同じ日付でも異なる値を返すことがある
- **自分のコードへの影響**: なし（通常の年数範囲では影響なし）
- https://bugs.ruby-lang.org/issues/21437

#### Bug #21497: building issue when using gcc15, because C23 is default
- **影響**: GCC 15ではC23がデフォルトになったため、Rubyのビルドに問題が発生
- **自分のコードへの影響**: なし（Bug #21500と合わせてGCC 15対応）
- https://bugs.ruby-lang.org/issues/21497

---

## ステップ3: リリースノートに載っていないコミット

### ⚠️ セキュリティ修正

なし（今回はCVE付きの修正なし。3.4.4ではnet-imapのDoS脆弱性があったが、今回はなし）

### ℹ️ CI/インフラ（影響なし）

| コミット | 内容 |
|---------|------|
| [c104fc4](https://github.com/ruby/ruby/commit/c104fc41b0e8ce80f4178c3b384d1405570714b2) | CodeQLのTRAPキャッシュを無効化（CI不安定化の対策） |
| windows-2025 D drive | windows-2025ランナー環境からDドライブを削除 |
| [8125827](https://github.com/ruby/ruby/commit/8125827578ed5ce3487f9ee08a4adc072f74a234) | RubyGems 3.6.8 + Bundler 2.6.8にアップデート |
| [616771e](https://github.com/ruby/ruby/commit/616771e34e651d2a09ab3dad77d826c2100512e8) | RubyGems 3.6.9 + Bundler 2.6.9にアップデート |
| [03eb777](https://github.com/ruby/ruby/commit/03eb777c69d64aa4941891a784c1fd67b44ea42c) | RDoc 6.14.0を同期 |
| [439428c](https://github.com/ruby/ruby/commit/439428c8c5eb694a1262fee1e7c13766f065e992) | Ubuntuランナーで失敗するBundlerテスト例をスキップ |
| [baa5f15](https://github.com/ruby/ruby/commit/baa5f15b336c92c8ee8498056bd5d0e8d5b80f57) | Bundlerを同期し新しい仕様設定に適応 |
| [3941954](https://github.com/ruby/ruby/commit/3941954fd48265f5eeeb4d339ffe48699fbbe1ec) | 不要なGEM_PATH変更を削除 |
| VS in windows-2025 | Windows-2025ランナーで最新版Visual Studioを使用 |
| [d2ed304](https://github.com/ruby/ruby/commit/d2ed304fd9546494ef865b5992e9b038a633bac5) | Windows-2025ランナーで最新版WinSDKを使用 |
| [8b59ba8](https://github.com/ruby/ruby/commit/8b59ba89a8c365ccf4a40ee6936ee855095eaf83) | test-bundled-gemsにwindows-2025ランナーを使用 |
| vcpkg cache | vcpkgの組み込みバイナリキャッシュをactions/cacheに置換 |
| vcpkg-root | `--vcpkg-root`をscoopディレクトリで指定 |
| [942d64b](https://github.com/ruby/ruby/commit/942d64b428ea72929d66198ff8751c4980b94777) | Windows-2022ランナーで最新版Visual Studioを使用 |
| [fd8a67f](https://github.com/ruby/ruby/commit/fd8a67fc8c10326042989da7d4e49b90c2f27ebb) | vcpkg関連のステップを並べ替え（VsDevCmd.batの影響回避） |

### ℹ️ コード整理（影響なし）

| コミット | 内容 |
|---------|------|
| [bc2e95e](https://github.com/ruby/ruby/commit/bc2e95ee93d3e0584e4d916bed8bb0382762c4ec) | RDocの新しい設定オプションを適用 |
| [df48793](https://github.com/ruby/ruby/commit/df487932fad588f57b6fc77e032beaef9751eb98) | test/lib/helper.rbをruby/rdocリポジトリ専用に |
| [2de5cb2](https://github.com/ruby/ruby/commit/2de5cb2f13b7afaa6e2f4914eb934211255b4d37) | RDoc 6.14.0のインターフェース変更によりRBSテストをスキップ |
| [877ae93](https://github.com/ruby/ruby/commit/877ae93e83d67eb78460af98384d96e7361fbd99) | bundled_gems_specスイート初期化時にgems tmpを初期化 |
| [84a9063](https://github.com/ruby/ruby/commit/84a90636c5547f104ac382e996f44f0b2cab1050) | Visual C 17.14.1の誤警告を抑制（`-wd5287`） |
| [bb2c266](https://github.com/ruby/ruby/commit/bb2c266498c4791b66071f426936fc49588a0922) | windows-2025でのrakeテスト失敗を許容に設定 |
| [acb19e8](https://github.com/ruby/ruby/commit/acb19e8707093593e967b6af03d92da5c570ffc6) | [Backport #21370] 一度バックポートされたがCIテスト失敗でrevert。3.4.5には未適用。3.4.7以降で修正見込み |
| [cfdc246](https://github.com/ruby/ruby/commit/cfdc2465d9fcd14eba512bfa80b5fd7c9e67f18e) | vcpkgキャッシュのrestore/saveをaction/cacheから分離 |

---

## ステップ4・5: コード深掘り — Bug #21380

### 選定理由

`String#split`はRubyで最も日常的に使うメソッドの一つ。3.4.4のBug #21331（`Hash#transform_values!`のuse-after-free）と同じ「イテレーション中の変更を検知して安全にエラーにする」パターンなので、比較して学べる。

### テストコード（test/ruby/test_string.rb）

```ruby
s = S("abc ") * 20
assert_raise(RuntimeError) {
  10.times do
    s.split {s.prepend("xxx" * 100)}
  end
}
```

- `S("abc ") * 20` → 十分に長い文字列を作る（短いとRuby内部の最適化でメモリ再確保が起きにくい）
- `s.prepend("xxx" * 100)` → splitの処理中に文字列の先頭に大量のデータを追加。これによりRubyがメモリを再確保するので、元のメモリアドレスが無効になる
- **期待動作**: `RuntimeError`が発生すること
- **修正前**: use-after-free（クラッシュ）
- **修正後**: `RuntimeError`で安全に停止

### 本体コード（string.c）

```diff
- #define SPLIT_STR(beg, len) (empty_count = split_string(result, str, beg, len, empty_count))
-     beg = 0;
-     char *ptr = RSTRING_PTR(str);
-     char *eptr = RSTRING_END(str);
+ #define SPLIT_STR(beg, len) ( \
+         empty_count = split_string(result, str, beg, len, empty_count), \
+         str_mod_check(str, str_start, str_len))
+
+     beg = 0;
+     char *ptr = RSTRING_PTR(str);
+     char *const str_start = ptr;
+     const long str_len = RSTRING_LEN(str);
+     char *const eptr = str_start + str_len;
```

### 修正のポイント

**修正前の問題:**
`split`は最初に文字列のメモリアドレス（`ptr`）と末尾（`eptr`）を取得して、それを信じて読み続ける。ブロック内で`s.prepend(...)`されるとメモリが再確保されるが、`split`は古いアドレスをまだ見ている → 解放済みメモリへのアクセス（use-after-free）。

**修正後の仕組み:**
1. `str_start`（最初のアドレス）と`str_len`（最初の長さ）を`const`で保存して基準点にする
2. `SPLIT_STR`マクロに`str_mod_check(str, str_start, str_len)`を追加
3. 分割のたびに「今の文字列のアドレスと長さが基準点と変わっていないか」をチェック
4. 変わっていたら → `RuntimeError`で安全に停止

**Rubyで表現するとこういうこと:**
```ruby
# split内部のイメージ
元の場所 = s.object_id
元の長さ = s.length

s.each_split_part do |part|
  yield part  # ← ここでユーザーのブロックが実行される（s.prependなど）

  # ★ 修正で追加されたチェック
  if s.object_id != 元の場所 || s.length != 元の長さ
    raise RuntimeError, "string modified"  # 引っ越してたら止める！
  end
end
```

### 3.4.4 Bug #21331 との比較

| | Bug #21331（3.4.4） | Bug #21380（3.4.5） |
|---|---|---|
| **対象メソッド** | `Hash#transform_values!` | `String#split` |
| **問題** | イテレーション中にrehash | split中に文字列変更 |
| **修正方法** | `hash_iter_lev_inc`でフラグを立てる | `str_mod_check`でアドレス・長さを比較 |
| **共通パターン** | **処理中にデータを変更されたら検知して、クラッシュではなくRuntimeErrorで安全に止める** |

### コミットの経緯

1. `fee9200` — splitメソッド実行中に文字列が変更された場合、`str_mod_check`によるチェックを追加し、RuntimeErrorで安全に停止するように修正