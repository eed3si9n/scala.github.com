---
layout: tour
title: クラス
language: ja

discourse: true

partof: scala-tour

num: 4
next-page: traits
previous-page: unified-types
topics: classes
prerequisite-knowledge: no-return-keyword, type-declaration-syntax, string-interpolation, procedures

redirect_from: "/tutorials/tour/classes.html"
---

Scalaにおけるクラスはオブジェクトを作るための設計図です。
それらはメソッド、値、変数、型、オブジェクト、トレイト、クラスを持ち、まとめて _メンバー_ と呼ばれます。
型、オブジェクト、トレイトはツアーで後ほど取り扱います。

## クラスを定義する

最小のクラス定義は単純に`class`キーワードと識別子になります。
クラス名はキャメルケースであるべきです。

```tut
class User

val user1 = new User
```
`new`キーワードはクラスのインスタンスを作るために使われます。
`User`は引数を受け取らないデフォルトコンストラクターを持ちます。なぜならコンストラクターは定義されていないからです。
しかしながら、コンストラクターとクラス本体は頻繁に欲しくなるでしょう。
こちらは位置情報のクラス定義の例になります。

```tut
class Point(var x: Int, var y: Int) {

  def move(dx: Int, dy: Int): Unit = {
    x = x + dx
    y = y + dy
  }

  override def toString: String =
    s"($x, $y)"
}

val point1 = new Point(2, 3)
point1.x  // 2
println(point1)  // prints (2, 3)
```
この`Point`クラスは4つのメンバーを持ちます。
変数`x` と `y` そしてメソッド `move` と `toString`です。
多くの他の言語とは異なり、プライマリコンストラクタはクラスのシグネイチャになります。
（シグネイチャはメソッドの名、そのメソッドに対する引数の数と型で構成されています。）
`move`メソッドは2つのInteger引数を受け取り、意味を運ばないUnitの値である`()` を返します。
これは乱暴に言えば、Javaのような言語における`void`と一致します。
その一方で`toString`は引数を受け取りませんが、`String`の値を返します。
なぜなら[`AnyRef`](unified-types.html)の`toString`を`toString`がオーバーライドするためで、それは`override`キーワードでタグ付けされます。

## コンストラクター

コンストラクターはデフォルト値を提供することで、以下のようにオプションパラメーターを持つことができます。

```tut
class Point(var x: Int = 0, var y: Int = 0)

val origin = new Point  // x と y には共に0がセットされます。
val point1 = new Point(1)
println(point1.x)  // 1 が出力されます。
```
このバージョンの`Point`クラスでは、`x` と `y` はデフォルト値0を持ち、引数が必須ではありません。
しかしながらコンストラクタは引数を左から右に読み込むため、もし`y`の値だけを渡したい場合は、パラメーターに名前をつけてやる必要があります。

```
class Point(var x: Int = 0, var y: Int = 0)
val point2 = new Point(y=2)
println(point2.y)  // 2 が出力されます。
```

これは明快さを高める良い習慣にもなります。

## プライベートメンバーとゲッター/セッター構文
メンバーはデフォルトではパブリックになります。
クラスの外から隠したい場合は`private`アクセス修飾詞を使いましょう。

```tut
class Point {
  private var _x = 0
  private var _y = 0
  private val bound = 100

  def x = _x
  def x_= (newValue: Int): Unit = {
    if (newValue < bound) _x = newValue else printWarning
  }

  def y = _y
  def y_= (newValue: Int): Unit = {
    if (newValue < bound) _y = newValue else printWarning
  }

  private def printWarning = println("WARNING: Out of bounds")
}

val point1 = new Point
point1.x = 99
point1.y = 101 // 警告が出力されます。
```
このバージョンの`Point`クラスでは、データはプライベート変数 `_x` と `_y` に保存されます。
メソッド`def x` と `def y` はプライベートなデータにアクセスするためのものです。
`def x_=` と `def y_=` は `_x` と `_y` の値を検証し設定するためのものになります。
セッターのための特別な構文に注意してください。
セッターメソッドはゲッターメソッドの識別子に`_=`を追加し、その後ろにパラメーターを取ります。

プライマリコンストラクタの`val` と `var` を持つパラメーターはパブリックになります。
しかしながら`val` は不変となるため、以下のように記述することはできません。

```
class Point(val x: Int, val y: Int)
val point = new Point(1, 2)
point.x = 3  // <-- コンパイルされません。
```

`val` や `var` が存在しないパラメーターはクラス内でだけで参照できるプライベートな値や変数となります。
```
class Point(x: Int, y: Int)
val point = new Point(1, 2)
point.x  // <-- コンパイルされません。
```