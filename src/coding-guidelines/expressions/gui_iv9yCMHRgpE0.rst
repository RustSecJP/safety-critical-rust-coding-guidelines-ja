.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

整数を無効なポインタに変換してはならない
=========================================

.. guideline:: 整数を無効なポインタに変換してはならない
   :id: gui_iv9yCMHRgpE0
   :category: <TODO>
   :status: draft
   :release: <TODO>
   :fls: fls_9wgldua1u8yt
   :decidability: undecidable
   :scope: system
   :tags: defect, undefined-behavior

   結果のポインタのアラインメントが正しくない場合、参照先の型の実体を指していない場合、
   または無効な表現である場合は、数値型の式をポインタに変換してはなりません。

   .. rationale::
      :id: rat_OhxKm751axKw
      :status: draft

      ポインタと整数の間の対応付けは、実行環境のアドレス構造と一致していなければなりません。
      たとえば、セグメント化されたメモリモデルを持つアーキテクチャでは、問題が起こることがあります。

   .. non_compliant_example::
      :id: non_compl_ex_CkytKjRQezfQ
      :status: draft

      この例は、すべてのプラットフォームで成り立つわけではない、アドレス空間のレイアウトについての仮定をしています。
      操作されたアドレスでは元のアドレス空間の一部が捨てられているかもしれず、フラグがアドレスの値に
      気づかれないまま干渉するかもしれません。ポインタが 64 ビットのプラットフォームでは、
      特に予期しない結果になることがあります。

      .. rust-example::

        #[allow(dead_code)]
        fn f1(flag: u32, ptr: * const u32) {
          /* ... */
          let mut rep = ptr as usize;
          rep = (rep & 0x7fffff) | ((flag as usize) << 23);
          let _p2 = rep as * const u32;
        }
        #
        # fn main() {}

   .. compliant_example::
      :id: compl_ex_oBoluiKSvREu
      :status: draft

      この適合する解決策では、構造体を使って、ポインタとフラグの値の両方を格納する場所を用意します。
      この解決策は、32 ビットより小さいものも大きいものも含め、ワードサイズの異なるマシンに移植でき、
      ポインタをどの整数型でも表現できない場合でも動作します。

      .. rust-example::

        #[allow(dead_code)]
        struct PtrFlag {
          pointer: * const u32,
          flag: u32
        }

        #[allow(dead_code)]
        fn f2(flag: u32, ptr: * const u32) {
          let _ptrflag = PtrFlag {
            pointer: ptr,
            flag: flag
          };
          /* ... */
        }
        #
        # fn main() {}
