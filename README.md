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
  - ClassクラスはModuleクラスを継承しており、ModuleクラスのsuperclassはObjectクラスである。(図に描くとループする。)
  - モジュールのMix-inは、インタプリタがモジュールに対応した無名クラスを生成して継承関係に組込んでいる。
  - 特異クラス(singleton class)は特異メソッド定義、或いは特異クラス式評価を確認したときに生成される。

上記を前提条件として、クラスの継承関係をまとめていきます。
Ruby技術者認定試験のGoldは、クラス/モジュールまわりだけでも半分近く回答出来るので頑張っていきましょう。

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

  - ary = Array.new()をイメージすると、クラスをインスタンスとして生成していることが理解しやすくなります。
  - do～endはお馴染みのブロックなので、処理のかたまりと認識すると、こちらも理解しやすくなるのではないでしょうか。

```ruby
# とは言え、この継承の基本がないと始まらないので原点回帰用に書いておく
class Foo < Bar
  ...
end
```
なお、書籍でよく紹介される「レシーバのオブジェクトからひとつ右へ、それからクラス継承チェーンを上へ辿る」図を描くと理解しやすくなるのでオススメです。

## 実はよく使っている特異クラス

元々、RubyはObjectクラスより上位の階層を
**意識しなくても使えるようにデザインされている**わけですが、
更に「意識させたくないクラス」が存在しています。

そのひとつが「特異クラス」です。
特異クラスは「特定のオブジェクトにのみ適用されるクラス」と定義付けされており、オブジェクトを1つしか持てない特徴があります。

```ruby
# お馴染みインスタンス生成メソッドnew
Class.new()
```

よく見ると、クラスを固有オブジェクトとしてnewメソッドを呼び出しているのにお気付きでしょうか？
これは「クラスの特異メソッド=クラスメソッド」を定義しているので、クラス=オブジェクトとして振る舞い、メソッド呼び出しができるようになっています。

#### クラスメソッドの定義方法
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

では、実際に動作させて確認してみましょう。
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

これは図で表すとき「クラス継承チェーンを上へ辿る」図の右側に「特異クラスのみの継承チェーン」をModuleクラスまで描くと理解しやすくなるのではないでしょうか。

また、るりまを御覧頂くと明記されております通り、newメソッドはClassクラスに定義されている特異メソッドのため、全ての特異クラスがnewメソッドを継承しているのですね。

## Mix-inによる継承関係
よく使うMix-inですが、Module#ancestorsメソッドで確認すると、Kernelを含めたモジュール名が継承関係に組込まれていることに「不思議」と感じたことはありませんでしょうか？

```ruby
module Hoge
  def hoge; puts "hoge"; end
end

module Fuga
  def fuga; puts "fuga"; end
end

class Foo
  include Hoge
  def foo; puts "foo"; end
end

class Bar < Foo
  def bar; puts "bar"; end
  prepend Fuga
end

Bar.ancestors #=> [Fuga, Bar, Foo, Hoge, Object, Kernel, BasicObject]

```

モジュールの特徴として
  - インスタンスを持つことが出来ない
  - 継承関係を持つことが出来ない

という2点が挙げられるのですが、これはどういうことなのでしょうか？

実はRubyの意識させたくないクラスのひとつ「無名クラス」を利用して実装されています。
内部では、Rubyのインタプリタが判断し、「モジュールに対応した無名クラス」を生成し、継承チェーンに割込み拡張しているのです。

また、意識させたくないクラスであることから、この無名クラスはModule#ancestorsやClass#superclassといったメソッドでも参照出来ない仕組みになっているため、複雑に感じる部分でもあります。

とはいえ、Rubyの優秀なインタプリタが実装していることを踏まえて、動作させてみるのが理解への1番の近道だと感じます。

#### include
呼び出したクラスの上に、モジュールと対応した無名クラスが挿入されます。
```ruby
module M1
  def method1; puts "M1"; end
end

module M2
  def method1; puts "M2"; end
end

class C1
  def method1; puts "C1"; end
end

class C2 < C1
  include M1
  def method1; puts "C2"; end
  include M2
end

C2.new.method1
```
さて、この実行結果はどうなるでしょうか？
ポリテクの授業では、includeのみ説明がありましたが、皆さん「えっ？」となっていた部分ですので少しだけ解説致しますね。

まず、実行結果は探索経路がカレントクラス優先となりますので、答えはC2のメソッド呼び出しが優先となります。
```ruby
C2.new.method1 #=> "C2"
```
  - モジュールの機能として、名前空間が提供されているため、同名メソッドであっても衝突は起こらない。よってErrorにはならない。
  - includeしただけでは継承チェーンに組み込まれるだけなので、呼び出しメソッドがカレントクラスにあれば、そのメソッドが実行される。
```ruby
# ancestorsで確認する
C2.ancestors #=> [C2, M2, M1, C1, Object, Kernel, BasicObject]
```
上記を覚えておくと、格段に理解しやすくなるのではないでしょうか。

次に、カレントクラスのメソッドが無い場合はどうなるでしようか？上記のC2クラスのメソッドをコメントアウトしてみてくださいませ。
```ruby
C2.new.method1 #=> "M2"
```
この場合は、後にincludeしたモジュールが実行されます。
  - カレントクラスから見た探索経路で1番近い箇所で見つかったメソッドが実行される。
  - 複数のインクルードを実行すると、後に記述したモジュールが呼び出しクラスの直ぐ上に割込み挿入される。

上記を理解しておくと、複数のインクルードがなされてもほぼ解釈を間違える事は無くなりますね。

#### prepend
呼び出したクラスの下に、モジュールと対応した無名クラスが挿入されます。
```ruby
module M1
  def method1; puts "M1"; end
end

module M2
  def method1; puts "M2"; end
end

module M3
  def method1; puts "M3"; end
end

class C1
  def method1; puts "C1"; end
end

class C2 < C1
  prepend M1
  def method1; puts "C2"; end
  prepend M2
  include M3
end

C2.new.method1
```
includeの説明で使ったコードを少しだけ複雑に見せ掛けてみました。この実行結果はどうなるでしょうか？
```ruby
C2.new.method1 #=> "M2"
```
includeでどこに挿入されるかを理解しておくと、挿入方向が下になるだけなので、簡単に回答出来ることでしょう。

prependの場合、探索経路の1番手前に配置されます。
```ruby
# ancestorsで確認する
C2.ancestors #=> [M2, M1, C2, M3, C1, Object, Kernel, BasicObject]
```
ancestorsの結果は割と試験でも出る内容ですので、是非沢山コードを動かして見てくださいませ。

#### extend
固有オブジェクトに定義した特異クラスに、includeされる。
```ruby
# extendの使い方
# 特異クラスBazにPiyoモジュールをインクルードする
baz = Baz.new
baz.extend(Piyo)
p baz.piyo_method

# 上記で行われている内容として
# Bazクラスのインスタンスbazを固有オブジェクトとして
baz = Baz.new
# 特異クラスを開いて
class << baz
# Piyoモジュールをインクルードする
  include Piyo
end
# 特異メソッドなので、固有オブジェクトをレシーバに呼び出す
p baz.piyo_method
```
この場合の継承関係は、特異クラスチェーンに「baz→Piyo→Baz→...」と図にすると確認し易いのではないでしょうか。
なお、extendはancestorsでは確認出来ない仕様になっております。

## Gold試験で問われる関連メソッド
  - Module#append_features
    - includeの実体であるメソッド。
    - インクルードされる前に呼び出される。
    - 引数にインクルードされるモジュールが入る。
    - また、モジュール/クラスにselfの機能を追加する。
    - superの記述が無いと上書き時にインクルード出来ない仕様になっている。

  - Module#included
    - インクルード後に呼び出される。
    - インクルード後にやりたい処理を実装しておくと、フックメソッドにできる。

  - Module#ancestors
    - レシーバがModule/Classクラスのインスタンスの場合のみ有効。
    - クラス/モジュールの、superclassとインクルードしているモジュールの優先順を配列で返却。

## 最後に
ここまでの長文をお読みいただき、ありがとうございます。
これで、一通りの継承関係を解説致しました。

詳細部分をもう少し噛み砕いたものをつくりたかったのですが、私自身が今週末に資格試験を控えておりますので、もう少し掘り下げた内容は受験後に改めて編集しようと考えております。少しでもお役にたてれば幸いです。
