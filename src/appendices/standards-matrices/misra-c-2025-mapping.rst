MISRA C 2025 と Rust の相互参照
=======================================

以下の表は、MISRA C 2025 のガイドラインと、それらが Rust に当てはまるかどうかの相互参照を示します。
1 つ目の表は Rust 全般に当てはまるガイドラインを扱い、2 つ目の表は ``unsafe`` なコードがある場合に
追加で当てはまるガイドラインを扱います。3 つ目の表は Rust に当てはまらないガイドラインの一覧です。

この評価の出典は `MISRA C:2025 Addendum 6`_ にあります。


表 1 – Rust 全般に当てはまるガイドライン（安全な Rust、unsafe なコードがない場合）
--------------------------------------------------------------------------------------

.. list-table::
   :header-rows: 1
   :widths: auto

   * - ガイドライン
     - カテゴリ
     - セーフティクリティカル Rust ルール
     - カテゴリ
     - コメント
   * - **ディレクティブ**
     -
     -
     -
     -
   * - D.1.1
     - Required
     - 
     -
     -
   * - D.1.2
     - Advisory
     -
     -
     - 実験的なフィーチャや不安定なフィーチャに適用し、完全な文書化を強制することを意図している
   * - D.2.1
     - Required
     -
     -
     -
   * - D.3.1
     - Required
     -
     -
     -
   * - D.4.1
     - Required
     -
     -
     - 多くの場合パニックの形をとる
   * - D.4.4
     - Advisory
     -
     -
     - 条件付きコンパイルは cfg アトリビュートで提供される
   * - D.4.5
     - Advisory
     -
     -
     - 「曖昧さ」はプロジェクトが判断する
   * - D.4.7
     - Required
     -
     -
     - 帯域内のエラー値よりも Option や Result などを優先する
   * - D.4.9
     - Advisory
     -
     -
     -
   * - D.4.11
     - Required
     -
     -
     -
   * - D.4.12
     - Required
     -
     -
     -
   * - D.4.13
     - Advisory
     -
     -
     - Rust の多くの API は、型システムを使って順序を強制する
   * - D.4.14
     - Required
     -
     -
     -
   * - D.4.15
     - Required
     -
     -
     - Rust は IEEE-754 を実装している
   * - D.5.2
     - Required
     -
     -
     -
   * - D.5.3
     - Required
     -
     -
     -
   * - **ルール**
     -
     -
     -
     -
   * - R.1.3
     - Required
     -
     -
     -
   * - R.1.5
     - Required
     -
     -
     - 非推奨の API に適用される
   * - R.2.1
     - Required
     -
     -
     -
   * - R.2.2
     - Required
     -
     -
     -
   * - R.2.3
     - Advisory
     -
     -
     -
   * - R.2.5
     - Advisory
     -
     -
     -
   * - R.2.6
     - Advisory
     -
     -
     -
   * - R.2.7
     - Advisory
     -
     -
     -
   * - R.2.8
     - Advisory
     -
     -
     -
   * - R.3.1
     - Required
     -
     -
     - 入れ子のコメントは完全にサポートされている
   * - R.5.2
     - Required
     -
     -
     - 文字数の制限はないが、制限を適用することはできる。名前空間がある
   * - R.5.3
     - Required
     - :need:`gui_SJMrWDYZ0dN4`
     - Required
     - マクロ名にも適用される
   * - R.5.6
     - Required
     -
     -
     - きちんとしたモジュールシステムにより、予期しない名前の衝突はずっと起こりにくい
   * - R.5.8
     - Required
     -
     -
     - きちんとしたモジュールシステムにより、予期しない名前の衝突はずっと起こりにくい
   * - R.5.9
     - Advisory
     -
     -
     -
   * - R.7.1
     - Required
     -
     -
     - Rust の 8 進数には 10 進数とは異なる接頭辞がある
   * - R.7.2
     - Required
     -
     -
     - デフォルトではエラーだが、有効にできる。なお、接尾辞によってサイズも明示される
   * - R.8.7
     - Advisory
     -
     -
     - 1 つのモジュールでしか参照されない要素は pub と宣言すべきでない
   * - R.8.9
     - Advisory
     -
     -
     -
   * - R.8.13
     - Advisory
     -
     -
     - 必要でない限り ``mut`` は避けるべき
   * - R.9.1
     - Mandatory
     -
     -
     - rustc によって強制されるが、unsafe によって回避できる
   * - R.9.4
     - Required
     -
     -
     - rustc によって強制される
   * - R.11.3
     - Required
     -
     -
     -
   * - R.11.4
     - Advisory
     - :need:`gui_PM8Vpf7lZ51U`
     - <TODO>
     -
   * - R.11.11
     - Advisory
     -
     -
     - rustc によって強制される
   * - R.12.1
     - Advisory
     -
     -
     -
   * - R.13.1
     - Required
     -
     -
     - Rust では評価順序が厳密に決まっている
   * - R.13.5
     - Required
     -
     -
     -
   * - R.14.3
     - Required
     -
     -
     -
   * - R.14.4
     - Required
     -
     -
     - rustc によって強制される
   * - R.15.4
     - Advisory
     -
     -
     -
   * - R.15.5
     - Advisory
     -
     -
     -
   * - R.15.7
     - Required
     -
     -
     -
   * - R.17.2
     - Required
     - :need:`gui_ot2Zt3dd6of1`
     - Required
     -
   * - R.17.7
     - Required
     -
     -
     - ``must_use`` はこれが重要な箇所を示すのに役立つが、適用可能性には影響しない
   * - R.17.8
     - Advisory
     -
     -
     - パラメータを ``mut`` と宣言しない限り、うっかりこれを行うことはできない
   * - R.17.11
     - Advisory
     -
     -
     - 返らない関数を、値の型を返すように宣言できる
   * - R.18.3
     - Required
     -
     -
     -
   * - R.18.5
     - Advisory
     -
     -
     -
   * - R.19.2
     - Advisory
     - :need:`gui_0cuTYG8RVYjg`
     - Required
     -
   * - R.19.3
     - Required
     -
     -
     -
   * - R.21.25
     - Required
     -
     -
     -
   * - R.22.13
     - Required
     -
     -
     -
   * - R.22.18
     - Required
     -
     -
     -
   * - R.22.19
     - Required
     -
     -
     -


表 2 – unsafe なコードがある場合に Rust に当てはまるガイドライン
-----------------------------------------------------------------------

表 1 のルールに加えて、扱う必要がある追加のガイドラインは次のとおりです。

.. list-table::
   :header-rows: 1
   :widths: auto

   * - ガイドライン
     - カテゴリ
     - セーフティクリティカル Rust ルール
     - カテゴリ
     - コメント
   * - **ディレクティブ**
     -
     -
     -
     -
   * - D.4.2
     - Advisory
     -
     -
     -
   * - D.4.3
     - Required
     -
     -
     -
   * - D.5.1
     - Required
     -
     -
     - 安全な Rust の型がすべて競合しないわけではない
   * - **ルール**
     -
     -
     -
     -
   * - R.1.1
     - Required
     -
     -
     -
   * - R.5.1
     - Required
     -
     -
     - extern "C" 以外では文字数の制限はないが、プロジェクトで制限を設定できる
   * - R.5.5
     - Required
     -
     -
     - マクロと関数は異なる構文を使う
   * - R.5.10
     - Required
     -
     -
     - 一部の場合にのみ可能。以前の Rule 21.2
   * - R.8.3
     - Required
     -
     -
     - extern 宣言は C の宣言と互換性のある型を持たなければならない
   * - R.8.5
     - Required
     -
     -
     - extern "C" の宣言に影響することがある
   * - R.8.6
     - Required
     -
     -
     - extern "C" の宣言に影響することがある
   * - R.8.15
     - Required
     -
     -
     - extern "C" の宣言に影響することがある
   * - R.8.17
     - Advisory
     -
     -
     - アラインメントはオブジェクトではなく型に適用される
   * - R.9.7
     - Mandatory
     -
     -
     -
   * - R.10.5
     - Advisory
     - :need:`gui_ADHABsmK9FXz`
     - Advisory
     - 安全な ``as`` と unsafe な ``transmute`` の操作の両方を含む
   * - R.10.8
     - Required
     - :need:`gui_HDnAZ7EZ4z6G`
     - Required
     - 安全な ``as`` と unsafe な ``transmute`` の操作の両方を含む
   * - R.11.1
     - Required
     -
     -
     - 安全な ``as`` と unsafe な ``transmute`` の操作の両方を含む
   * - R.11.2
     - Required
     -
     -
     -
   * - R.11.5
     - Advisory
     -
     -
     -
   * - R.11.6
     - Required
     - :need:`gui_PM8Vpf7lZ51U`
     - <TODO>
     -
   * - R.11.8
     - Required
     -
     -
     -
   * - R.12.2
     - Required
     - :need:`gui_LvmzGKdsAgI5`, :need:`gui_RHvQj8BHlz9b`
     - Mandatory, Advisory
     -
   * - R.12.4
     - Advisory
     -
     -
     - これは明確に定義されているか、起こらないかのどちらか
   * - R.14.1
     - Required
     -
     -
     - while ループにのみ適用される
   * - R.17.9
     - Mandatory
     -
     -
     - これは ``!`` （Never）型で表され、rustc によって強制される
   * - R.18.1
     - Required
     -
     -
     - unsafe な API による
   * - R.18.2
     - Required
     -
     -
     - unsafe な API による
   * - R.18.4
     - Advisory
     -
     -
     - unsafe な API の使用に適用される
   * - R.18.6
     - Required
     -
     -
     -
   * - R.19.1
     - Mandatory
     -
     -
     -
   * - R.20.4
     - Required
     -
     -
     - 生識別子を使えば可能だが、コンパイラが見た目の衝突を防ぐ
   * - R.20.7
     - Required
     -
     -
     - ``macro_rules`` ではなく、手続き的マクロでのみ表現できる
   * - R.21.3
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.4
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.5
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.6
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.7
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.8
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.9
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.10
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.12
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.13
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.14
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.15
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.16
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.17
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.18
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.19
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.20
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.21
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.24
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.21.26
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.1
     - Required
     -
     -
     - FFI を通じて取得したリソースにのみ適用される
   * - R.22.2
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.3
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.4
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.5
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.6
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.7
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.8
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.9
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.10
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.11
     - Required
     -
     -
     -
   * - R.22.12
     - Mandatory
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.14
     - Mandatory
     -
     -
     - 同期オブジェクトを、それを使うスレッドより前に生成することに適用される
   * - R.22.15
     - Required
     -
     -
     - 同期オブジェクトを、それを使うスレッドより後に解放することに適用される
   * - R.22.16
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.17
     - Required
     -
     -
     - unsafe な extern "C" を通じてのみアクセスできる
   * - R.22.20
     - Mandatory
     -
     -
     -


表 3 – Rust に当てはまらないガイドラインのルール
---------------------------------------------------------

.. list-table::
   :header-rows: 1
   :widths: auto

   * - ガイドライン
     - カテゴリ
     - コメント
   * - **ディレクティブ**
     -
     -
   * - D.4.6
     - Advisory
     - すべてのプリミティブ型がすでにこれを満たしている
   * - D.4.8
     - Advisory
     -
   * - D.4.10
     - Required
     -
   * - **ルール**
     -
     -
   * - R.1.4
     - Required
     - これは C のバージョニングに固有のもの
   * - R.2.4
     - Advisory
     - Rust には独立したタグの名前空間がない
   * - R.3.2
     - Required
     -
   * - R.4.1
     - Required
     -
   * - R.4.2
     - Advisory
     -
   * - R.5.4
     - Required
     -
   * - R.5.7
     - Required
     - Rust には独立したタグの名前空間がない
   * - R.6.1
     - Required
     - ライブラリの機能としてのみ提供される
   * - R.6.2
     - Required
     -
   * - R.6.3
     - Required
     -
   * - R.7.3
     - Required
     -
   * - R.7.4
     - Required
     -
   * - R.7.5
     - Mandatory
     -
   * - R.7.6
     - Required
     -
   * - R.8.1
     - Required
     -
   * - R.8.2
     - Required
     -
   * - R.8.4
     - Required
     -
   * - R.8.8
     - Required
     -
   * - R.8.10
     - Required
     -
   * - R.8.11
     - Advisory
     -
   * - R.8.12
     - Required
     -
   * - R.8.14
     - Required
     -
   * - R.8.16
     - Advisory
     - 明示的には指定できない。このアラインメントを持つのは ZST のみ
   * - R.8.18
     - Required
     -
   * - R.8.19
     - Advisory
     -
   * - R.9.2
     - Required
     -
   * - R.9.3
     - Required
     -
   * - R.9.5
     - Required
     -
   * - R.9.6
     - Required
     -
   * - R.10.1
     - Required
     -
   * - R.10.2
     - Required
     -
   * - R.10.3
     - Required
     -
   * - R.10.4
     - Required
     -
   * - R.10.6
     - Required
     -
   * - R.10.7
     - Required
     -
   * - R.11.9
     - Required
     - Rust にはヌルポインタ定数がない（C に固有の概念）
   * - R.11.10
     - Required
     -
   * - R.12.3
     - Advisory
     -
   * - R.12.5
     - Mandatory
     -
   * - R.12.6
     - Required
     -
   * - R.13.2
     - Required
     - Rust では評価順序が厳密に決まっている
   * - R.13.3
     - Advisory
     -
   * - R.13.4
     - Advisory
     - Rust では結果はユニット型であり、評価順序が厳密に決まっている
   * - R.13.6
     - Required
     - Rust ではこれは式の演算子ではない
   * - R.14.2
     - Required
     -
   * - R.15.1
     - Advisory
     -
   * - R.15.2
     - Required
     -
   * - R.15.3
     - Required
     -
   * - R.15.6
     - Required
     -
   * - R.16.1
     - Required
     -
   * - R.16.2
     - Required
     -
   * - R.16.3
     - Required
     -
   * - R.16.4
     - Required
     - 対応する match 式は網羅的でなければならない
   * - R.16.5
     - Required
     - 論駁不可能なパターンは、後続の論駁可能なパターンを到達不能にする
   * - R.16.6
     - Required
     -
   * - R.16.7
     - Required
     -
   * - R.17.1
     - Required
     -
   * - R.17.3
     - Mandatory
     -
   * - R.17.4
     - Mandatory
     - Rust では値を返すのに return キーワードは不要であり、抜けるためだけに使う
   * - R.17.5
     - Required
     -
   * - R.17.10
     - Required
     -
   * - R.17.12
     - Advisory
     -
   * - R.17.13
     - Required
     -
   * - R.18.7
     - Required
     -
   * - R.18.8
     - Required
     -
   * - R.18.9
     - Required
     -
   * - R.18.10
     - Mandatory
     -
   * - R.20.1
     - Advisory
     - C のプリプロセッサに固有のルールは Rust には当てはまらない
   * - R.20.2
     - Required
     -
   * - R.20.3
     - Required
     -
   * - R.20.5
     - Advisory
     -
   * - R.20.6
     - Required
     -
   * - R.20.8
     - Required
     -
   * - R.20.9
     - Required
     -
   * - R.20.10
     - Advisory
     -
   * - R.20.11
     - Required
     -
   * - R.20.12
     - Required
     -
   * - R.20.13
     - Required
     -
   * - R.20.14
     - Required
     -
   * - R.20.15
     - Required
     -
   * - R.21.11
     - Advisory
     - 外部インタフェースがない
   * - R.21.22
     - Mandatory
     - 外部インタフェースがない
   * - R.21.23
     - Required
     - 外部インタフェースがない
   * - R.23.1
     - Advisory
     -
   * - R.23.2
     - Required
     -
   * - R.23.3
     - Advisory
     -
   * - R.23.4
     - Required
     -
   * - R.23.5
     - Advisory
     -
   * - R.23.6
     - Required
     -
   * - R.23.7
     - Advisory
     -
   * - R.23.8
     - Required
     -


用語集
--------

カテゴリ
........

MISRA C の各ガイドラインには、次のようにカテゴリが割り当てられています。

.. list-table::
   :header-rows: 1
   :widths: auto

   * - カテゴリ
     - 解釈
   * - Mandatory
     - コードは常にこのガイドラインに準拠しなければならない。正式な逸脱は認められない
   * - Required
     - コードはこのガイドラインに準拠しなければならない。準拠しない場合は正式な逸脱が必要
   * - Advisory
     - 合理的に実行可能な限り従うべき推奨事項。advisory のガイドラインには正式な逸脱は必要ないが、正式な逸脱プロセスに従わない場合は、不遵守を文書化するための代わりの手立てを講じるべき


脚注
.........

.. rubric:: 脚注

.. _MISRA C\:2025 Addendum 6: https://misra.org.uk/app/uploads/2025/03/MISRA-C-2025-ADD6.pdf

