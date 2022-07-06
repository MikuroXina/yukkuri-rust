---
登場人物:
    - 桜ねね
    - 鳴海ツバメ
    - 阿波根うみこ
---

「げ, なんかわからんがうまく動かない」

「サメサメ♪サメサメ〜♪」

「(聞きたいけどうみこさんは今打ち合わせ中で相談できないな……、でもねねっちには頼りたくない……)」

「んーどしたの〜?」

「げっ. い, いやなんでもないよー」

「なるっちも作業終わって暇なんでしょ?」

「あ, あはは. そんな感じ〜」

「なに画面隠してんの〜? なんかやましいことでもあったりー」

「……」

「……」

「えいっ, こちょこちょ〜」

「うわっ, あはっ, ちょ, ちょっとやめてぇ〜」

*OP*

「えー, わかんないなんてなるっちらしくないじゃん」

「う〜, ……そういう感じになるから見せたくないんだよ」

「まぁまぁ, 一緒に考えよーよ. それで, 何書いてる?」

「読み込んだ効果音の音声データを, 同時再生できるよう複製しておくってやつ」

```cpp
// 見やすさのためにメンバ関数の呼び出しは展開してある
auto &audio_buffer = audio_channel->buffer;
auto target_size = audio_channel->size_map[target_index];
auto target_ptr = audio_buffer.data() + audio_channel->shift_map[target_index];

audio_buffer.resize(audio_buffer.size() + target_size);
std::copy(target_ptr, target_ptr + target, audio_buffer.begin());
```

「チャンネルのバッファに, 前の区間のデータを挿入してるのか」

「でも, できたと思ったら稀に消えたりしてうまく動かないんだよねー」

「なるほどー. あ, この現象を再現するコード思いついたかも」

「えっ, まじ?」

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = { 1, 4, 2, 3, 5 };

    for (auto const &num : nums) {
        nums.push_back(num);
        std::cout << num << ", ";
    }
    // 実行結果の例:
    // 0, 0, 6058000, 0, 5,
}
```

「ほら, こうすると意味不明な値が出てくるよ」

「うわなにこの出力, 気持ち悪っ」

「それでこのコードはね, `nums` から可変と不変の借用を同時にしてるからうまく動かないんだよ」

「借用? あたし聞いたことないかも」

「ほらここ, 範囲 for を展開すると……」

```cpp
std::vector<int> nums = { 1, 4, 2, 3, 5 };

for (auto it = nums.cbegin(); it != nums.cend(); **it) {
    auto const &num = *it;
    nums.push_back(num);
    std::cout << num << ", ";
}
```

「`it` は `nums` から借りた `const` のイテレータじゃん?」

「はい, 変更する必要ないですからね」

「でも, この後に `push_back` で `nums` を更に借りてるんだよ. しかも, 変更するために!」

「まずいんですか?」

「すごくまずいよ! まだ `it` が借りてる最中なんだもん. そのイテレータの指す先を書き換えたりしたら……!」

「あっなるほど, `it` に入っているイテレータが無効になってるんだねぇ〜」

「Rust ならコンパイラがこういう借用のルールをチェックしてくれるよ!」

「急にぶっこんできたな……ホントに Rust 好きだよね, ねねっち」

「むー, 絶対気に入ると思うんだけど」

*アイキャッチ*

「それで, 借用のルールってどういうものなの?」

「まず不変の借用は何個でもしていいぞ」

「読むだけならいくら共有しても問題ないはずだから, そりゃそうだよね」

「でも可変借用は 1 個までで, 不変借用とは同時に存在できない」

「まぁさっきみたいに, 他の借用が指す先を書き換えちゃったら困るもんね」

| OK/NG |  `&` | `&mut` |
| :---: | ---: | -----: |
|  OK   |    0 |      0 |
|  OK   |    3 |      0 |
|  OK   |    0 |      1 |
|       |      |        |
|  NG   |    1 |      1 |
|  NG   |    4 |      1 |
|  NG   |    0 |      2 |

「表にまとめるとこんな感じかな♪」

「ところでさっきのプログラムを Rust で書くとどうなるの?」

```rs
fn main() {
    let mut nums = vec![1, 4, 2, 3, 5];

    for &num in &nums {
        nums.push(num);
        print!("{num}, ");
    }
}
```

「……まぁ一応こうやって書けるけど」

```
error[E0502]: cannot borrow `nums` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:9
  |
4 |     for &num in &nums {
  |                 -----
  |                 |
  |                 immutable borrow occurs here
  |                 immutable borrow later used here
5 |         nums.push(num);
  |         ^^^^^^^^^^^^^^ mutable borrow occurs here
```

「コンパイルエラーになるんだ」

「これすごいでしょ! 丁寧に説明出してくれるし. `rustc --explain E0502` でエラーの詳しい説明も見られるよ」

> A variable already borrowed as immutable was borrowed as mutable.
>
> Erroneous code example:
>
> ```rs
> fn bar(x: &mut i32) {}
> fn foo(a: &mut i32) {
>     let y = &a; // a is borrowed as immutable.
>     bar(a); // error: cannot borrow `*a` as mutable because `a` is also borrowed
>             //        as immutable
>     println!("{}", y);
> }
> ```
>
> To fix this error, ...

「うんうん. ところで, さっきのバグの原因はわかったけどこれどうやって直そう……」

「う, それは私もわからんにゃ……」

*アイキャッチ*

「この借用ルール, バグを減らすのに意外と役立ちそうだね」

「借用ルールってのは, ただバグを減らすだけじゃないんだよ! 例えば参照を 2 つ受け取るこのコード」

```cpp
// a も b も 1 になる?
int hoge(int &a, int &b) {
    a = 1;
    b = 2;
    b = a;
    return a;
}
```

「ふむ, `b = 2` の代入の意味がないのでは?」

「いや, コンパイラはそういう最適化ができないよん. なぜなら同じ変数への参照かもしれないから」

```cpp
int a = 0;
hoge(a, a);
/*
    a = 1;
    a = 2;
    a = a;
    return a;
    だから a は 2
*/
```

「あー, もしかして可変借用のルール?」

「そう! Rust なら可変借用は必ず 1 個までだから同じ変数を指すことは無いの. こういうコードを書いたとしても」

```rs
fn hoge(a: &mut i32, b: &mut i32) -> i32 {
    *a = 1;
    *b = 2;
    *b = *a;
    *a
}
```

「こういうコードに最適化できるんだよね」

```rs
fn hoge(a: &mut i32, b: &mut i32) -> i32 {
    *a = 1;
    *b = 1;
    1
}
```

「ほへー. ちょっと調べてみるか……こうすれば C でも同じことできそう?」

```c
int hoge(int *restrict a, int *restrict b) {
    *a = 1;
    *b = 2;
    *b = a;
    return *a;
}
```

「へーそんなのあるんだ. ……でも Rust はコンパイラがチェックしてくれるから」

「あぁ, たしかにそれは強いね」

*アイキャッチ*

「そういえば前にうみこさんからこんな人を紹介してもらってさ」

「なんか独特なセンスの名刺だね……」

「フリーのフルスタックエンジニアのすごい人らしいよ」

「うみこさんが紹介するってことは確かにすごそう……」

「それでね, 関数型言語について詳しいし別分野だから参考になるかもってことで教えてもらえることになったの♪」

「良いじゃんか」

「駅前の小脇で会おうって話になったんだけど」

「小脇って何?」

「コワーキングスペースの略」

「いや癖がすごいな」

「どう, 一緒に来る?」

「うーん, どうしよっかな. 後でチャットするね」

うみこが後ろからブースに帰ってくる

「スーパープログラマたるもの, いろんな技術を身に着けないとね♪」

「前から思ってたけど, スーパープログラマってなんなの?」

「そりゃあ, うみこさんみたいな……」

「ばーん, 油断してましたね」

「うわわ, ってうみこさん!」「おかえりなさーい」

「音声処理のパフォーマンス改善のやつ, できてますか?」

「はい, こんな感じです. どうですかっ」

「あれ, これ排他制御してないじゃないですか. 元々は同期処理だったので問題なかったんですが, 今は負荷がかかるとバグりますよ」

「えっ, うっそまじか」

「できてないんかい!」

「う〜ん, 貸し借りって大変なんだね…」

---

動画: Mikuroさいな
監修: かわえもん

参考文献:

- The Rust Programming Language a.k.a. "The Book" https://doc.rust-lang.org/book/
- Crate std - Rust https://doc.rust-lang.org/std/

---
