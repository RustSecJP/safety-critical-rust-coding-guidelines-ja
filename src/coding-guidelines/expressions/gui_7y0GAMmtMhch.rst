.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

整数の除算で整数型を除数に使わない
==================================

.. guideline:: 整数の除算で整数型を除数に使わない
   :id: gui_7y0GAMmtMhch
   :category: advisory
   :status: draft
   :release: latest
   :fls: fls_Q9dhNiICGIfr
   :decidability: decidable
   :scope: module
   :tags: numerics, subset

   除算式 :cite:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR` または剰余式 :cite:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR` で、左オペランドも整数型であるときは、
   整数型 :cite:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES` の右オペランドを与えてはいけません。

    この規則は次のプリミティブな整数型に適用されます。

    * ``i8``
    * ``i16``
    * ``i32``
    * ``i64``
    * ``i128``
    * ``u8``
    * ``u16``
    * ``u32``
    * ``u64``
    * ``u128``
    * ``usize``
    * ``isize``

   .. rationale::
      :id: rat_vLFlPWSCHRje
      :status: draft

      整数の除算と整数の剰余演算は、どちらも右オペランドの値がゼロのときにパニックします。
      ゼロによる除算は数学では定義されていません。矛盾を生じ、その結果として割り当てられる一貫した値が存在しないからです。

   .. non_compliant_example::
      :id: non_compl_ex_0XeioBrgfh5z
      :status: draft

      この不適合例では、右オペランドがゼロなので、除算と剰余演算のどちらも評価されるとパニックします。

      .. rust-example::
          :compile_fail:

          fn main() {
              let x = 0;
              let _y = 5 / x; // この行はパニックする。
              let _z = 5 % x; // この行もパニックするだろう。
          }

   .. compliant_example::
      :id: compl_ex_k1CD6xoZxhXb
      :status: draft

      検査付き除算を使うと、ゼロによる除算が起こるのを防げます。
      プログラマはそのうえで、返された :std:`std::option::Option` を処理できます。
      検査付きの除算と剰余を使うことは、符号付き整数の場合に特に重要です。
      符号付き整数では、表現できる最小の値を -1 で割ったときに算術オーバーフローも起こりうるからです。

      .. rust-example::

         fn main() {
             // 検査付き除算の API を使う
             let _y = match 5i32.checked_div(0) {
                 None => 0,
                 Some(r) => r,
             };

             // 検査付き剰余の API を使う
             let _z = match 5i32.checked_rem(0) {
                 None => 0,
                 Some(r) => r,
             };
         }

   .. compliant_example::
      :id: compl_ex_k1CD6xoZxhXc
      :status: draft

      この適合例では、:std:`std::num::NonZero` を使って除数を作ります。
      :std:`std::num::NonZero` はプリミティブな整数型のラッパで、中の値が決してゼロにならないことを保証します。
      :std:`std::num::NonZero::new` は、ゼロでないことがわかっている値を表す新しい束縛を作ります。
      これにより、その値を扱う関数は、入力としてゼロが与えられないと正しく仮定できます。

      なお、この適合例では、表現できる最小の値を -1 で割ったときに起こる算術オーバーフローの検査は不要です。
      除算式の結果が符号なし整数型だからです。

      .. rust-example::
         :version: 1.79

         use std::num::NonZero;

         fn main() {
             let x = 0u32;
             if let Some(divisor) = NonZero::<u32>::new(x) {
                 let _result = 5u32 / divisor;
             }
         }

   .. bibliography::
      :id: bib_7y0GAMmtMhch
      :status: draft

      .. list-table::
         :header-rows: 0
         :widths: auto
         :class: bibliography-table

         * - :bibentry:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
           - The Rust FLS. "Types and Traits - Integer Types." https://rust-lang.github.io/fls/types-and-traits.html#integer-types
         * - :bibentry:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR`
           - The Rust FLS. "Expressions - Syntax - DivisionExpression." https://rust-lang.github.io/fls/expressions.html#syntax_divisionexpression
         * - :bibentry:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR`
           - The Rust FLS. "Expressions - Syntax - RemainderExpression." https://rust-lang.github.io/fls/expressions.html#syntax_remainderexpression
