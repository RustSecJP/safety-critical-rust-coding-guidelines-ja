.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

手続き的マクロを使うべきではない
================================

.. guideline:: 手続き的マクロを使うべきではない
   :id: gui_66FSqzD55VRZ
   :category: advisory
   :status: draft
   :release: 1.85.0;1.85.1
   :fls: fls_wn1i6hzg2ff7
   :decidability: decidable
   :scope: crate
   :tags: readability, reduce-human-error

   マクロは、手続き的な構文よりも
   宣言的な構文を優先して表現すべきです。

   .. rationale::
      :id: rat_AmCavSymv3Ev
      :status: draft

      手続き的マクロは純粋な転写に限定されず、任意の Rust コードを含められます。
      そのため、理解するのが難しくなることがあり、意図どおりに動作することを簡単には証明できません。
      手続き的マクロは任意の副作用を持てるので、コンパイラの資源を使い果たしたり、
      採用したコードのユーザに脆弱性をさらしたりすることがあります。

   .. non_compliant_example::
      :id: non_compl_ex_pJhVZW6a1HP9
      :status: draft

      (手続き的マクロを使った単純な展開の例)

      .. rust-example::

        // TODO
        fn main() {}

   .. compliant_example::
      :id: compl_ex_4VFyucETB7C3
      :status: draft

      (同じ単純な展開を宣言的マクロで行う例)

      .. rust-example::

        // TODO
        fn main() {}
