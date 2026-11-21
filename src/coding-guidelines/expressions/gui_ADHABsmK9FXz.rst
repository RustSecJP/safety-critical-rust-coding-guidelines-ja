.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

'as' 演算子を数値のオペランドに使うべきではない
================================================

.. guideline:: 'as' 演算子を数値のオペランドに使うべきではない
   :id: gui_ADHABsmK9FXz
   :category: advisory
   :status: draft
   :release: <TODO>
   :fls: fls_otaxe9okhdr1
   :decidability: decidable
   :scope: module
   :tags: subset, reduce-human-error

   二項演算子 ``as`` は、次のものと一緒に使うべきではありません。

   * 数値型。サポートされるすべての整数型、浮動小数点型、マシン依存の算術型を含みます。または
   * ``bool``。または
   * ``char``

   これは、右オペランドとしても、左オペランドの型としても同様です。

   **例外:** 右オペランドが ``usize`` で、左オペランドが生ポインタ型の式であるときは、
   ``as`` を使ってもかまいません。

   .. rationale::
      :id: rat_v56bjjcveLxQ
      :status: draft

      数値型の間で ``as`` が行う変換はすべて定義されていますが、``as`` は値を変換先の型に収まるように強制します。
      そのため、値を切り詰めたり、丸めたり、等しくない最も近い値にしたりする必要がある場合には、
      予期しないデータの損失が起こることがあります。

      損失のない変換もありますが、対称でない変換もあります。定義された損失のある振る舞いに頼ったり、
      精度の損失を覚悟したりする代わりに、``Into`` や ``From``、``TryInto`` や ``TryFrom`` を使えば、
      コードで意図を伝えられます。つまり、どの変換が元の値を完全に保つことを意図しており、
      どの変換が失敗しうることを意図しているかを示せます。後者は const 関数からは使えません。
      これは、const 関数では失敗しうる変換を避けるべきだということを示しています。

      ポインタからアドレスへのキャストは値を失いませんが、変換先の型がアドレスの値を保持できるほど
      大きくない限り、切り詰められます。``usize`` 型は、この目的に十分な幅があることが保証されています。

      ポインタからアドレスへのキャストは対称ではありません。結果のポインタは、有効なオブジェクトを指していないかもしれず、
      正しい型のオブジェクトを指していないかもしれず、正しくアラインされていないかもしれないからです。
      この方向の変換が必要な場合は、:std:`std::mem::transmute` を使うと、
      ``unsafe`` な操作を行う意図を伝えられます。

   .. non_compliant_example::
      :id: non_compl_ex_hzGUYoMnK59w
      :status: draft

      ここで使われている ``as`` は、値の範囲を変えたり、精度を失ったりすることがあります。
      そうならない場合でも、正しい振る舞いを強制するものは何もありません。また、損失のある変換を許すつもりなのか、
      有効な変換だけを想定しているのかも伝わりません。

      .. rust-example::

         #[allow(dead_code)]
         fn f1(x: u16, y: i32, _z: u64, w: u8) {
           let _a = w as char;           // 不適合
           let _b = y as u32;            // 不適合 - 値の範囲が変わり、負の値が変換される
           let _c = x as i64;            // 不適合 - .into() を使える

           let d = y as f32;            // 不適合 - 損失がある
           let e = d as f64;            // 不適合 - .into() を使える
           let _f = e as f32;            // 不適合 - 損失がある

           let _g = e as i64;            // 不適合 - オブジェクトのサイズにかかわらず損失がある

           let b: u32 = 0;
           let p1: * const u32 = &b;
           let _a1 = p1 as usize;        // 例外により適合
           let _a2 = p1 as u16;          // 不適合 - アドレスの範囲を失うことがある
           let _a3 = p1 as u64;          // 不適合 - 意図を示すには usize を使う

           let a1 = p1 as usize;
           let _p2 = a1 as * const u32;  // 不適合 - transmute を使うほうがよい
           let a2 = p1 as u16;
           let _p3 = a2 as * const u32;  // 不適合 (しかも、おそらく有効なアドレスの範囲にない)
         }
         #
         # fn main() {}

   .. compliant_example::
      :id: compl_ex_uilHTIOgxD37
      :status: draft

      正確な値を保つことが保証されている有効な変換は、``into()`` や ``from()`` を使うと、
      よりよく伝えられます。
      値を失うおそれがあり、それがエラーとなるような有効な変換では、``try_into`` や ``try_from`` を使うと、
      そのことを伝え、エラーの検査も含められます。
      それ以外の形の変換では、``transmute`` のほうが意図をよりよく伝えられる場合があります。

      .. rust-example::
         :miri:

         use std::convert::TryInto;

         #[allow(dead_code)]
         fn f2(x: u16, y: i32, _z: u64, w: u8) {
           let _a: char            = w.into();
           let _b: Result <u32, _> = y.try_into(); // 範囲外ならエラーを生成する
           let _c: i64             = x.into();

           let d = f32::from(x);  // u16 は範囲内だが、u32 はそうではない
           let _e = f64::from(d);
           // let f = f32::from(e); // From は存在しない

           // let g = ...            // From は存在しない

           let h: u32 = 0;
           let p1: * const u32 = &h;
           let a1 = p1 as usize;     // (適合)

           unsafe {
             let _a2: usize = std::mem::transmute(p1);  // OK
             let _a3: u64   = std::mem::transmute(p1);  // OK、サイズが検査される
             // let a3: u16   = std::mem::transmute(p1);  // 無効、サイズが異なる

             #[allow(integer_to_ptr_transmutes)]
             let _p2: * const u32 = std::mem::transmute(a1); // OK
             #[allow(integer_to_ptr_transmutes)]
             let _p3: * const u32 = std::mem::transmute(a1); // OK
           }

           unsafe {
             // まったく別のことをする。
             // 整数の値を変換するのではなく、z のビットを
             // 倍精度オブジェクトの IEEE ビットパターンとして再解釈する
             #[allow(unnecessary_transmutes)]
             let _f1: f64 = std::mem::transmute(_z);
           }
         }
         #
         # fn main() {}
