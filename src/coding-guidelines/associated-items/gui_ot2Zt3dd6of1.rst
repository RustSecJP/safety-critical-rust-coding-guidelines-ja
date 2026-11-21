.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

再帰関数は許可されません
==================================

.. guideline:: 再帰関数は許可されません
    :id: gui_ot2Zt3dd6of1 
    :category: required
    :status: draft
    :release: 1.3.0-latest
    :fls: fls_927nfm5mkbsp
    :decidability: undecidable
    :scope: system
    :tags: stack-overflow

    いかなる関数も、直接的にも間接的にも自分自身を呼び出してはなりません

    .. rationale:: 
        :id: rat_gvoKeVSKK8fD 
        :status: draft

        再帰関数はスタックオーバーフローを容易に引き起こします。その結果、例外が発生したり、場合によっては未定義動作になったりします（典型的には一部の組み込みシステム）。Rust コンパイラは末尾呼び出し最適化 :cite:`gui_ot2Zt3dd6of1:WIKI-TAIL-CALL` をサポートしていますが、この最適化は保証されておらず、具体的な実装や関数の構造に依存します。Rust コンパイラで末尾呼び出し最適化を保証するためのオープンな RFC :cite:`gui_ot2Zt3dd6of1:PROPOSED-RFC-EXPLICIT-TAIL-CALLS` がありますが、この機能はまだ安定化されていません。末尾呼び出し最適化が保証され安定化されるまでは、起こりうるスタックオーバーフローを防ぎ、プログラムの信頼性を確保するために、開発者は再帰関数の使用を避けるべきです。

    .. non_compliant_example::
        :id: non_compl_ex_MxqhjfkStJJy 
        :status: draft

        以下の関数 ``concat_strings`` は適合していません。自分自身を呼び出しており、入力として与えられたデータの深さによっては、スタックオーバーフローの例外や未定義動作を発生させる可能性があるからです。

        .. rust-example::

            // 文字列または `MyEnum` のリストを表す再帰的な列挙型
            #[allow(dead_code)]
            enum MyEnum {
                Str(String),
                List(Vec<MyEnum>),
            }

            // 再帰を使って、`MyEnum` の入れ子構造から文字列を連結します。
            #[allow(dead_code)]
            fn concat_strings(input: &[MyEnum]) -> String {
                let mut result = String::new();
                for item in input {
                    match item {
                        MyEnum::Str(s) => result.push_str(s),
                        MyEnum::List(list) => result.push_str(&concat_strings(list)),
                    }
                }
                result
            }
            #
            # fn main() {}

    .. compliant_example::
        :id: compl_ex_9pK3h65rfceO 
        :status: draft

        以下のコードは、再帰の代わりに反復を使って同じ機能を実装しています。``stack`` 変数は、ループの各ステップで処理のコンテキストを保持するために使われます。この方法では、メモリ使用量を明示的に制御できます。入力の構造やサイズのためにスタックがあらかじめ定めた上限を超えて大きくなった場合、この関数はスタックオーバーフローやメモリ不足の例外を起こす危険を冒すのではなく、エラーを返します。これにより、リソースが限られた環境で、より予測しやすく堅牢な動作が保証されます。

        .. rust-example::

            // 文字列または `MyEnum` のリストを表す再帰的な列挙型
            #[allow(dead_code)]
            enum MyEnum {
                Str(String),
                List(Vec<MyEnum>),
            }

            /// 再帰を使わずに、`MyEnum` の入れ子構造から文字列を連結します。
            /// スタックのサイズが `MAX_STACK_SIZE` を超えた場合はエラーを返します。
            #[allow(dead_code)]
            fn concat_strings_non_recursive(input: &[MyEnum]) -> Result<String, &'static str> {
                const MAX_STACK_SIZE: usize = 1000;
                let mut result = String::new();
                let mut stack = Vec::new();

                // すべての要素をスタックに追加する
                stack.extend(input.iter());

                while let Some(item) = stack.pop() {
                    match item {
                        MyEnum::Str(s) => result.insert_str(0, s),
                        MyEnum::List(list) => {
                            // リストの要素をスタックに追加する
                            for sub_item in list.iter() {
                                stack.push(sub_item);
                                if stack.len() > MAX_STACK_SIZE {
                                    return Err("Too big structure");
                                }
                            }
                        }
                    }
                }
                Ok(result)
            }
            #
            # fn main() {}

    .. bibliography::
        :id: bib_ot2Zt3dd6of1
        :status: draft

        .. list-table::
           :header-rows: 0
           :widths: auto
           :class: bibliography-table

           * - :bibentry:`gui_ot2Zt3dd6of1:WIKI-TAIL-CALL`
             - Wikipedia. "Tail call." https://en.wikipedia.org/wiki/Tail_call
           * - :bibentry:`gui_ot2Zt3dd6of1:PROPOSED-RFC-EXPLICIT-TAIL-CALLS`
             - Philipp Görz. "Open RFC - explicit_tail_calls." https://github.com/phi-go/rfcs/blob/guaranteed-tco/text/0000-explicit-tail-calls.md
