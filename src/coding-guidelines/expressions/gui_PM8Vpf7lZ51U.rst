.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

整数をポインタに変換してはならない
==================================

.. guideline:: 整数をポインタに変換してはならない
   :id: gui_PM8Vpf7lZ51U
   :category: <TODO>
   :status: draft
   :release: <TODO>
   :fls: fls_59mpteeczzo
   :decidability: decidable
   :scope: module
   :tags: subset, undefined-behavior

   ``as`` 演算子を、左オペランドが数値型の式で、右オペランドが何らかのポインタ型である形で
   使ってはなりません。

   :std:`std::mem::transmute` を、``Src`` パラメータの引数が (浮動小数点型を含む) 何らかの数値型で、
   ``Dst`` パラメータの引数が何らかのポインタ型である形で使ってはなりません。

   .. rationale::
      :id: rat_YqhEiWTj9z6L
      :status: draft

      任意の算術式から作られたポインタは、無効なアドレスを指すことがあります。
      たとえば、有効なオブジェクトを指していないアドレス、誤った型のオブジェクトを指すアドレス、
      正しくアラインされていないアドレスなどです。そのようなポインタを使ってメモリにアクセスすると、
      未定義動作になります。

      また、``as`` 演算子は、元のオペランドのサイズがポインタのサイズと同じであることを検査しません。
      そのため、アドレスの計算がもともと異なるサイズのアドレス空間で行われていた場合には、
      予期しない結果になることがあります。

      ``as`` を使って概念上はヌルポインタを作れますが、
      :std:`core::ptr::null` や :std:`core::ptr::null_mut` という関数を使うほうが慣用的です。

   .. non_compliant_example::
      :id: non_compl_ex_0ydPk7VENSrA
      :status: draft

      ``as`` や ``transmute`` を使って算術的なアドレスの値からポインタを作ることは、
      どのような使い方でも不適合です。

      .. rust-example::
        :miri:

        #[allow(dead_code)]
        fn f1(x: u16, y: i32, z: u64, w: usize) {
          let _p1 = x as * const u32;  // 不適合
          let _p2 = y as * const u32;  // 不適合
          let _p3 = z as * const u32;  // 不適合
          let _p4 = w as * const u32;  // サイズは正しいが不適合

          let _f: f64 = 10.0;
          // let p5 = f as * const u32;  // 無効

          unsafe {
            // let p5: * const u32 = std::mem::transmute(x);  // 無効
            // let p6: * const u32 = std::mem::transmute(y);  // 無効

            #[allow(integer_to_ptr_transmutes)]
            let _p7: * const u32 = std::mem::transmute(z); // 不適合
            #[allow(integer_to_ptr_transmutes)]
            let _p8: * const u32 = std::mem::transmute(w); // 不適合

            let _p9: * const u32 = std::mem::transmute(_f); // 不適合であり、とても奇妙
          }
        }
        #
        # fn main() {}

   .. compliant_example::
      :id: compl_ex_oneKuF52yzrx
      :status: draft

      この操作の適合例はありません。
