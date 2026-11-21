.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

マクロを呼び出してはならない
============================

.. guideline:: マクロを呼び出してはならない
   :id: gui_a1mHfjgKk4Xr
   :category: mandatory
   :status: draft
   :release: todo
   :fls: fls_vnvt40pa48n8
   :decidability: decidable
   :scope: system
   :tags: reduce-human-error

   ここにガイドラインの説明を書きます。

   .. rationale::
      :id: rat_62mSorNF05kD
      :status: draft

      このガイドラインが重要である理由の説明。

   .. non_compliant_example::
      :id: non_compl_ex_hP5KLhqQfDcd
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
      :id: compl_ex_ti7GWHCOhUvT
      :status: draft

      コード例の説明。

      .. rust-example::

        #[allow(dead_code)]
        fn example_function() {
            // 適合する実装
        }
        #
        # fn main() {}
