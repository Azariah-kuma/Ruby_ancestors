# Ruby_ancestors
Ruby技術者認定試験対策の継承関係をまとめてみる

## Rubyの継承関係で知っておくべきこと

- silverの範囲
  - **すべてのクラスは、Classクラスから継承されている。**
  - Classクラス(トップレベル)は、生成機能をもっている。
  - スーパークラスを明示せずクラス定義すると、暗黙的にObjectクラスを継承する。
  - クラス名は定数であることを覚えておくと、Goldの範囲に入っていきやすくなる。
    - 但し、関数型言語を扱う際は、クラス名=定数の概念はよろしくないとのことで注意が必要。

- goldの範囲
  - ClassクラスはModuleクラスを継承しており、Moduleクラスのsuper classはObjectクラスである。(図に描くとループする。)
  - モジュールのMix-inは、インタプリタがモジュールから無名クラスを生成して継承関係に組込んでいる。
  - 特異クラス(singleton class)は特異メソッド定義、或いは特異クラス式評価を確認したときに生成される。

上記を前提条件として、クラスの継承関係をまとめていきます。
Ruby技術者認定試験のGoldは、クラス/モジュールまわりだけでも半分程回答出来るので頑張っていきましょう。

## イディオムといえる記法でクラス定義する

定数Fooへの代入の部分は、恐らく"初見殺し"と感じる方も多いのではないでしょうか？

```ruby
# ここは明示せずともObjectクラスを継承してBarクラスを生成
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
# 生成インスタンスをレシーバに、メソッドを呼び出す
p foo.f_method(" welcome!")

#=> "hello kuma welcome!"
```

この記法だと特にわかり易くなるのが「定数に、Classクラスのオブジェクト=インスタンスを割り当てている」ということですね。
これは「クラスはオブジェクトである」ことの根拠のひとつなのですが、インスタンス変数やメソッドが絡むので、別途まとめを作りたいと思います。

*ary = Array.new()をイメージすると、クラスをインスタンスとして生成していることが理解しやすくなります。
*do～endはお馴染みのブロックなので、処理のかたまりと認識すると、こちらも理解しやすくなるのではないでしょうか。

```ruby
# とは言え、この継承の基本がないと始まらないので原点回帰用に書いておく
class Foo < Bar
  ...
end
```

## 実はよく使っている特異クラス

元々、RubyはObjectクラスより上位の階層を
**意識しなくても使えるようにデザインされている**わけですが、
更に「意識させたくないクラス」が存在しています。

それが特異クラスです。
特異クラスは「特定のオブジェクトにのみ適用されるクラス」と定義付けされており、オブジェクトを1つしか持てない特徴があります。

```ruby
# お馴染みインスタンス生成メソッドnew
Class.new()
```

よく見ると、クラスを固有オブジェクトとしてnewメソッドを呼び出しているのにお気付きでしょうか？
これは「クラスの特異メソッド=クラスメソッド」を定義しているので、クラス=オブジェクトとして振る舞い、メソッド呼び出しができるようになっています。

- クラスメソッドの定義方法
```ruby
# 特異メソッド形式、単体に定義するのに向いている
def self.class_method
   ...
end

# 特異クラス形式、複数に定義するのに向いている
# 何よりメソッドにself.の記述をしなくても補完してくれるのが有難い
class << self
  def class_method
   ...
end
```

早速、実際に動作させて確認してみましょう。
```ruby
Baz = Class.new do
  def inst_method
    puts "I am instance_method"
  end
# 特異メソッド形式で定義する
  def self.class_method
    puts "I am class_method"
  end
end
# インスタンスメソッド呼び出しなので、newで生成したインスタンスをレシーバにする
Baz.new.inst_method  #=> I am instance_method
# クラスメソッド呼び出しなので、固有オブジェクトであるクラスをレシーバにする
Baz.class_method  #=> I am class_method
```
```ruby
Baz = Class.new do
  def inst_method
    puts "I am instance_method"
  end
# 特異クラス形式で定義する
# self(今回はBazクラス)の特異クラスを開いて直接特異メソッドを定義
  class << self
    def class_method
      puts "I am class_method"
    end
  end
end
# 定義は同じなので、呼び出しも実行結果も同じ
Baz.new.inst_method  #=> I am instance_method
Baz.class_method  #=> I am class_method

```

前提条件として申し上げた通り「特異メソッドを定義すると特異クラスが生成される」ため、
通常の継承チェーン同様、特異クラスチェーンも生成されています。

なお、るりまを御覧頂くと明記されております通り、newメソッドはClassクラスに定義されている特異メソッドのため、全ての特異クラスがnewメソッドを継承しているのですね。

## Mix-inによる継承関係
- include 上に
- prepend 下に
- extend 特異クラスに

coming soon...
