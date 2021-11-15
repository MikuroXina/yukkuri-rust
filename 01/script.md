「まさか, 商店街近くにこんな穴場があるとは…

「ランチもおいしいんですよー

「うーん, 私はコーヒーパフェにでもしますか

「えー, もうちょっと甘いのにしましょーよー

「ちょっと高いので…

「じゃあじゃあ, 今日は私のおごりにするから〜♪

「珍しいですね, 何かあったんですか

「いやその, せっかくだから聞きたいこととかあるし…授業料みたいなもんですよ！

(さては食べ比べしたいだけですね…

「じゃあお言葉に甘えますか

「すいませーん！クリームあんみつと宇治金時で！

(待てよ, 財布に樋口あったっけ…

*OP*

「それで, 聞きたいことがあるんでしたね

「えっとね, Rust の `Option` とか `Result` の使い方がわかんない！

「あーなるほど……, まずは `Option` の定義から見ていきますか

「おっけー♪


「たしかこんな感じの定義でしたね

```rs
enum Option<T> {
  Some(T),
  None,
}
```

「そうそう, ソメとノンだよね

「サムとナンです…

「そ, そうとも言うし〜

「まぁ値が有るのか無いのかを示しているだけですね


「そしてこれが `Result` です

```rs
enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

「よく関数の戻り値になってたりするやつだ

「ポジティブな情報である `Ok` か, ネガティブな情報である `Err` のどちらかですね

「ここまで簡潔に書かれちゃうと, 何ができるのかよくわかんなくなっちゃうー


`map`

「`Option` や `Result` にはいろいろ便利なメソッドが `impl` されてます. 代表的なものはやはり `map` でしょう

「あーこれこれ. 使ってるのみたことある

「ソースコードを見るとそのまんまですが, このように中身を取り出して渡された関数に入れています

```rs
impl<T> Option<T> {
  fn map<U, F: FnOnce(T) -> U>(self, f: F) -> Option<U> {
    match self {
      Some(x) => Some(f(x)),
      None => None,
    }
  }
}

impl<T, E> Result<T, E> {
  pub fn map<U, F: FnOnce(T) -> U>(self, op: F) -> Result<U, E> {
    match self {
      Ok(t) => Ok(op(t)),
      Err(e) => Err(e),
    }
  }
}
```

「あー, C++ の関数テンプレートとおんなじ感じだ

`as_ref` `as_mut` `as_deref`

「`map` はムーブして所有権を奪うようになっています. しかし `&Option<T>` を `Option<&T>` に変換するようなメソッドも提供されているのでこれを使えば大丈夫です

```rs
let mut text: Option<String> = Some("Hello, world!".into());

let text_length: Option<usize> = text.as_ref().map(|s| s.len());

if let Some(n) = text.as_mut() {
  n.push('?');
}

let text_str: Option<&str> = text.as_deref();
```

「ふーん, いっつも `clone` しちゃってたかも……

「`&Option<String>` を `Option<&str>` に変換するみたいな処理は, 自分で `map` しなくても `as_deref` でできます

「へぇ〜便利ぃ

`unwrap` `expect` `unwrap_or`

「中身をそのまま取り出すときは `unwrap` 系の関数ですね

```rs
Some("air").unwrap(); // "air"
None.unwrap(); // パニックする

Some("car").unwrap_or("bike"); // "car"
None.unwrap_or("bike"); // "bike"

Some("an important message").expect("message must be provided");
None.expect("message must be provided"); // パニックする
```

「ふむふむ……, `expect` は何に使うの?

「これは取り出しに失敗すると指定のメッセージ付きでパニックするんです. 原因がわかりやすくなります

`and_then`

「中身の値から新しい `Option` や `Result` を作るときは `and_then` が便利ですね

```rs
impl<T> Option<T> {
  pub fn and_then<U, F: FnOnce(T) -> Option<U>>(self, f: F) -> Option<U> {
    match self {
      Some(x) => f(x),
      None => None,
    }
  }
}
```

「……これ要るの? そのまま新しいのに置き換えて, 要は再代入してるだけじゃん

「じゃあちょっと再代入で表現してみましょうか

「えーと, こんな感じかな?

```rs
/// 平方数のときだけその平方根を求める
fn sqrt_if_sq(x: u32) -> Option<u32> {
  for n in 1..x {
    if n * n == x {
      return Some(n);
    }
  }
  None
}

let mut opt = Some(9);
if let Some(n) = opt {
  opt = sqrt_if_sq(n);
}
println!("{:?}", opt);
```

「もうちょっと処理を増やしてみますか


```rs
let mut opt = Some(6561);
if let Some(n) = opt {
  opt = sqrt_if_sq(n);
}
if let Some(n) = opt {
  opt = sqrt_if_sq(n);
}
if let Some(n) = opt {
  opt = sqrt_if_sq(n);
}
if let Some(n) = opt {
  opt = sqrt_if_sq(n);
}
println!("{:?}", opt);
```

「おうおう, すごいことに

「そうやっていろんな処理をつなぎ始めると, 取り出す処理も増えていきます. `and_then` で書き直してみましょう

「こう, かな?

```rs
let opt = Some(6561)
  .and_then(sqrt_if_sq)
  .and_then(sqrt_if_sq)
  .and_then(sqrt_if_sq)
  .and_then(sqrt_if_sq);
println!("{:?}", opt);
```

「そうですね, だいぶわかりましたか?

「やっぱり取り出すとこはそれを持ってる方がやれば楽なんだねー

`ok_or` `ok` `err`

「最後に, `Option` と `Result` の相互変換を提供する関数です

```rs
let x: Option<&str> = Some("foo");
x.ok_or(0); // Ok("foo")

let x: Option<&str> = None;
x.ok_or(0); // Err(0)


let x: Result<u32, &str> = Ok(2);
x.ok(); // Some(2)

let x: Result<u32, &str> = Err("Nothing here");
x.ok(); // None


let x: Result<u32, &str> = Ok(2);
x.err(); // None

let x: Result<u32, &str> = Err("Nothing here");
x.err(); // Some("Nothing here")
```

「おー, いつでも取っ換えられるんだ


「他にもいろいろ便利な関数が用意されてますから, `doc.rust-lang.org` で検索してみてください

「そいで結局, `Option` と `Result` はどっち使えばいいの?

「まぁ基本的には `Result` で, 提供できるエラーの情報がどうしても無いなら `Option` でしょう. それが存在しない理由が, 分かりきっている or 知りようが無いなら `Result` にする意味ないですし

「クリームあんみつと, 宇治金時になりまーす

「わー来た来た〜♪ いっただっきまーす!

*アイキャッチ*

「どんな関数のエラーも拾えるように, 最初から全部 `Result` だったらいいのにー. そう思いません〜?

「ふふ, すべてのエラーがハンドリングできるとは限りませんよ

「へー?

「

```rs
fn hoge() -> Result<HogePayload, HogeError> {
}
```

「


「他にも, 関数に正しい引数を渡すことを呼び出す側の責任にしたほうが, 数学的な関数の実装がシンプルになります

```rs
/// 平方根の逆数を高速に計算する.
fn inv_sqrt(x: f64) -> f64 {
  // x は正の数でなければならない.
  assert!(x.is_sign_positive());

  // mu := 0.045
  // log(x) ≒ x + mu
  // log(a) := log(1 / √x) = - 1/2 log(x)
  // 1/2^52 (a_man + 2^52 * a_exp) - 1023 + mu
  //  = - 1/2 (1/2^52 (x_man + 2^52 * x_exp) - 1023 + mu)
  // a_man + 2^52 * a_exp
  //  = 2^52 * (1023 - mu - 1/2 (1/2^52 (x_man + 2^52 * x_exp) - 1023 + mu))
  //  = 2^52 * (1023 - mu) - 1/2 ((x_man + 2^52 * x_exp) - 1023 + mu)
  //  = 2^52 * 3/2 (1023 - mu) - 1/2 (x_man + 2^52 * x_exp)
  // a = 0x5fe6_eb85_1eb8_5400 - (x >> 1)
  let i = x.to_bits();
  let i = 0x5fe6_eb85_1eb8_5400 - (i >> 1);
  let mut a = f64::from_bits(i);

  // ニュートン法の繰り返し
  // a = 1 / √x
  // a^2 = 1 / x
  // 1 / a^2 = x
  // f(a) := 1 / a^2 - x = 0
  // f'(a) = -2 / a^3
  // a_+ = a - f(a) / f'(a)
  //  = a + a^3 (1 / a^2 - x) / 2
  //  = a + (a - a^3 x) / 2
  //  = 3/2 a - a^3 x / 2
  //  = a (3 / 2 - a^2 x / 2)
  const THREE_HALF: f64 = 1.5;
  let half_x = 0.5 * x;
  for _ in 0..2 {
    a = a * (THREE_HALF - (half_x * a * a));
  }
  a
}
```

「えっなにこれ……, まぁその方が逆にパパっと使いやすいこともあるってことね?

「制約が付いた値を別の型として, その制約が維持されていることを保証するのもテクニックです

```rs
/// 十分に小さい `f64` の値を 0 として扱う
#[derive(Debug, Default, Clone, Copy, PartialOrd, PartialEq)]
struct FuzzyFloat(f64);

impl FuzzyFloat {
  fn new(x: f64) -> Self {
    Self(if x.abs() < 1e-6 { 0.0 } else { x })
  }

  fn as_f64(self) -> f64 { self.0 }
}
```

「あっ, なんかオブジェクト指向の本で似たようなことやらされた気がするぞ

「あぁ, 確かに OO で触れられることが多い考え方ですね


「それでは, 実際にパニックを起こしてみましょうか

「はい, うみこさん! どういうときにパニックすればいいんですか?

「あぁその説明がまだでした. 抑えるべきポイントはこんな感じでしょうか

1. 制約違反 - 守られているはずの制約が守られていないことを検知したとき
2. ありえない状況 - 論理的にありえないはずの状況に遭遇したとき
3. 未実装 - 意図的に未実装にしたか実装作業中でその処理を続行できない場合

`panic!`

「まず, `panic!` はシンプルに与えられたメッセージでパニックを起こします

```rs
panic!();

panic!("");

panic!("", x);
```

「なんだか `print!` みたーい

`assert!`

「そして, 制約の検証をする場合はもっぱらこっちです

```rs

```

「`assert_eq!` と `assert_ne!` もあるんだ, なんで分かれてるの?

「まぁいちいち比較して, `assert!` して, パニックのメッセージ用意するのは手間ですからね. `==` できるものならこれですぐ検証コードが書けます

「ほうほう, これは覚えといたほうがよさそう……. これいちいち `assert!` で検証してたらそのぶん遅くなったりとかしないの?

「あー, まぁそこの実行コストを削ってバグの検出機会を捨てるのかという判断は必要ですね. 一応, `debug_assert` というデバッグビルド時だけ有効になるやつもありますから

「ふんふん, 気になるときはそっち使えばいっか♪

`unreachable!`

「そのコードパスに分岐することがありえない場合は, `unreachable!` の出番ですね

「へー, こんなのあるんだぁ

```rs
fn foo(x: Option<i32>) {
 match x {
  Some(n) if n >= 0 => println!("Some(Non-negative)"),
  Some(n) if n <  0 => println!("Some(Negative)"),
  Some(_)           => unreachable!(), // コメントアウトしたらエラー
  None              => println!("None")
 }
}
```

`unimplemented!` `todo!`

「最後に未実装の場合は 2 種類が用意されています

「どっちも即座にパニックするだけじゃん. なにか違うの?

「一応, `unimplemented!` は意図的に未実装, `todo!` は作業中だから未実装という使い分けがあります

「ふーん, こっちはあんまり使うイメージ湧かないかな……


「ふー, エラー処理だけで結構盛り上がっちゃった

「かなり話し込んじゃいましたね……. パーキング代がちょっと嵩んでます


「Rust って関数型言語ってやつなんだっけ?

「そうですね. といっても手続き型言語やオブジェクト指向言語の毛色がかなり強いですが

「ほぇー, よーわからん. でも, クラスが無いから C++ での設計のやり方が通用しなくてどうしたものかと……

「そういえば, 私の知り合いに純粋関数型言語でいろいろ作ってるがいるんですよ

「へー, そんな人がいるんだ

「その人に関手とかモナドとか教わってみてもいいと思いません? 桜さん

「気になる気になる! ぜひ紹介してください

「はいこれ, 名刺です

「うわなにこのケバケバしい名刺……. えーと, 神谷 みお さん?

---

動画: Mikuro さいな
監修: かわえもん

参考文献:

- The Rust Programming Language a.k.a. "The Book" https://doc.rust-lang.org/book/
- Crate std - Rust https://doc.rust-lang.org/std/
- Fast Inverse Square Root — A Quake III Algorithm https://www.youtube.com/watch?v=p8u_k2LIZyo

---
