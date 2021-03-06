### 6.5 特異メソッド


Rubyのクラスはオープンクラスと言われ、定義後に好きな場所でメソッドの再定義や追加ができる。このオープンクラスの特徴を使って実際にクラス定義を書き換えることをモンキーパッチという。では実際にモンキーパッチを利用してプログラムを書いてみよう。

```ruby
# Arrayクラスにメソッドを追加
class Array
  # メディアン（中央値）を計算
  def median
    size = self.size
    half = size / 2
    dec_half = half - 1
    if size % 2 == 0
      (self[dec_half] + self[half]) / 2.0
    else
      self[half]
    end
  end
end

array1 = [1, 2, 3, 4, 5]
array2 = [1, 2, 3, 4, 5, 6]

p array1.median
#=> 3

p array2.median
#=> 3.5
```

モンキーパッチはどこでも簡単にクラスを拡張できるので非常に便利な機能だが、使い方によっては危険な機能なのでモンキーパッチという言葉は批判的な意味を込めている場合がある。次のコードは危険な使い方の例である。

```ruby
class Array
  # メディアン（中央値）を計算
  def median
    size = self.size
    half = size / 2
    dec_half = half - 1
    if size % 2 == 0
      (self[dec_half] + self[half]) / 2.0
    else
      self[half]
    end
  end

  # sizeメソッドを書き換え
  def size
    1
  end
end

array1 = [1, 2, 3, 4, 5]
array2 = [1, 2, 3, 4, 5, 6]

p array1.median
#=> 1

p array2.median
#=> 1
```

Arrayクラスに元々定義してあるsizeメソッドを書き換えてしまうと、medianメソッドを含めてsizeメソッドを利用しているメソッドの動作がおかしくなってしまう。モンキーパッチをするときには既存のメソッドを書き換えることで起きる副作用をよく考えよう。

次に、モンキーパッチを利用した特異メソッドと呼ばれるメソッドの定義について解説する。クラスに定義されたメソッドはそのクラスのインスタンスならば全てオブジェクトが持っているものだが、特異メソッドはクラスに定義せずに、オブジェクトに対してあとから追加するもので、追加したオブジェクトしかメソッドを持たない。

```ruby
class Human
# ...
end

jhon = Human.new('Jhon', 18)
def jhon.callName
	puts "I\'m Jhon"
end

jhon.callName
#=> I'm Jhon
```

また、特異メソッドはクラス変数とクラスメソッドにはアクセスできないが、インスタンス変数とインスタンスメソッドにアクセスできるので、以下のようにもできる。

```ruby
def jhon.callName
	puts "I\'m #{@name}"
end
```

特異メソッドはクラスに対しても追加できる。クラスに追加する場合はクラス変数やクラスメソッドにアクセスできるように思えるが、オブジェクトに対しての特異メソッドと同様にインスタンス変数とインスタンスメソッドにしかアクセスできない。

```ruby
def Human.callName
	puts "I\'m #{@name}"
end
```

特異メソッドは動的にメソッドを追加できるという特徴から、メタプログラミングに使われることが多い。メタプログラミングとはメソッドを定義するメソッド、クラスを定義するメソッドなどの、プログラム上の処理を生成するための手続きを作るためのプログラミングのことで、言語の構文ではないが構文であるかのような記述を可能にする、内部DSLに使われる。Rubyに標準で定義されている内部DSLは、yieldメソッド、raiseメソッド、アクセサなどがある。他にも、Rails、Rspec、Sinatraといったライブラリを組み込むことで利用可能になる内部DSLもある。

```ruby
require 'sinatra'

get '/' do
    'get'
end
  
post '/' do
    'post'
end
```

これはsinatraというライブラリを使って内部DSLでWebサーバを書いた例である。getやpostはRubyに標準で定義されている構文やメソッドではないが、あたかもこの様な構文があるかのように書くことができる。特定の目的に対してこの内部DSLは可読性の向上やコードを直感的に書けるという点で非常に有効である。