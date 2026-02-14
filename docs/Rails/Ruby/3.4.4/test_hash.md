## test_transform_values_bangについて
```ruby
def test_transform_values_bang
  # @cls => ハッシュに値を入れている
  x = @cls[a: 1, b: 2, c: 3]
  # transform_values! => ハッシュの値だけを変換するメソッド
  y = x.transform_values! {|v| v ** 2 }
  assert_equal([1, 4, 9], y.values_at(:a, :b, :c))
  # xとyが同じオブジェクトかどうかを判定
  assert_same(x, y)

  x = @cls[a: 1, b: 2, c: 3]
  ## v == 2ならbreakを実行する（数値のまま返す）
  ## v == 3の時はv == 2でbreakされているため実行されない
  x.transform_values! { |v| v == 2 && break }
  assert_equal({a: false, b: 2, c: 3}, x)

  x = @cls[a: 1, b: 2, c: 3]
  # with_index => ブロックに第2引数としてインデックス（0から始まる連番）を渡す
  y = x.transform_values!.with_index {|v, i| "#{v}.#{i}" }
  assert_equal(%w(1.0  2.1  3.2), y.values_at(:a, :b, :c))

  x = @cls[a: 1, b: 2, c: 3, d: 4, e: 5, f: 6, g: 7, h: 8, i: 9, j: 10]
  assert_raise(FrozenError) do
    x.transform_values!() do |v|
    x.freeze if v == 2
    # 次の文字列を返す(1だったら2)
    v.succ
    end
  end
  assert_equal(@cls[a: 2, b: 2, c: 3, d: 4, e: 5, f: 6, g: 7, h: 8, i: 9, j: 10], x)

  x = (1..1337).to_h {|k| [k, k]}
    # 例外の発生とエラーメッセージを確認するメソッド
    assert_raise_with_message(RuntimeError, /rehash during iteration/) do
      x.transform_values! {|v|
      # ハッシュの構造を再計算
      # 1337なのは、数字が少ないと配列で管理するから
      # 1337なら、ハッシュテーブルで管理できる
      x.rehash if v == 1337
      v * 2
    }
  end
end
```