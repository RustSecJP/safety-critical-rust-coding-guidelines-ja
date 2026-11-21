.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

論理的に異なる値を区別するために強い型を使用する
===================================================================

.. guideline:: 論理的に異なる値を区別するために強い型を使用する
   :id: gui_xztNdXA2oFNC
   :category: advisory
   :status: draft
   :release: 1.85.0;1.85.1
   :fls: fls_cokwseo3nnr
   :decidability: undecidable
   :scope: module
   :tags: types, safety, understandability

   論理的に異なる型を持つパラメータや変数は、型システムによって静的に区別できなければなりません。

   次のような場合には、ニュータイプ（例: ``struct Meters(u32);``）を使用してください。
   
   * 2 つ以上の量が、基になるプリミティブ表現は同じだが、論理的には異なる場合
   * それらを取り違えると意味上の誤りになる場合
   * 型安全性とカプセル化を向上させる必要がある場合
   * トレイトに基づく振る舞いを可能にする必要がある場合
   * 新しい不変条件を確立する必要がある場合
   
   .. rationale::
      :id: rat_kYiIiW8R2qD2
      :status: draft

      このルールにより、パラメータや変数は型システムを通じて意図を直接伝えるようになります。これは、プリミティブは同じでも意味が異なる値を誤って使ってしまうことを避けるためです。
      具体的には次のとおりです。

      * 論理的に異なる値が混ざるのを防ぎます。
        ``u32`` や ``u64`` のようなプリミティブ型は、長さ、カウンタ、タイムスタンプ、期間、ID、その他の値を表すことができます。
        異なる意味領域が取り違えられると、誤った計算につながります。
        意味が別々の型として表現されていれば、Rust の型システムはそのような誤りを防ぎます。
      * 静的な安全性を向上させます。
        静的に異なる型により、コンパイラが領域の区別を強制できます。
        パラメータを誤って入れ替えたり、間違った量を返したりすると、コンパイル時エラーになります。
      * 読みやすさと見つけやすさを向上させます。
        意図を表す名前（``Meters``、``Seconds``、``UserId``）により、コードが自己文書化されます。
        型シグネチャが読みやすく、理解しやすくなります。
      * 領域固有のトレイト実装を可能にします。
        静的に異なる型により、``Add``、``Mul``、または独自のトレイトを、領域のロジックに合った方法で実装できます。
        エイリアスではこれができません。エイリアスは別個の型ではないからです。
      * API の進化を支えます。
        静的に異なる型は強力な API 契約として機能し、基になる表現とは独立して進化させることができます。

   .. non_compliant_example::
      :id: non_compl_ex_PO5TyFsRTlWw
      :status: draft

      この不適合例はプリミティブ型を直接使っているため、``distance`` と ``time`` を取り違えるおそれがあります。
      呼び出し側が ``time`` を ``distance`` として渡すこと、またはその逆を妨げるものは何もありません。
      関数シグネチャだけからは、それぞれの型の単位がわかりません。
      誤りがあっても問題なくコンパイルされ、黙って誤った結果を生み出します。

       .. rust-example::

         fn travel(distance: u32, time: u32) -> u32 {
            distance / time
         }

         fn main() {
            let d = 100;
            let t = 10;
            let _result = travel(t, d);  // コンパイルは通るが、意味的に誤っている
         }

   .. non_compliant_example::
      :id: non_compl_ex_PO5TyFsRTlWv
      :status: draft

      この不適合例は、別個の型の代わりにエイリアスを使っています。
      エイリアスは新しい型を作らないので、コンパイラは ``Meters`` と ``Seconds`` の区別を強制できません。
     
      エイリアスではこれができません。エイリアスは別個の型ではないからです。
      この不適合例はプリミティブ型を直接使っているため、``distance`` と ``time`` を取り違えるおそれがあります。
      呼び出し側が ``time`` を ``distance`` として渡すこと、またはその逆を妨げるものは何もありません。
      関数シグネチャだけからは、それぞれの型の単位がわかりません。
      誤りがあっても問題なくコンパイルされ、黙って誤った結果を生み出します。

       .. rust-example::

         type Meters = u32;
         type Seconds = u32;
         type MetersPerSecond = u32;

         fn travel(distance: Meters, time: Seconds) -> MetersPerSecond {
            distance / time
         }

         fn main() {
            let d: Meters = 100;
            let t: Seconds = 10;
            let _result = travel(t, d);  // コンパイルは通るが、意味的に誤っている
         }

   .. compliant_example::
      :id: compl_ex_WTe7GoPu5Ez1
      :status: draft

      この適合例は、ニュータイプを使って ``Meters``、``Seconds``、``MetersPerSecond`` を別個の型にしています。
      コンパイラが正しい使い方を強制するので、パラメータを誤って入れ替えることを防げます。
      関数シグネチャは、各パラメータと戻り値の意図した意味を明確に伝えます。

       .. rust-example::

         use std::ops::Div;

         #[derive(Debug, Clone, Copy)]
         struct Meters(u32);

         #[derive(Debug, Clone, Copy)]
         struct Seconds(u32);

         #[derive(Debug, Clone, Copy)]
         struct MetersPerSecond(u32);

         impl Div<Seconds> for Meters {
             type Output = MetersPerSecond;

             fn div(self, rhs: Seconds) -> Self::Output {
                  MetersPerSecond(self.0 / rhs.0)
             }
          }

          fn main() {
              let d = Meters(100);
              let t = Seconds(10);
              let result = d / t;  // すっきりしていて型安全！
              println!("Speed: {} m/s", result.0);  // 内部の値にアクセスする
          }
