.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

アトリビュートマクロを使ってはならない
======================================

.. guideline:: アトリビュートマクロを使ってはならない
   :id: gui_13XWp3mb0g2P
   :category: required
   :status: draft
   :release: todo
   :fls: fls_4vjbkm4ceymk
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   アトリビュートマクロを宣言してはならず、呼び出してもなりません。
   ソースコードを拡張するだけの、より強力でないマクロを使ってください。

   .. rationale::
      :id: rat_X8uCF5yx7Mpo
      :status: draft

      アトリビュートマクロは、アイテムを丸ごと書き換えたり、その他の予期しない方法で書き換えたりできます。これは混乱を招き、エラーを持ち込むことがあります。

   .. non_compliant_example::
      :id: non_compl_ex_eW374waRPbeL
      :status: draft

      ``#[test]`` アトリビュートマクロは、関数をテストハーネスのエントリポイントに変換します。

      .. rust-example::
          :no_run:

          #[test]  // 不適合: アトリビュートマクロがアイテムを書き換える
          fn example_test() {
              assert!(true);
          }
          #
          # fn main() {}

   .. compliant_example::
      :id: compl_ex_Mg8ePOgbGJeW
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
