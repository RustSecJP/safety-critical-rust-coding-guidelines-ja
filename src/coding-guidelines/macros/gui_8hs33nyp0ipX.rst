.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

マクロの完全な衛生性を確保しなければならない
============================================

.. guideline:: マクロの完全な衛生性を確保しなければならない
   :id: gui_8hs33nyp0ipX
   :category: mandatory
   :status: draft
   :release: todo
   :fls: fls_xlfo7di0gsqz
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   ここにガイドラインの説明を書きます。

   .. rationale::
      :id: rat_e9iS187skbHH
      :status: draft

      このガイドラインが重要である理由の説明。

   .. non_compliant_example::
      :id: non_compl_ex_lRt4LBen6Lkc
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
      :id: compl_ex_GLP05s9c1g8N
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
