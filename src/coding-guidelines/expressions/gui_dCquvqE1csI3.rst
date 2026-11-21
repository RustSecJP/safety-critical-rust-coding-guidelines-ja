.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

整数演算で算術オーバーフローが起こらないようにする
====================================================================

.. guideline:: 整数演算で算術オーバーフローが起こらないようにする
    :id: gui_dCquvqE1csI3
    :category: required
    :status: draft
    :release: 1.0 - latest
    :fls: fls_oFIRXBPXu6Zv
    :decidability: decidable
    :scope: system
    :tags: security, performance, numerics

    符号付き整数型と符号なし整数型の両方で、算術オーバーフロー :cite:`gui_dCquvqE1csI3:FLS-ARITHMETIC-OVERFLOW` をなくしてください。
    デバッグモードとリリースモードの両方で同じ振る舞いになるように、ラップアラウンドの振る舞いはすべて明示的に指定しなければなりません。

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

    .. rationale::
        :id: rat_LvrS1jTCXEOk
        :status: draft

        実行時のパニックや予期しないラップアラウンドの振る舞いを避けるために、算術オーバーフローをなくしてください。
        算術オーバーフローは、デバッグモードではパニックしますが、リリースモードではラップアラウンドするため、振る舞いが一貫しません。
        これらの振る舞いが意図したものである場合は、明示的に :std:`std::num::Wrapping` や
        :std:`std::num::Saturating` のセマンティクスを使ってください。
        範囲の検査を使えば、算術オーバーフローの可能性をなくせます。

    .. non_compliant_example::
        :id: non_compl_ex_cCh2RQUXeH0N
        :status: draft

        この不適合なコード例では、符号付きのオペランド ``si_a`` と ``si_b`` の加算で算術オーバーフローが起こりえます。

        .. rust-example::

            #[allow(dead_code)]
            fn add(si_a: i32, si_b: i32) {
              let _sum: i32 = si_a + si_b;
              // ...
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BgUHiRB4kc4b_1
        :status: draft

        この適合する解決策では、符号付き 32 ビット整数の最大の範囲に基づいて、
        加算の演算で算術オーバーフローが起こりえないことを保証します。
        オーバーフローの検出には、
        :std:`u32::overflowing_add`、
        :std:`u32::overflowing_sub`、
        :std:`u32::overflowing_mul`
        などの関数も使えます。
        これらの関数を呼び出すコードでは、通常、想定される入力の範囲に基づいて、
        取りうる値の範囲をさらに制限することになるでしょう。

        .. rust-example::

            # #[derive(Debug)]
            #[allow(dead_code)]
            enum ArithmeticError { Overflow, DivisionByZero }
            #[allow(dead_code)]
            fn add(si_a: i32, si_b: i32) -> Result<i32, ArithmeticError> {
                if (si_b > 0 && si_a > i32::MAX - si_b)
                    || (si_b < 0 && si_a < i32::MIN - si_b)
                {
                    Err(ArithmeticError::Overflow)
                } else {
                    Ok(si_a + si_b)
                }
            }

            #[allow(dead_code)]
            fn sub(si_a: i32, si_b: i32) -> Result<i32, ArithmeticError> {
                if (si_b < 0 && si_a > i32::MAX + si_b)
                    || (si_b > 0 && si_a < i32::MIN + si_b)
                {
                    Err(ArithmeticError::Overflow)
                } else {
                    Ok(si_a - si_b)
                }
            }

            #[allow(dead_code)]
            fn mul(si_a: i32, si_b: i32) -> Result<i32, ArithmeticError> {
                if si_a == 0 || si_b == 0 {
                    return Ok(0);
                }

                // 乗算を行う前にオーバーフローを検出する
                if (si_a == -1 && si_b == i32::MIN) || (si_b == -1 && si_a == i32::MIN) {
                    Err(ArithmeticError::Overflow)
                } else if (si_a > 0 && (si_b > i32::MAX / si_a || si_b < i32::MIN / si_a))
                    || (si_a < 0 && (si_b > i32::MIN / si_a || si_b < i32::MAX / si_a))
                {
                    Err(ArithmeticError::Overflow)
                } else {
                    Ok(si_a * si_b)
                }
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BgUHiRB4kc4c
        :status: draft

        この適合例では、手作業での境界の検査の代わりに、安全な検査付き加算を使います。
        複雑な算術式が必要な場合、検査付きの関数を使うと可読性が下がることがあります。

        .. rust-example::

            # #[derive(Debug)]
            #[allow(dead_code)]
            enum ArithmeticError { Overflow, DivisionByZero }

            #[allow(dead_code)]
            fn add(si_a: i32, si_b: i32) -> Result<i32, ArithmeticError> {
                si_a.checked_add(si_b).ok_or(ArithmeticError::Overflow)
            }

            #[allow(dead_code)]
            fn sub(a: i32, b: i32) -> Result<i32, ArithmeticError> {
                a.checked_sub(b).ok_or(ArithmeticError::Overflow)
            }

            #[allow(dead_code)]
            fn mul(a: i32, b: i32) -> Result<i32, ArithmeticError> {
                a.checked_mul(b).ok_or(ArithmeticError::Overflow)
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BgUHiRB4kc4b
        :status: draft

        ラップアラウンドの振る舞いは明示的に要求しなければなりません。この適合例では、ラップアラウンドする関数を使います。

        .. rust-example::

            #[allow(dead_code)]
            fn add(a: i32, b: i32) -> i32 {
                a.wrapping_add(b)
            }

            #[allow(dead_code)]
            fn sub(a: i32, b: i32) -> i32 {
                a.wrapping_sub(b)
            }

            #[allow(dead_code)]
            fn mul(a: i32, b: i32) -> i32 {
                a.wrapping_mul(b)
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BhUHiRB4kc4b
        :status: draft

        この適合する解決策のように、:std:`std::num::Wrapping` 型を使ってラップアラウンドの振る舞いを実現することもできます。
        :std:`std::num::Wrapping` 型は ``core::num`` モジュールにある ``struct`` で、
        内部の型 ``T`` (整数か ``usize/isize`` でなければなりません) に対して、2 の補数によるラップアラウンド算術を明示的に有効にします。
        :std:`std::num::Wrapping` 型は、すべてのビルドモードでラップアラウンドの振る舞いを一貫して強制する方法を提供します。
        これは、暗号やハッシュ関数の実装のように、ラップアラウンド算術が意図した振る舞いである特定の場面で役に立ちます。

        .. rust-example::

            use std::num::Wrapping;

            fn add(si_a: Wrapping<i32>, si_b: Wrapping<i32>) -> Wrapping<i32> {
                si_a + si_b
            }

            #[allow(dead_code)]
            fn sub(si_a: Wrapping<i32>, si_b: Wrapping<i32>) -> Wrapping<i32> {
                si_a - si_b
            }

            #[allow(dead_code)]
            fn mul(si_a: Wrapping<i32>, si_b: Wrapping<i32>) -> Wrapping<i32> {
                si_a * si_b
            }

            fn main() {
                let si_a = Wrapping(i32::MAX);
                let si_b = Wrapping(i32::MAX);
                println!("{} + {} = {}", si_a, si_b, add(si_a, si_b))
            }

    .. compliant_example::
        :id: compl_ex_BgUHiSB4kc4b
        :status: draft

        飽和セマンティクスとは、ラップアラウンドしたりエラーになったりする代わりに、
        整数型の有効な範囲から外れる結果を次のように切り詰めることです。

        - 結果が最大値より大きくなる場合は、最大値に。または
        - 結果が最小値より小さくなる場合は、最小値に。

        飽和セマンティクスは、整数演算で算術オーバーフローが起こらないことを保証するので、常にこの規則に適合します。
        この適合する解決策では、飽和する関数を使って、いくつかの基本的な算術演算に飽和セマンティクスを持たせる方法を示します。

        .. rust-example::

            #[allow(dead_code)]
            fn add(a: i32, b: i32) -> i32 {
                a.saturating_add(b)
            }

            #[allow(dead_code)]
            fn sub(a: i32, b: i32) -> i32 {
                a.saturating_sub(b)
            }

            #[allow(dead_code)]
            fn mul(a: i32, b: i32) -> i32 {
                a.saturating_mul(b)
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BgUHiSB4kd4b
        :status: draft

        :std:`std::num::Saturating` は Rust の ``core`` ライブラリにあるラッパ型で、包んだ値に対する算術演算を、ラップアラウンドやパニック、オーバーフローの代わりに飽和算術で行わせます。
        :std:`std::num::Saturating` は、すべての算術を飽和させなければならないコードの部分やデータ型があるときに役に立ちます。
        この適合する解決策では、:std:`std::num::Saturating` 型を使って、飽和セマンティクスで基本的な整数演算を行う関数をいくつか定義します。

        .. rust-example::

            use std::num::Saturating;

            fn add(si_a: Saturating<i32>, si_b: Saturating<i32>) -> Saturating<i32> {
                si_a + si_b
            }

            #[allow(dead_code)]
            fn sub(si_a: Saturating<i32>, si_b: Saturating<i32>) -> Saturating<i32> {
                si_a - si_b
            }

            #[allow(dead_code)]
            fn mul(si_a: Saturating<i32>, si_b: Saturating<i32>) -> Saturating<i32> {
                si_a * si_b
            }

            fn main() {
                let si_a = Saturating(i32::MAX);
                let si_b = Saturating(i32::MAX);
                println!("{} + {} = {}", si_a, si_b, add(si_a, si_b))
            }

    .. non_compliant_example::
        :id: non_compl_ex_cCh2RQUXeH0O
        :status: draft

        この不適合なコード例は、ゼロ除算のエラーは防ぎますが、算術オーバーフローは防ぎません。

        .. rust-example::

            # #[derive(Debug)]
            #[allow(dead_code)]
            enum DivError { DivisionByZero, Overflow }

            #[allow(dead_code)]
            fn div(s_a: i64, s_b: i64) -> Result<i64, DivError> {
                if s_b == 0 {
                    Err(DivError::DivisionByZero)
                } else {
                    Ok(s_a / s_b)
                }
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_BgUHiRB4kc4d
        :status: draft

        この適合する解決策では、ゼロ除算のエラーと算術オーバーフローの両方の可能性をなくします。

        .. rust-example::

            # #[derive(Debug)]
            #[allow(dead_code)]
            enum DivError { DivisionByZero, Overflow }

            #[allow(dead_code)]
            fn div(s_a: i64, s_b: i64) -> Result<i64, DivError> {
                if s_b == 0 {
                    Err(DivError::DivisionByZero)
                } else if s_a == i64::MIN && s_b == -1 {
                    Err(DivError::Overflow)
                } else {
                    Ok(s_a / s_b)
                }
            }
            #
            # fn main() {}

    .. bibliography::
        :id: bib_dCquvqE1csI3
        :status: draft

        .. list-table::
           :header-rows: 0
           :widths: auto
           :class: bibliography-table

           * - :bibentry:`gui_dCquvqE1csI3:FLS-ARITHMETIC-OVERFLOW`
             - The Rust FLS. "Expressions - Arithmetic Overflow." https://rust-lang.github.io/fls/expressions.html#arithmetic-overflow
