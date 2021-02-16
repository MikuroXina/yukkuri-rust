「さて, そろそろ帰りますか……

ねねから電話がかかってくる

「はい, うみこですが

「もしもし, 今だいじょうぶですかー？

「いいですよ, なんですか

「C++ の std::move とかいうやつがわかんないんだけどー

*OP*

「それはムーブセマンティクスを実現するためのものです

「メーヴェ？ロマンティクス？

「ムーブセマンティクスです. 値をこちらからあちらへ移すことです

「まずコピーのデメリットから説明しますね

「コピーコンストラクタを使ってこのようにコピーをすると, それぞれの作成と破棄が行われるのはわかってますよね?

「あーうん. クラス型のオブジェクトのコピーは, コピーコンストラクタとコピー代入演算子でやるんだよね

```cpp
using std::puts;
class Hoge final {
  int *ptr;

public:
  Hoge() : ptr(new int) {
    puts("Hoge created");
  }
  ~Hoge() { 
    puts("Hoge dropped");
    delete ptr;
  }

  Hoge(Hoge const &src) : ptr(new int{*src.ptr}) {
    puts("Hoge copied");
  }
  Hoge &operator=(Hoge const &) = default;
};
Hoge a; // a の作成
Hoge b{a}; // a をコピーして b の作成
// しかしこれ以降 a は使わない
// b の破棄
// a の破棄
```

「しかし, コピーした後からはコピー元を使わないのであれば, ちょっとメモリが無駄になります

「んー, 無駄なのー?

「既に確保したポインタを使い回して, `delete` する回数も 1 回に減らすんですよ

```cpp
Hoge a; // a の作成
Hoge b /* <-???-- */ a; // a のポインタを b に移す
// b の破棄
// a は破棄しない
```

「えーでも, そんなことできるの?

「ムーブコンストラクタを定義することで, できるんです

```cpp
using std::puts;
class Hoge final {
  int *ptr;

public:
  Hoge() : ptr(new int) {
    puts("Hoge created");
  }
  ~Hoge() { 
    if (!ptr) { return; }
    puts("Hoge dropped");
    delete ptr;
  }

  Hoge(Hoge &&src) : ptr(src.ptr) {
    src.ptr = nullptr;
    puts("Hoge moved");
  }
  Hoge &operator=(Hoge &&) = default;
};
Hoge a;
Hoge b{static_cast<Hoge &&>(a)};
```

「ん? この `Hoge &&` ってなんだろ

「それが rvalue 参照型ですね. 関数にこの型の値を渡した後から, その値は使えなくなります

「`b` のとこのキャストで rvalue 参照にしてるってわけ?

「そうですね. こうすることで `a` は使えなくなります

「あー, ムーブコンストラクタの中で `ptr` を `nullptr` にしてるのはなんで?

「先ほど『使えなくなる』と言いましたが, そういうことにしているだけで実際には触れてしまいます

「えー, そうなの?

「だからムーブ元オブジェクトの後始末をムーブコンストラクタの中でやります

「後始末なの?

「デストラクタのところで, `ptr` が `nullptr` だったら解放処理を飛ばしているでしょう?

「あー, ムーブ元は解放しなくていいってことか

「そうですね, おかげで実際の `new` と `delete` はちゃんと 1 回ずつになります

*アイキャッチ*

「このように, std::vector はムーブ構築によって効率化できます. しかし原理的には, スタック領域上のバイト列だけをコピーするということですね

「勝手にコピーされるんじゃなくて、代入がムーブになる言語があったらいいのになー

「Rust という言語なら, ムーブがデフォになっていますね

「仕事で使ったりはしていませんが, システムプログラミング言語としては人気のようです

「ググってみたら, Stack Overflow の愛されてる言語ランキングで 4 年連続 1 位なんだって！

「C++ と似た部分が多いので, ねねさんみたいな物好きなら趣味でやっても楽しいと思いますよ

「Rust ってやつやってみる！

「検索の際には同名ゲームが出てくることがあるのでお気をつけくださいね

*ED*

ねねっち on SNS

「Rustマジで何しようとしてもコンパイルエラー出る

「Rust楽しい、、、

「Rust、言う事聞いてくれたときは嬉しい
そもそもコンパイルが通った時点で喜ぶ

「Rust、超難しい言語なんだろうなと勝手に思ってたけど、実際少し難しいけどコンパイラがめちゃくちゃ優しくて救いの手を差し伸べてくれるので、多分みんなやれば出来ると思う

（何か悪い影響を与えてしまったような気がする…）

---

動画: Mikuro さいな
監修: かわえもん

参考文献:

- Rust (プログラミング言語) - Wikipedia https://ja.wikipedia.org/wiki/Rust_(%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E8%A8%80%E8%AA%9E)
- Rust Programming Language https://www.rust-lang.org/
- The Rust Programming Language a.k.a. "The Book" https://doc.rust-lang.org/book/

---