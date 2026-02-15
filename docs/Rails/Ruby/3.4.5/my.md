# Ruby 3.4.5 コードリーディングノート

- 対象バージョン: 3.4.4 → 3.4.5
- リリース日: 2025年07月15日
- リリース種別: 通常リリース

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

- https://www.ruby-lang.org/en/news/2025/07/15/ruby-3-4-5-released/
- https://github.com/ruby/ruby/releases/tag/v3_4_5
- https://github.com/ruby/ruby/compare/v3_4_4...v3_4_5

---

## ステップ2: チケット詳細

### ⚠️ 自分のコードに影響する可能性があるもの
- なし

### ℹ️ 環境依存の修正
#### Sync lockfile from rubygems/rubygems
https://github.com/ruby/ruby/pull/13472
- rakeリリースのたびにBundler CIが動作しなかったバグの修正
  - rubyの安定ブランチがロックファイルを使用しないため、このジョブもロックファイルを使用しないため
  - 📝ロックファイル：Gemfile.lockのこと

#### Bug #21255：Can't build Ruby with Windows SDK 10.0.26100
https://bugs.ruby-lang.org/issues/21255
- windowsSDKでinstallできないため、SDKをダウングレード

#### Backport GH-13617 for s390x
https://github.com/ruby/ruby/pull/13757
- s390xアーキテクチャで失敗していたテストをスキップ
  - 📝s390x：s390x はビッグエンディアンかつ独自のメモリアライメント要件を持つアーキテクチャ
  - 📝ビッグエンディアン：コンピュータがメモリに複数バイトのデータを格納するときの並び順の方式。間が数字を読む順番と同じで、上位バイトが先頭に来る
  - 📝メモリアライメント：CPUがメモリからデータを読み書きする際のデータの配置場所

#### Bump up resolv-0.6.2 for Ruby 3.4
https://github.com/ruby/ruby/pull/13818
- windows向けの機能変更が含まれていたためresolvライブラリをバージョンアップ
  - 📝resolv：Rubyの標準ライブラリに含まれるDNS名前解決のためのライブラリ

#### Bug #21500：Backport gcc 15 support
https://bugs.ruby-lang.org/issues/21500
- gcc15をサポートするように修正

### ℹ️ 通常は影響なしの修正

#### Bug #21340:Bump autoconf version to properly handle C23 bool/stdbool defines
https://bugs.ruby-lang.org/issues/21340
- C23 bool/stdbool定義を適切に処理するためにautoconfバージョンを上げた
  - Rubyの古い./configureのテストがC23ではbool/true/falseの新しいネイティブ定義のために機能しない
    - 📝C23：C言語の最新の国際標準規格。正式名称は ISO/IEC 9899:2024
    - 📝stdbool：C言語で ブーリアン型（真偽値） を使うために用意された標準ヘッダファイル

#### Bug #21438：use-after-free when resizing exivars
https://bugs.ruby-lang.org/issues/21438
- 汎用インスタンス(`exivars`)をリサイズする際に、メモリ解放後にアクセスされバグが発生

#### Ensure that memory is not freed before calling free_fast_fallback_getaddrinfo_*
https://github.com/ruby/ruby/pull/12661
- 内部構造のバグ修正

#### Fix heap-use-after-free in free_fast_fallback_getaddrinfo_entry
https://github.com/ruby/ruby/pull/13231
- 内部構造のバグ修正

#### Bug #21441:SEGV during thread cleanup if profiler calls thread_profiles_frames at wrong time
- vm.cを間違ったタイミングで呼び出すと、SEGVが発生するバグの修正
  - 📝SEGV：プログラムがアクセスを許可されていないメモリ領域に読み書きしようとしたときにOSが発するシグナル

#### Bug #21197：Prism does not accept newline after defined? keyword
https://bugs.ruby-lang.org/issues/21197
- defined?で改行が入っていても受け入れられるように修正

#### Bug #21333：heap-use-after-free caused by rehash during update
https://bugs.ruby-lang.org/issues/21333
- to_hでハッシュ後にrehashで再ハッシュするとヒープ使用後の解放が発生するバグの修正
  - 📝ヒープ（メモリ）：プログラムが動的にメモリを確保するための領域。malloc/freeで管理される

#### Bug #21357：Crash in Hash#merge! with ruby-dev in rubocop-rspec test suite
https://bugs.ruby-lang.org/issues/21357
- `merge`などのハッシュに関するメソッドでクラッシュする件の修正

#### Bug #21383：Prism leaks memory with invalid yield
- 無効なyieldによるメモリリークの修正
  - 📝yield：メソッド呼び出しの際に引数と一緒に渡すことのできる処理のかたまり

#### Bug #21394：Memory leak in Prism's RubyVM::InstructionSequence.new
https://bugs.ruby-lang.org/issues/21394
- `RubyVM::InstructionSequence.new`したときのメモリリークの修正

#### Bug #21099：TestGc#test_gc_stress_at_startup assertion failure
https://bugs.ruby-lang.org/issues/21099
- test_gc_stress_at_startupのassertionが失敗するバグの修正
  - Ractorリストが初期化されていない状態で起動時にGCが実行されると、`newobj_cache`がクリアされないためクラッシュ
    - 📝Ractor：Ruby 3.0 で導入された並列実行の仕組み
    - 📝GC：(Garbage Collection / ガベージコレクション)は、プログラムが使い終わったメモリを自動的に回収する仕組み

#### Bug #21395：Please backport caa6ba1a46afa1bc696adc5fe91ee992f9570c89
https://bugs.ruby-lang.org/issues/21395
- debug.gemを使用しているときに、`rescue`と`ensure`のフレームがバックトレースから除外されるようになったバグの修正
  - 📝バックポート：新しいバージョン（master/開発版）で行われた修正を、古い安定版ブランチにも適用すること

#### Bug #21439：Crash with PM_SPLAT_NODE compiler error (Prism)
https://bugs.ruby-lang.org/issues/21439
- `PM_SPLAT_NODE`のコンパイラーエラーのバグ修正
- forループの左側にスプラット(*)を使用するとクラッシュが発生する

#### Bug #21354：Symbol#to_proc is not ractor safe
https://bugs.ruby-lang.org/issues/21354
- `to_proc`をラクターセーフに修正
  - 📝ラクターセーフ：Ractorセーフとは、複数のRactorから同時に使っても安全に動作すること

#### Bug #20009：Marshal.load raises exception when load dumped class include non-ASCII
https://bugs.ruby-lang.org/issues/20009
- `Marshal.load`に非ASCII文字が含まれている場合に例外が発生するバグの修正
  - デフォルトのエンコーディング情報をUTF-8と想定する
  - クラス名に日本語などの非ASCII文字を使うと、`Marshal.dump` → `Marshal.load` で例外が発生していた
    - 日本語クラスを使用していないので、影響なし
    - 📝非ASCII文字：ASCII（American Standard Code for Information Interchange）は、英数字や基本的な記号（A-Z, a-z, 0-9, !, @ など）を7ビット（0〜127）で表す文字コード

#### Bug #21380：Use-After-Free in String#split with In-Block String Modification
https://bugs.ruby-lang.org/issues/21380
- `split`メソッド内部で文字列を変更すると、`use-after-free`が発生するバグの修正
  - 📝use-after-free：プログラムが既に解放（free）したメモリ領域に、その後もアクセスしてしまうバグのこと

#### Bug #21447：Fix handling of PM_CONSTANT_PATH_NODE node in keyword arguments with ARGS_SPLAT
- 最適化不可能なケースのテストを追加

#### Bug #21448：Random.urandom may fail to fall back to reading /dev/urandom on Linux < 3.17
https://bugs.ruby-lang.org/issues/21448
- `Random.urandom`が`/dev/urandom`の読み取りにフォールバックできない（Linux3.17未満）バグの修正
  - 📝フォールバック：最初に試した方法がうまくいかなかったときに、別の代替手段に切り替えること

#### Bug #21440：Cannot create instances of frozen Data subclasses
- Ruby3.2で`Data`が追加されたため失敗したバグ修正

#### Bug #21437：Date#hash may return different values for equal dates with large years
https://bugs.ruby-lang.org/issues/21437
- `Date#hash`が、同じ日付でも年数が大きい場合に異なる値を返すことがあるバグの修正

#### Bug #21497：building issue when using gcc15, because C23 is default
https://bugs.ruby-lang.org/issues/21497
- gcc15を使用するとC23がデフォルトであるためビルドに問題が発生するバグの修正

## ステップ3: リリースノートに載っていないコミット

### ⚠️ セキュリティ修正
- なし（リリースノートに全て記載済み）

### ℹ️ CI/インフラ（影響なし）

#### Disabled TRAP cache of CodeQL
https://github.com/ruby/ruby/commit/c104fc41b0e8ce80f4178c3b384d1405570714b2
- コードQLのキャッシュ無効。キャッシュが原因でCI（GitHub Actions）が不安定になったり、ディスク容量の問題が発生していた可能性があるため
  - 📝TRAPキャッシュ：CodeQLがソースコードを解析する際に生成する中間データの形式
  - 📝コードQL：GitHubが提供する静的コード解析ツール。ソースコードをデータベースのように扱い、クエリ（問い合わせ）を実行することで、セキュリティ脆弱性やバグのパターンを自動的に検出

#### windows-2025 runner removed D drive from their environment
<!-- TODO: 正しいコミットURLを確認する（439428c8は「Skip failing example on Ubuntu runner」のコミット） -->
- windows-2025ランナー環境からDドライブを削除

#### Merge RubyGems-3.6.8 and Bundler-2.6.8
https://github.com/ruby/ruby/commit/8125827578ed5ce3487f9ee08a4adc072f74a234
- gemとbundlerをアップデート

#### Merge RubyGems-3.6.9 and Bundler-2.6.9
https://github.com/ruby/ruby/commit/616771e34e651d2a09ab3dad77d826c2100512e8
- gemとbundlerをアップデート

#### Sync RDoc 6.14.0
https://github.com/ruby/ruby/commit/03eb777c69d64aa4941891a784c1fd67b44ea42c
- RDoc6.14.0を同期した

#### Skip failing example on Ubuntu runner of ruby/ruby
https://github.com/ruby/ruby/commit/439428c8c5eb694a1262fee1e7c13766f065e992
- Ubuntu の ruby​​/ruby ランナーで失敗する例をスキップした

#### Sync Bundler and adapt to new spec setup
https://github.com/ruby/ruby/commit/baa5f15b336c92c8ee8498056bd5d0e8d5b80f57
- Bundler を同期して新しい仕様設定に適応する

#### Remove unnecessary GEM_PATH modification
https://github.com/ruby/ruby/commit/3941954fd48265f5eeeb4d339ffe48699fbbe1ec
- 不要なGEM_PATH変更を削除した

#### Try to use the latest version of Visual Studio in windows-2025 runner.
- Windows-2025 ランナーで最新バージョンの Visual Studio を使用するように修正

#### Try to use the latest version of winsdk in windows-2025 runner
https://github.com/ruby/ruby/commit/d2ed304fd9546494ef865b5992e9b038a633bac5
- Windows-2025ランナーで最新バージョンのWinSDKを使用するようにした

#### Try to use windows-2025 runner for test-bundled-gems
https://github.com/ruby/ruby/commit/8b59ba89a8c365ccf4a40ee6936ee855095eaf83
- test-bundled-gemsにwindows-2025ランナーを使用した

#### Replaced built-in binary cache of vcpkg to actions/cache
- vcpkg の組み込みバイナリ キャッシュを actions/cache に置き換えました
  - vcpkgのインストール結果をキャッシュするように切り替えた
  - 📝vcpkg：Microsoft が開発したC/C++向けのパッケージマネージャ

#### Specified --vcpkg-root with scoop directory
- installする際にディレクトリで`--vcpkg-root`を指定

#### Use the latest version of Visual Studio with windows-2022 runner image
https://github.com/ruby/ruby/commit/942d64b428ea72929d66198ff8751c4980b94777
- Windows 2022 ランナー イメージで最新バージョンの Visual Studio を使用するように修正

#### Re-ordered vcpkg related steps. It may be affected with VsDevCmd.bat
https://github.com/ruby/ruby/commit/fd8a67fc8c10326042989da7d4e49b90c2f27ebb
- VsDevCmd.batの影響を受ける可能性があり、vcpkg関連の手順を並べ替えた
  - 📝VsDevCmd.bat：Visual Studio Developer Command Prompt を初期化するバッチファイル

### ℹ️ コード整理（影響なし）

#### Apply new RDoc config options
https://github.com/ruby/ruby/commit/bc2e95ee93d3e0584e4d916bed8bb0382762c4ec
- RDocの新しいオプションを追加した

#### test/lib/helper.rb is only for ruby/rdoc repo
https://github.com/ruby/ruby/commit/df487932fad588f57b6fc77e032beaef9751eb98
- test/lib/helper.rbをruby​​/rdocリポジトリ専用にした

#### Skip RBS tests for RDocPluginParserTest caused by interface change of RDoc 6.14.0
https://github.com/ruby/ruby/commit/2de5cb2f13b7afaa6e2f4914eb934211255b4d37
- RDoc6.14.0 のインターフェース変更により、`RDocPluginParserTest`のRBSテストをスキップした


#### Initialize gems tmp when initializing bundled_gems_spec suite
https://github.com/ruby/ruby/commit/877ae93e83d67eb78460af98384d96e7361fbd99
- gemsbundled_gems_specスイートを初期化するときに`tmp`を初期化
  - こうすることで、これまで`Bundler`仕様が実行されていない場合でも機能する

#### Win: Suppress false warnings from Visual C 17.14.1
https://github.com/ruby/ruby/commit/84a90636c5547f104ac382e996f44f0b2cab1050
- Win: Visual C 17.14.1 からの誤った警告を抑制するように修正
  - 明示的なキャストを使用しても、「オペランドが異なる列挙型です」という警告が出る

#### Added rake test to allow failures
https://github.com/ruby/ruby/commit/bb2c266498c4791b66071f426936fc49588a0922
- 失敗を許可するrakeテストを追加

#### merge revision(s) ff222ac: [Backport #21370]
https://github.com/ruby/ruby/commit/acb19e8707093593e967b6af03d92da5c570ffc6
- `https://bugs.ruby-lang.org/issues/21370`の修正
  - 一度バックポートされたがCIテストが失敗しrevert。3.4.5には未適用。3.4.7以降で修正される見込み

#### Split restore and save actions from action/cache. We need to save always vcpkg cache
https://github.com/ruby/ruby/commit/cfdc2465d9fcd14eba512bfa80b5fd7c9e67f18e
- vcpkgのキャッシュを常に保存する必要があるため、復元と保存のアクションをaction/cacheから分離

## ステップ4・5: コード深掘り
### Bug #21380（`split`メソッド内部で文字列を変更すると、`use-after-free`が発生するバグの修正）

#### テストコード
```ruby
s = S("abc ") * 20
assert_raise(RuntimeError) {
  10.times do
    # 大きい値にすることで、元のメモリアドレスを無効化している
    s.split {s.prepend("xxx" * 100)}
  end
}
```

- 期待動作:RuntimeErrorが発生する
- 修正前:`use-after-free`が発生
- 修正後:`RuntimeError`が発生（壊れるくらいなら明示的にエラーにする）

#### 本体コード（C言語だからよくわからない、、、）
```diff
- #define SPLIT_STR(beg, len) (empty_count = split_string(result, str, beg, len, empty_count))
-     beg = 0;
-     // splitが分割するたびに呼ぶマクロ
-     char *ptr = RSTRING_PTR(str); // 文字列のメモリアドレスを取得
-     char *eptr = RSTRING_END(str); // 文字列の末尾アドレスを取得
+ #define SPLIT_STR(beg, len) ( \
+         empty_count = split_string(result, str, beg, len, empty_count), \
+         str_mod_check(str, str_start, str_len))
+
+    beg = 0;
+    char *ptr = RSTRING_PTR(str);
+    char *const str_start = ptr;
+    const long str_len = RSTRING_LEN(str);
+    char *const eptr = str_start + str_len;
```

- 修正のポイント:
  - 修正前はsplitの処理中に文字列が書き換えられても気づかず、無効なメモリを読んでクラッシュしていた（use-after-free）
  - 修正後は分割のたびに「文字列のメモリアドレスと長さが最初と変わっていないか」をチェック（`str_mod_check`）し、変わっていたらRuntimeErrorで安全に止めるようになった
  - C言語の変更点：
    - `str_start`（最初のアドレス）と`str_len`（最初の長さ）を`const`で保存して基準点にする
    - `SPLIT_STR`マクロに`str_mod_check(str, str_start, str_len)`を追加し、分割のたびに基準点と比較する

#### コミットの経緯
1. `fee9200` — splitメソッド実行中に文字列が変更された場合、明示的にエラーを発生させて検知するように修正

---

## 📝 用語メモ

- **ロックファイル**: Gemfile.lockのこと
- **s390x**: ビッグエンディアンかつ独自のメモリアライメント要件を持つアーキテクチャ
- **ビッグエンディアン**: コンピュータがメモリに複数バイトのデータを格納するときの並び順の方式。人間が数字を読む順番と同じで、上位バイトが先頭に来る
- **メモリアライメント**: CPUがメモリからデータを読み書きする際のデータの配置場所
- **resolv**: Rubyの標準ライブラリに含まれるDNS名前解決のためのライブラリ
- **C23**: C言語の最新の国際標準規格。正式名称は ISO/IEC 9899:2024
- **stdbool**: C言語でブーリアン型（真偽値）を使うために用意された標準ヘッダファイル
- **SEGV**: プログラムがアクセスを許可されていないメモリ領域に読み書きしようとしたときにOSが発するシグナル
- **ヒープ（メモリ）**: プログラムが動的にメモリを確保するための領域。malloc/freeで管理される
- **yield**: メソッド呼び出しの際に引数と一緒に渡すことのできる処理のかたまり
- **Ractor**: Ruby 3.0 で導入された並列実行の仕組み
- **GC**: (Garbage Collection / ガベージコレクション) プログラムが使い終わったメモリを自動的に回収する仕組み
- **バックポート**: 新しいバージョン（master/開発版）で行われた修正を、古い安定版ブランチにも適用すること
- **ラクターセーフ**: 複数のRactorから同時に使っても安全に動作すること
- **非ASCII文字**: ASCII（英数字や基本的な記号を7ビットで表す文字コード）に含まれない文字。日本語など
- **use-after-free**: プログラムが既に解放（free）したメモリ領域に、その後もアクセスしてしまうバグのこと
- **フォールバック**: 最初に試した方法がうまくいかなかったときに、別の代替手段に切り替えること
- **TRAPキャッシュ**: CodeQLがソースコードを解析する際に生成する中間データの形式
- **CodeQL**: GitHubが提供する静的コード解析ツール。ソースコードをデータベースのように扱い、クエリで脆弱性やバグを自動検出
- **vcpkg**: Microsoftが開発したC/C++向けのパッケージマネージャ
- **VsDevCmd.bat**: Visual Studio Developer Command Promptを初期化するバッチファイル