.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

マクロ定義の中の名前には完全修飾パスを使わなければならない
==========================================================

.. guideline:: マクロ定義の中の名前には完全修飾パスを使わなければならない
   :id: gui_SJMrWDYZ0dN4
   :category: required
   :status: draft
   :release: 1.85.0;1.85.1
   :fls: fls_7kb6ltajgiou
   :decidability: decidable
   :scope: module
   :tags: reduce-human-error

   マクロの定義の中の各名前には、グローバルパスか、$crate を前に付けたパスを使わなければなりません。

   .. rationale::
      :id: rat_VRNXaxmW1l2s
      :status: draft

      マクロの中で実体を相対的に参照するパスを使うと、そのパスは、マクロが使われる場所によって変わりうる
      パス解決の結果に左右されます。マクロを使うときに、実体を参照するための意図したパスがシャドーイングされ、
      予期しない振る舞いにつながることがあります。これにより、マクロが場所によって異なる振る舞いをする理由や、
      マクロの中の実体がどこに解決されるのかについて、開発者が混乱するおそれがあります。

   .. non_compliant_example::
      :id: non_compl_ex_m2XR1ihTbCQS
      :status: draft

      次のマクロは、グローバルでないパスを使ってベクタの実体を参照する例です。マクロが使われる場所によっては、
      意図したものとは異なる `Vec` が使われることがあります。これが使われるスコープで、マクロの定義時には
      存在しない構造体 `Vec` が定義されていれば、マクロのユーザはマクロの中でそれを使うつもりなのかもしれません。

      .. rust-example::

        #[macro_export]
        macro_rules! my_vec {
            ( $( $x:expr ),* ) => {
                {
                    let mut temp_vec = Vec::new(); // グローバルでないパス
                    $(
                        temp_vec.push($x);
                    )*
                    temp_vec
                }
            };
        }

        fn main() {
            let _ = my_vec![1, 2, 3];
        }

   .. compliant_example::
      :id: compl_ex_xyaShvxL9JAM
      :status: draft

      次のマクロは、グローバルパスを使って Vec を参照します。マクロを使うスコープで `Vec` という別の構造体が
      定義されていても、このマクロは曖昧さなく標準ライブラリの `Vec` を使います。

      .. rust-example::

        #[macro_export]
        macro_rules! my_vec_global {
            ( $( $x:expr ),* ) => {
                {
                    let mut temp_vec = ::std::vec::Vec::new(); // グローバルパス
                    $(
                        temp_vec.push($x);
                    )*
                    temp_vec
                }
            };
        }

        fn main() {
            let _ = my_vec_global![1, 2, 3];
        }
