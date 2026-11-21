.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

関数の代わりにマクロを使うべきではない
======================================

.. guideline:: 関数の代わりにマクロを使うべきではない
   :id: gui_2jjWUoF1teOY
   :category: mandatory
   :status: draft
   :release: todo
   :fls: fls_xa7lp0zg1ol2
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   常にマクロより関数を優先すべきです。ただし、可変長引数のインターフェイス、コンパイル時のコード生成、カスタム derive やアトリビュートマクロによる構文の拡張など、関数では提供できない不可欠な機能をマクロが提供する場合は例外です。

   |

   .. rationale::
      :id: rat_M9bp23ctkzQ7
      :status: draft

      コンパイラはマクロの展開とその呼び出し箇所の両方を報告しますが、マクロの中から生じる診断は、通常の関数や型の定義から生じる診断よりも解釈が難しいことがあります。複雑なマクロや深くネストしたマクロは、意図をわかりにくくし、静的解析を妨げることがあります。その結果、コードレビューで誤解したりエラーを見落としたりするリスクが高まります。


      **デバッグの複雑さ**

      - エラーはソース上の位置ではなく展開されたコードを指すので、コンパイル時のエラーを元のマクロ呼び出しまでたどるのが難しくなります。

      **最適化**

      - マクロは、関数のほうがうまく働くコンパイラの最適化を妨げることがあります。
      - マクロは ``#[inline(always)]`` の関数のように振る舞うので、コードの肥大化につながることがあります。
      - マクロはコンパイラのインライン化のヒューリスティクスの恩恵を受けられず、インライン化が有益な場合をコンパイラが判断する選択的なインライン化を活かせません。

      **関数が提供するもの**

      - 明確な型シグネチャ。
      - 予測可能な振る舞い。
      - 適切なスタックトレース。
      - 一貫した最適化の機会。


   .. non_compliant_example::
      :id: non_compl_ex_TZgk2vG42t2r
      :status: draft

      単純な関数で十分なところでマクロを使うと、隠れた変更につながります。

      .. rust-example::

        macro_rules! increment_and_double {
            ($x:expr) => {
                {
                    $x += 1; // 変更が暗黙的
                    $x * 2
                }
            };
        }

        fn main() {
            let mut num = 5;
            let result = increment_and_double!(num);
            println!("Result: {}, Num: {}", result, num);
            // Result: 12, Num: 6
        }

      この例では、マクロを呼び出すと、値のインクリメントと値を返すことが一度に行われます。しかも、引数を変更することがその「シグネチャ」のどこにも明示されていません。その結果、num は裏で変更され、読み手を驚かせたり、デバッグを難しくしたりすることがあります。


   .. compliant_example::
      :id: compl_ex_iPTgzrvO7qr3
      :status: draft

      同じ機能を、明示的な借用を使う関数として実装したものです。

      .. rust-example::

        fn increment_and_double(x: &mut i32) -> i32 {
            *x += 1; // 変更が明示的
            *x * 2
        }

        fn main() {
            let mut num = 5;
            let result = increment_and_double(&mut num);
            println!("Result: {}, Num: {}", result, num);
            // Result: 12, Num: 6
        }

      関数版では、変更と借用がシグネチャに明示されるので、可読性、安全性、デバッグのしやすさが向上します。
