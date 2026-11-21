.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

関数風マクロを使ってはならない
==============================

.. guideline:: 関数風マクロを使ってはならない
   :id: gui_WJlWqgIxmE8P
   :category: mandatory
   :status: draft
   :release: todo
   :fls: fls_utd3zqczix
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   ここにガイドラインの説明を書きます。

   .. rationale::
      :id: rat_C8RRidiVzhRj
      :status: draft

      このガイドラインが重要である理由の説明。

   .. non_compliant_example::
      :id: non_compl_ex_TjRiRkmBY6wG
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
      :id: compl_ex_AEKEOYhBWPMl
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
