# Ruby_ancestors
Ruby技術者認定試験対策の継承関係をまとめてみる

## Rubyの継承関係で知っておくべきこと

- silverの範囲
  - すべてのクラスは、Classクラスから継承されている。
  - Classクラス(トップレベル)は、生成機能をもっている。
  - スーパークラスを省略してクラス定義すると、暗黙的にObjectクラスを継承する。
  - クラス名は定数であることを覚えておくと、Goldの範囲に入っていきやすくなる。

- goldの範囲
  - ClassクラスはModuleクラスを継承しており、Moduleクラスのsuper classはObjectクラスである。(図に描くとループする。)
  - モジュールのMix-inは、インタプリタがモジュールから無名クラスを生成して継承関係に組込んでいる。
  - 特異クラス(singleton class)は特異メソッド定義、或いはスーパークラスの継承時に生成される。


## イディオム記法でクラス定義する

定数Fooへの代入の部分は、恐らく"初見殺し"と感じる方も多いのではないでしょうか？

```ruby
# ここはObjectクラスを継承してBarクラスを生成
class Bar
  def initialize(greet)
    @greet = greet
  end
end

# Fooという定数に、引数のBarクラスをスーパークラスとして
# Classクラスからクラス生成
Foo = Class.new(Bar) do
  def initialize(greet, name)
    @name = name
    super(greet)
  end

  def f_method(msg)
    @greet + @name + msg
  end
end

# インスタンスメソッドなので、クラスのインスタンスを生成しておくこと
foo = Foo.new("hello", " kuma")
# 生成インスタンスをレシーバに、
p foo.f_method(" welcome!")

#=> "hello kuma welcome!"
```
ary = Array.newをイメージすると、インスタンスとしてクラス生成していることが理解しやすくなり、
do～endはお馴染みのブロックなので、処理のかたまりと認識すると、こちらも理解しやすくなるのではないでしょうか。



coming soon...
