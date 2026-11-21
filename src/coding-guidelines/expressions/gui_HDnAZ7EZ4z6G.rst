.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

as によるアンダースコアへのポインタのキャストを避ける
======================================================

.. guideline:: as によるアンダースコアへのポインタのキャストを避ける
   :id: gui_HDnAZ7EZ4z6G
   :category: required
   :status: draft
   :release: <TODO>
   :fls: fls_1qhsun1vyarz
   :decidability: decidable
   :scope: module
   :tags: readability, reduce-human-error

   ``var as Type`` や :std:`core::mem::transmute` で明示的にポインタをキャストするとき、コードは Rust の型推論に頼ってはいけません。
   代わりに、``as`` 式や :std:`core::mem::transmute` の呼び出し式で、変換先の完全な型を明示的に指定してください。

   .. rationale::
      :id: rat_h8LdJQ1MNKu9
      :status: draft

      生ポインタ型の間の ``var as Type`` によるキャストや :std:`core::mem::transmute` は、変換先のポインタ型が細いポインタである限り、一般に有効であり、コンパイラによって検査されません。
      変換先の具体的なポインタ型を指定しないと、コンパイラは周囲の文脈からその型を推論します。そのため、周囲の型が変わったことでキャストが意図せず変わり、意味的に無効なポインタのキャストになることがあります。

      生ポインタには、手作業で把握しておかなければならないさまざまな不変条件があります。
      このような場面で具体的な型を指定すると、こうした潜在的な問題の一部をコンパイラがユーザに代わって検出できるようになります。

   .. non_compliant_example::
      :id: non_compl_ex_V37Pl103aUW4
      :status: draft

      次のコードは、生ポインタのキャストの具体的な型を型推論に任せています。そのため、``with_base`` の関数シグネチャを変更すると、コンパイルエラーにならないまま、``non_compliant_example`` の関数本体の型に影響が及ぶことがあります。

      .. rust-example::
         :miri:

         #[repr(C)]
         #[allow(dead_code)]
         struct Base {
            position: (u32, u32)
         }

         #[repr(C)]
         #[allow(dead_code)]
         struct Extended {
            base: Base,
            scale: f32
         }

         #[allow(dead_code)]
         fn non_compliant_example(extended: &Extended) {
            let extended = extended as *const _;
            with_base(unsafe { &*(extended as *const _) })
         }

         fn with_base(_: &Base) {}
         #
         # fn main() {}

   .. compliant_example::
      :id: compl_ex_W08ckDrkOhkt
      :status: draft

      ポインタのキャストで変換先の具体的な型を指定しています。そのため、``with_base`` の関数シグネチャが変更されると、コンパイルエラーになります。

      .. rust-example::
         :miri:

         #[repr(C)]
         #[allow(dead_code)]
         struct Base {
            position: (u32, u32)
         }

         #[repr(C)]
         #[allow(dead_code)]
         struct Extended {
            base: Base,
            scale: f32
         }

         #[allow(dead_code)]
         fn compliant_example(extended: &Extended) {
            let extended = extended as *const Extended;
            with_base(unsafe { &*(extended as *const Base) })
         }

         fn with_base(_: &Base) {}
         #
         # fn main() {}
