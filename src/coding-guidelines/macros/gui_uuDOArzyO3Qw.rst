.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

マクロを展開するコードを書いてはならない
========================================

.. guideline:: マクロを展開するコードを書いてはならない
   :id: gui_uuDOArzyO3Qw
   :category: mandatory
   :status: draft
   :release: todo
   :fls: fls_wjldgtio5o75
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   ここにガイドラインの説明を書きます。

   .. rationale::
      :id: rat_dNgSvC0SZ3JJ
      :status: draft

      このガイドラインが重要である理由の説明。

   .. non_compliant_example::
      :id: non_compl_ex_g9j8shyGM2Rh
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 不適合な実装
        }
        #
        # fn main() {}

   .. compliant_example::
      :id: compl_ex_cFPg6y7upNdl
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
