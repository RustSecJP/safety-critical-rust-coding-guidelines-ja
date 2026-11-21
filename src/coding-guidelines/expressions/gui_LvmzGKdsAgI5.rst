.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

範囲外のシフトを避ける
======================

.. guideline:: 範囲外のシフトを避ける
    :id: gui_LvmzGKdsAgI5
    :category: mandatory
    :status: draft
    :release: 1.0.0-latest
    :fls: fls_caxn774ij8lk
    :decidability: undecidable
    :scope: module
    :tags: numerics, surprising-behavior, defect

    左シフト式と右シフト式 :cite:`gui_LvmzGKdsAgI5:FLS-BIT-EXPRESSIONS` で、負の位置数、
    または左オペランドの幅以上の値だけシフトすることを、
    このガイドラインでは *範囲外のシフト* と定義します。
    Rust FLS は、この振る舞いを誤って算術オーバーフローと説明しています :cite:`gui_LvmzGKdsAgI5:FLS-ISSUE-632`。

    両方のオペランドの型が整数型である場合、
    左シフト式 ``lhs << rhs`` は、左オペランド ``lhs`` のビットを
    右オペランド ``rhs`` で指定された位置数だけ左にシフトした値に評価されます。
    空いたビットはゼロで埋められます。
    式 ``lhs << rhs`` は、:math:`\mathrm{lhs} \times 2^{\mathrm{rhs}}` を
    左オペランドの型にキャストしたものに評価されます。
    右オペランドの値が負であるか、左オペランドの幅以上である場合、
    その演算は範囲外のシフトになります。

    両方のオペランドの型が整数型である場合、
    右シフト式 ``lhs >> rhs`` は、左オペランド ``lhs`` のビットを
    右オペランド ``rhs`` で指定された位置数だけ右にシフトした値に評価されます。
    左オペランドの型が符号付き整数型で、その値が負である場合、
    空いたビットは 1 で埋められます。
    それ以外の場合、空いたビットはゼロで埋められます。
    式 ``lhs >> rhs`` は、:math:`\mathrm{lhs} / 2^{\mathrm{rhs}}` を
    左オペランドの型にキャストしたものに評価されます。
    右オペランドの値が負であるか、
    左オペランドの幅以上である場合、
    その演算は範囲外のシフトになります。

    この規則は次のプリミティブ型に適用されます。

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

    トレイトを実装すれば、どんな型でも ``<<`` や ``>>`` をサポートできます。

    .. rust-example::

       use core::ops::Shl;
       #[allow(dead_code)]
       struct MyType;

       impl Shl<u32> for MyType {
           type Output = MyType;
           fn shl(self, _rhs: u32) -> Self::Output { MyType }
       }
       #
       # fn main() {}

    実装は自分で制御できるので、右オペランドには (整数に限らず) どんな型でも選べます。

    この規則は、
    `式を負のビット数、またはオペランドに存在するビット数以上だけシフトしない` を、
    ゆるくしたうえで決定不能にしたものです。
    その規則に適合するコードは、すべてこの規則にも適合します。

    この規則は、CERT C コーディングスタンダードの規則 :cite:`gui_LvmzGKdsAgI5:CERT-C-INT34` に基づいています。

    .. rationale::
        :id: rat_tVkDl6gOqz25
        :status: draft

        左シフト式と右シフト式では、範囲外のシフトを避けてください。
        負の値だけシフトすることや、左オペランドの幅以上の値だけシフトすることは、
        意味をなさない式であり、通常は論理エラーが起きていることを示します。

    .. non_compliant_example::
        :id: non_compl_ex_KLiDMsCesLx7
        :status: draft

        この不適合例では、負の値 (-1) だけシフトし、さらに左オペランドに存在するビット数以上 (40) だけシフトしています。

        .. rust-example::
            :should_panic:

            fn main() {
                let bits : u32 = 61;
                let shifts = vec![-1, 4, 40];

                for sh in shifts {
                    println!("{bits} << {sh} = {:?}", bits << sh);
                }
            }

    .. compliant_example::
        :id: compl_ex_Ux1WqHbGKV73
        :status: draft

        この適合例では、``sh`` の値を検査して、右オペランドの値が負であるか、
        左オペランドの幅以上であることを確かめます。

        .. rust-example::

            fn main() {
                let bits: u32 = 61;
                let shifts = vec![-1, 0, 4, 40];

                for sh in shifts {
                    if sh >= 0 && sh < 32 {
                        println!("{bits} << {sh} = {}", bits << sh);
                    }
                 }
            }

    .. compliant_example::
        :id: compl_ex_Ux1WqHbGKV74
        :status: draft

        この不適合における ``bits.overflowing_shl(sh)`` の呼び出しは、``bits`` を ``sh`` ビットだけ左にシフトします。
        シフトした self と、シフト量がビット数以上だったかどうかを示す真偽値とのタプルを返します。
        シフト量が大きすぎる場合、値は (N-1) でマスクされ (N はビット数)、この値を使ってシフトが行われます。

          .. rust-example::

             fn safe_shl(bits: u32, shift: u32) -> u32 {
                 let (result, overflowed) = bits.overflowing_shl(shift);
                 if overflowed {
                     0
                 } else {
                     result
                 }
             }

             fn main() {
                 let bits: u32 = 61;
                 let shifts = vec![4, 40];

                 for sh in shifts {
                     let result = safe_shl(bits, sh);
                     println!("{bits} << {sh} = {result}");
                 }
             }

    .. bibliography::
        :id: bib_LvmzGKdsAgI5
        :status: draft

        .. list-table::
           :header-rows: 0
           :widths: auto
           :class: bibliography-table

           * - :bibentry:`gui_LvmzGKdsAgI5:CERT-C-INT34`
             - SEI CERT C Coding Standard. "INT34-C. Do not shift an expression by a negative number of bits." https://wiki.sei.cmu.edu/confluence/x/ItcxBQ
           * - :bibentry:`gui_LvmzGKdsAgI5:FLS-BIT-EXPRESSIONS`
             - The Rust FLS. "Expressions - Bit Expressions." https://rust-lang.github.io/fls/expressions.html#bit-expressions
           * - :bibentry:`gui_LvmzGKdsAgI5:FLS-ISSUE-632`
             - The Rust FLS Repository. "Issue 632." https://github.com/rust-lang/fls/issues/632
