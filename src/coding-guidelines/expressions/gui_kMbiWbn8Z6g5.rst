.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

0 で割らない
============

.. guideline:: 0 で割らない
   :id: gui_kMbiWbn8Z6g5
   :category: required
   :status: draft
   :release: latest
   :fls: fls_Q9dhNiICGIfr
   :decidability: undecidable
   :scope: system
   :tags: numerics, defect

   整数のゼロによる除算はパニックになります。
   これには、除算式 :cite:`gui_kMbiWbn8Z6g5:FLS-DIVISION-EXPR` と剰余式 :cite:`gui_kMbiWbn8Z6g5:FLS-REMAINDER-EXPR` の両方が含まれます。

   符号付き整数に対する除算式と剰余式では、算術オーバーフローも起こりえます。
   オーバーフローについては、ガイドライン `整数演算で算術オーバーフローが起こらないようにする` ですべて扱っています。

   この規則は、次のプリミティブな整数型 :cite:`gui_kMbiWbn8Z6g5:FLS-INTEGER-TYPES` に適用されます。

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

   この規則は、整数型 :cite:`gui_kMbiWbn8Z6g5:FLS-INTEGER-TYPES` 以外の型に対する :std:`core::ops::Div` トレイトの評価には適用されません。

   この規則は、`整数の除算で整数型を除数に使わない` をゆるくしたものです。
   その規則に適合するコードは、すべてこの規則にも適合します。

   .. rationale::
      :id: rat_h84NjY2tLSBW
      :status: draft

      整数のゼロによる除算はパニックになります。パニックはプロセスを終了させることがある異常なプログラムの状態であり、避けなければなりません。

   .. non_compliant_example::
      :id: non_compl_ex_LLs3vY8aGz0F
      :status: draft

      この不適合例は、除算と剰余演算のどちらでも、右オペランドがゼロのときにパニックします。

      .. rust-example::
          :compile_fail:

          fn main() {
              let x = 0;
              let _y = 5 / x; // パニックになる。
              let _z = 5 % x; // これもパニックになる。
          }

   .. compliant_example::
      :id: compl_ex_Ri9pP5Ch3kcc
      :status: draft

      `整数の除算で整数型を除数に使わない` の適合例は、この規則でも有効です。
      加えて、この適合例のように、ゼロの検査を手作業で行うこともできます。
      ただし、その不変条件に至る制御フローが複雑になるほど、
      プログラマにとっても静的解析ツールにとっても、それについて推論するのが難しくなっていきます。

      なお、符号なし整数では算術オーバーフローの検査は必要ありません。

      .. rust-example::

         fn main() {
             // 手作業でゼロを検査する
             let x = 0u32;
             let _y = if x != 0u32 {
                 5u32 / x
             } else {
                 0u32
             };

             let _z = if x != 0u32 {
                 5u32 % x
             } else {
                 0u32
             };
         }

   .. bibliography::
      :id: bib_kMbiWbn8Z6g5
      :status: draft

      .. list-table::
         :header-rows: 0
         :widths: auto
         :class: bibliography-table

         * - :bibentry:`gui_kMbiWbn8Z6g5:FLS-INTEGER-TYPES`
           - The Rust FLS. "Types and Traits - Integer Types." https://rust-lang.github.io/fls/types-and-traits.html#integer-types
         * - :bibentry:`gui_kMbiWbn8Z6g5:FLS-DIVISION-EXPR`
           - The Rust FLS. "Expressions - Syntax - DivisionExpression." https://rust-lang.github.io/fls/expressions.html#syntax_divisionexpression
         * - :bibentry:`gui_kMbiWbn8Z6g5:FLS-REMAINDER-EXPR`
           - The Rust FLS. "Expressions - Syntax - RemainderExpression." https://rust-lang.github.io/fls/expressions.html#syntax_remainderexpression
