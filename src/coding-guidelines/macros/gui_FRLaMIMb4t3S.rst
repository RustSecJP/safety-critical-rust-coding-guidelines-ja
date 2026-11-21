.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

マクロ展開の中に unsafe ブロックを隠さない
==========================================

.. guideline:: マクロ展開の中に unsafe ブロックを隠さない
   :id: gui_FRLaMIMb4t3S
   :category: required
   :status: draft
   :release: todo
   :fls: fls_4vjbkm4ceymk
   :decidability: todo
   :scope: todo
   :tags: reduce-human-error

   ここにガイドラインの説明を書きます。

   .. rationale::
      :id: rat_WJubG7KuUDLW
      :status: draft

      このガイドラインが重要である理由の説明。

   .. non_compliant_example::
      :id: non_compl_ex_AyFnP0lJLHxi
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
      :id: compl_ex_pO5gP1aj2v4F
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
