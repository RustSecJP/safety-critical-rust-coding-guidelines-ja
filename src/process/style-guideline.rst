.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

##############################
スタイルガイドライン
##############################

******************************
要求レベルの指定
******************************

要求レベルの指定には `IETF RFC 2119 <https://datatracker.ietf.org/doc/html/rfc2119>`_
に従います。

******************************
コーディングガイドラインの例
******************************

以下はコーディングガイドラインの例です。

それぞれの部分を見ていきます。

* ``guideline``
* ``rationale``
* ``non_compliant_example``
* ``compliant_example``
* ``bibliography``

::

   .. guideline:: Do not use an integer type as a divisor during integer division
      :id: gui_7y0GAMmtMhch
      :category: advisory
      :status: draft
      :release: latest
      :fls: fls_Q9dhNiICGIfr
      :decidability: decidable
      :scope: module
      :tags: numerics, subset

      Do not provide a right operand of integer type :cite:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
      during a division expression :cite:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR` or remainder
      expression :cite:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR` when the left operand also has
      integer type.

      .. rationale::
         :id: rat_vLFlPWSCHRje
         :status: draft

         Integer division and integer remainder division both panic when the right operand
         has a value of zero. Division by zero is undefined in mathematics because it leads
         to contradictions and there is no consistent value that can be assigned as its result.

      .. non_compliant_example::
         :id: non_compl_ex_0XeioBrgfh5z
         :status: draft

         Both the division and remainder operations in this non-compliant example will panic
         if evaluated because the right operand is zero.

         .. rust-example::
             :compile_fail:

             fn main() {
                 let x = 0;
                 let _y = 5 / x; // この行はパニックします。
                 let _z = 5 % x; // この行もパニックします。
             }

      .. compliant_example::
         :id: compl_ex_k1CD6xoZxhXb
         :status: draft

         Checked division prevents division by zero from occurring.
         The programmer can then handle the returned :std:`std::option::Option`.

         .. rust-example::

            fn main() {
                // 検査付き除算の API を使う
                let _y = match 5i32.checked_div(0) {
                    None => 0,
                    Some(r) => r,
                };

                // 検査付き剰余の API を使う
                let _z = match 5i32.checked_rem(0) {
                    None => 0,
                    Some(r) => r,
                };
            }

      .. compliant_example::
         :id: compl_ex_k1CD6xoZxhXc
         :status: draft

         This compliant solution creates a divisor using :std:`std::num::NonZero`.

         .. rust-example::
            :version: 1.79

            use std::num::NonZero;

            fn main() {
                let x = 0u32;
                if let Some(divisor) = NonZero::<u32>::new(x) {
                    let _result = 5u32 / divisor;
                }
            }

      .. bibliography::
         :id: bib_7y0GAMmtMhch
         :status: draft

         .. list-table::
            :header-rows: 0
            :widths: auto
            :class: bibliography-table

            * - :bibentry:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
              - The Rust FLS. "Types and Traits - Integer Types." https://rust-lang.github.io/fls/types-and-traits.html#integer-types
            * - :bibentry:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR`
              - The Rust FLS. "Expressions - Syntax - DivisionExpression." https://rust-lang.github.io/fls/expressions.html#syntax_divisionexpression
            * - :bibentry:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR`
              - The Rust FLS. "Expressions - Syntax - RemainderExpression." https://rust-lang.github.io/fls/expressions.html#syntax_remainderexpression

``guideline``
=============

::

   .. guideline:: Do not use an integer type as a divisor during integer division
      :id: gui_7y0GAMmtMhch
      :category: advisory
      :status: draft
      :release: latest
      :fls: fls_Q9dhNiICGIfr
      :decidability: decidable
      :scope: module
      :tags: numerics, subset

``guideline`` のタイトル
------------------------

タイトルはガイドラインの説明を提供 **しなければなりません（MUST）** 。

``guideline`` の ``id``
-----------------------

各ガイドラインの一意な識別子です。ガイドラインの識別子は ``gui_`` で始まら
**なければなりません（MUST）** 。

これらの識別子はリリースをまたいで **安定** しているものとみなされ、削除しては
**なりません（MUST NOT）** 。詳しくは後述の ``status`` を参照してください。

規則に従うことを確実にするため、 ``generate_guideline_templates.py`` スクリプトを使って生成
**しなければなりません（MUST）** 。

``category``
------------

次のいずれかの値で **なければなりません（MUST）** 。

* ``mandatory``
* ``required``
* ``advisory``
* ``disapplied``

``mandatory``
^^^^^^^^^^^^^

この文書に準拠していると主張するコードは、 ``mandatory`` と指定されたすべてのガイドラインに従わ **なければなりません（MUST）** 。

*TODO(pete.levasseur): どのような場合にガイドラインにこれを選ぶのがよいか、ヒントをさらに追加する。*

``required``
^^^^^^^^^^^^

この文書に準拠していると主張するコードは、 ``required`` と指定されたすべてのガイドラインに従わ **なければなりません（MUST）** 。
従わない場合は、 :ref:`Compliance` に記載されているとおり、正式な逸脱が必要です。

組織やプロジェクトは、任意の ``required`` のガイドラインを ``mandatory`` に再分類することを選んで **もかまいません（MAY）** 。

*TODO(pete.levasseur): どのような場合にガイドラインにこれを選ぶのがよいか、ヒントをさらに追加する。*

``advisory``
^^^^^^^^^^^^

これらは推奨事項であり、適用 **すべきです（SHOULD）** 。ただし、 ``advisory`` というカテゴリは、
これらの項目を無視してよいという意味ではありません。合理的に実行可能な限り従う **べき（SHOULD）** である、という意味です。
advisory のガイドラインには正式な逸脱は必要ありません。しかし、正式な逸脱プロセスに従わない場合は、
不遵守を文書化するための代わりの手立てを講じ **なければなりません（MUST）** 。

組織やプロジェクトは、任意の ``advisory`` のガイドラインを ``mandatory`` や ``required`` 、
あるいは ``disapplied`` に再分類することを選んで **もかまいません（MAY）** 。

ガイドラインを提案する場合は、 ``advisory`` として提出し、
ガイドラインに正しいカテゴリを割り当てるための支援を求めて **もかまいません（MAY）** 。

*TODO(pete.levasseur): どのような場合にガイドラインにこれを選ぶのがよいか、ヒントをさらに追加する。*

``disapplied``
^^^^^^^^^^^^^^

これらは強制が想定されていないガイドラインであり、不遵守はすべて無視して **もかまいません（MAY）** 。

選択した Rust コンパイラのリリースにガイドラインが当てはまらない場合、コーディングガイドラインの
:ref:`Compliance` においては、そのガイドラインを ``disapplied`` として扱わ **なければなりません（MUST）** 。

組織やプロジェクトは、任意の ``disapplied`` のガイドラインを ``mandatory`` や ``required`` 、
あるいは ``advisory`` に再分類することを選んで **もかまいません（MAY）** 。

*注*: ガイドラインを適用対象外にしたいときは、ガイドラインのカテゴリを ``disapplied`` に変更するのではなく、
カテゴリはそのままにして、代わりに ``status`` を ``retired`` に変更 **しなければなりません（MUST）** 。

*TODO(pete.levasseur): どのような場合にガイドラインにこれを選ぶのがよいか、ヒントをさらに追加する。*

``guideline`` の ``status``
---------------------------

次のいずれかの値で **なければなりません（MUST）** 。

* ``draft``
* ``approved``
* ``retired``

ガイドラインにはライフサイクルがあります。最初に提案されたときは、採用とフィードバックが蓄積されるように
``draft`` と指定され **なければなりません（MUST）** 。コーディングガイドライン小委員会は、
``draft`` のガイドラインを定期的に見直し、 ``approved`` に昇格させるか ``retired`` に降格させるかを
決め **なければなりません（MUST）** 。

``approved`` のガイドラインが ``retired`` に移されることも、ときには **あり得ます（MAY）** 。
その理由はいくつか考えられます。たとえば、ガイドラインがうまく当てはまらなかったり間違っていたりした場合や、
1 つのガイドラインをより細かくして複数のガイドラインに置き換える場合です。

詳しくは :ref:`Guideline Lifecycle` を参照してください。

``draft``
^^^^^^^^^

これらのガイドラインはまだ効力があるとはみなされませんが、強制して **もかまわない（MAY）** 程度には成熟しています。
:ref:`Compliance` に記載されているような正式な逸脱は必要ありませんが、不遵守を文書化するための
代わりの手立てを講じ **なければなりません（MUST）** 。

*注*: ``draft`` のガイドラインを使ってもらい、フィードバックを得ることで、 ``approved`` への昇格か
``retired`` への降格かを決める助けになります。

``approved``
^^^^^^^^^^^^

これらのガイドラインは強制され **なければなりません（MUST）** 。逸脱はすべて、それぞれの
``category`` に応じた規則に従わ **なければなりません（MUST）** 。

``retired``
^^^^^^^^^^^^^^

これらは強制が想定されていないガイドラインであり、不遵守はすべて無視して **もかまいません（MAY）** 。

*注*: ``retired`` の ``status`` は、ガイドラインに割り当てられたどの ``category`` よりも優先されます。
つまり実質的に、そのガイドラインには ``disapplied`` の ``category`` が与えられ、 ``mandatory`` 、
``required`` 、 ``advisory`` のいずれにも再分類できなくなります。ガイドラインが廃止された時点で
割り当てられていた ``category`` はそのまま残されます。

``release``
------------------------

各ガイドラインには、そのガイドラインが当てはまる Rust コンパイラのリリースを記載 **しなければなりません（MUST）** 。

ガイドラインは、おそらく複数のリリースに当てはまることが **あり得ます（MAY）** 。

ガイドラインが複数のリリースに当てはまる場合、そのリストはセミコロン区切りで **なければなりません（MUST）** 。

``fls``
-------

各ガイドラインは、FLS の適切な ``paragraph-id`` へのつながりを持た **なければなりません（MUST）** 。
この FLS へのつながりによって、ガイドラインは仕様を過不足なく、ちょうど網羅します。

1 つの FLS の ``paragraph-id`` に、複数のガイドラインが当てはまって **もかまいません（MAY）** 。

``decidability``
----------------

次のいずれかの値で **なければなりません（MUST）** 。

* ``decidable``
* ``undecidable``

``decidability`` （決定可能性）は、静的解析器が「このコードはこのルールに準拠しているか？」
という問いに答えられるかどうかという、理論上の能力を表します。

そのような静的解析器が *すべての場合において* この問いに「はい」か「いいえ」で答えられるなら、
ガイドラインは ``decidable`` （決定可能）に分類され **なければならず（MUST）** 、そうでなければ
``undecidable`` （決定不能）に分類され **なければなりません（MUST）** 。


``scope``
---------

次のいずれかの値で **なければなりません（MUST）** 。

* ``module``
* ``crate``
* ``system``

``scope`` は、ガイドラインが当てはまるコードの各箇所について、プログラムのどのレベルのスコープで
ガイドラインに従っていることを確認できるかを表します。

たとえば、 ``unsafe`` なコードが使われている各箇所について、モジュールレベルで確認しなければならない
ガイドラインがあるかもしれません。これが必要なのは、あるモジュールの中で ``unsafe`` が 1 回でも
使われていれば、特定の不変条件についてモジュール全体を確認しなければならないからです。

ガイドラインを書くときは、適用しやすくするため、セマンティクスが許す限り ``scope`` をできるだけ小さく
するよう努め **なければなりません（MUST）** 。

``module``
^^^^^^^^^^

ほかのモジュールやクレートを参照せずにモジュールレベルで確認できるガイドラインは、
``module`` に分類され **なければなりません（MUST）** 。

``crate``
^^^^^^^^^

モジュールレベルでは確認できないものの、ソーステキスト全体を必要としないガイドラインは、
``crate`` に分類され **なければなりません（MUST）** 。

``system``
^^^^^^^^^^

モジュールレベルでもクレートレベルでも確認できず、ソーステキスト全体を必要とするガイドラインは、
``system`` に分類され **なければなりません（MUST）** 。


``tags``
--------

``tags`` は主に記述的なものであり、規範的なものではありません。似たガイドラインの間の共通点を
見つけるための手段です。

各ガイドラインは、 ``tags`` に少なくとも 1 つの項目を持た **なければなりません（MUST）** 。

ガイドラインの内容
------------------

各 ``guideline`` は、オプションに続けて、そのガイドラインが何を扱うかの概要を示す内容を持た
**なければなりません（MUST）** 。

内容は、ガイドラインの範囲を説明しつつも、できるだけ短く、それだけで完結したものを目指す
**べきです（SHOULD）** 。

ガイドラインの内容は、規範的な補足（Amplification）と例外（Exception）から成り、
規範的でない根拠（Rationale）と例によって裏付けられます。
ガイドラインの正当性を示す詳しい説明は、規範的な内容ではなく、規範的でない根拠に書く
**べきです（SHOULD）** 。

補足
^^^^

*補足* （Amplification）は、ガイドラインのアトリビュートブロックのすぐ下、ほかの小見出しより前に
置いて **もかまわない（MAY）** テキストのブロックです。
補足を書いた場合、それは規範的です。 ``guideline`` のタイトルと食い違う場合は、補足が優先され
**なければなりません（MUST）** 。この仕組みは便利です。複雑な概念を短いタイトルで伝え、
その下のテキストで詳しく絞り込めるからです。

補足の内容では、ガイドラインの根拠や規範的でない説明を扱う **べきではありません（SHOULD NOT）** 。
それらは、役に立つ場合には ``rationale`` や例のセクションで示す **べきです（SHOULD）** 。

補足では、 ``:cite:`` ロールを使って参考文献を引用して **もかまいません（MAY）**
（後述の `引用ロール`_ を参照）。

::

      Do not provide a right operand of integer type :cite:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
      during a division expression :cite:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR` or remainder
      expression :cite:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR` when the left operand also has
      integer type.

例外
^^^^

ガイドラインの内容には、 *例外* （Exception）という題のセクションを含めて **もかまいません（MAY）** 。
その後には、ガイドラインが当てはまらない状況を説明するテキストを続けます。例外を使うと、
一部のガイドラインの記述を簡単にできます。例外とは、ガイドラインが当てはまらない状況のことである、
という点に注意することが大切です。例外によってガイドラインに準拠しているコードには、逸脱は必要ありません。

例外を書いた場合、それは規範的です。 ``guideline`` のタイトルや補足と食い違う場合は、例外がその両方に
優先します。個々のガイドラインによって、補足やタイトルに明示的な例外を付けて記述の一部を上書きする方が
わかりやすいこともあれば、例外となる場合を補足の中の文として組み込んで表す方がわかりやすいこともあります。
この判断は編集上のものです。

``rationale``
=============

各ガイドラインは、それを含めて強制する理由として *根拠* （Rationale）を示さ **なければなりません（MUST）** 。

::

      .. rationale::
         :id: rat_vLFlPWSCHRje
         :status: draft

         Integer division and integer remainder division both panic when the right operand
         has a value of zero. Division by zero is undefined in mathematics because it leads
         to contradictions and there is no consistent value that can be assigned as its result.

``rationale`` の ``id``
-----------------------

各根拠の一意な識別子です。根拠の識別子は ``rat_`` で始まら **なければなりません（MUST）** 。

これらの識別子はリリースをまたいで **安定** しているものとみなされ、削除しては
**なりません（MUST NOT）** 。詳しくは後述の ``status`` を参照してください。

規則に従うことを確実にするため、 ``generate_guideline_templates.py`` スクリプトを使って生成
**しなければなりません（MUST）** 。

``rationale`` の ``status``
---------------------------

``rationale`` の ``status`` オプションは、親の ``guideline`` の ``status`` と一致し
**なければなりません（MUST）** 。

根拠の内容
----------

根拠の内容は、このガイドラインがなぜ役に立つのかについて、関連する背景を示す **べきです（SHOULD）** 。
根拠は、ガイドラインの対象に関係する未定義動作や既知のエラーに言及する **べきです（SHOULD）** 。
根拠は、ほかのガイドラインや、参考文献に挙げた外部の文書に言及して **もかまいません（MAY）** 。

簡潔な例を示せる場合は、根拠をコード例で裏付ける **べきです（SHOULD）** 。

``non_compliant_example``
=========================

::

      .. non_compliant_example::
         :id: non_compl_ex_0XeioBrgfh5z
         :status: draft

         Both the division and remainder operations in this non-compliant example will panic
         if evaluated because the right operand is zero.

         .. rust-example::
             :compile_fail:

             fn main() {
                 let x = 0;
                 let _y = 5 / x; // この行はパニックします。
                 let _z = 5 % x; // この行もパニックします。
             }

``non_compliant_example`` の ``id``
-----------------------------------

各 ``non_compliant_example`` の一意な識別子です。 ``non_compliant_example`` の識別子は
``non_compl_ex_`` で始まら **なければなりません（MUST）** 。

これらの識別子はリリースをまたいで **安定** しているものとみなされ、削除しては
**なりません（MUST NOT）** 。詳しくは後述の ``status`` を参照してください。

規則に従うことを確実にするため、 ``generate_guideline_templates.py`` スクリプトを使って生成
**しなければなりません（MUST）** 。

``non_compliant_example`` の ``status``
---------------------------------------

``non_compl_ex`` の ``status`` オプションは、親の ``guideline`` の ``status`` と一致し
**なければなりません（MUST）** 。

``non_compliant_example`` の内容
--------------------------------

``non_compliant_example`` の内容セクションには、コードの説明とコード例の両方を含め **なければなりません（MUST）** 。

``non_compliant_example`` は規範的でもなく、網羅的でもありません。 ``guideline`` の内容が優先され **なければなりません（MUST）** 。

``non_compliant_example`` のコードの説明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``non_compliant_example`` のコードの説明では、ガイドラインを適用しないと望ましくないコードになる理由を、
文章で説明し **なければなりません（MUST）** 。

``non_compliant_example`` のコードの説明は、1 文以内の簡単な説明で **もかまいません（MAY）** 。

``non_compliant_example`` のコードの説明は、続くコード例を説明するのに必要な長さにとどめる
**べきです（SHOULD）** 。

``non_compliant_example`` のコード例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``non_compliant_example`` のコード例は ``.. rust-example::`` ディレクティブを使わ **なければなりません（MUST）**
（後述の `rust-example ディレクティブ`_ を参照）。

``non_compliant_example`` のコード例は、できるだけ短く、わかりやすくする **べきです（SHOULD）** 。

``non_compliant_example`` のコード例が複雑または長い場合は、わかりやすくするためのコメントを含める
**べきです（SHOULD）** 。

``non_compliant_example`` のコード例は、現在のガイドライン以外のガイドライン違反を含んでは
**なりません（MUST NOT）** 。

``compliant_example``
=====================

ガイドラインがある行為を全面的に禁止している場合、つまり不適合なコードの目的を適合する方法で達成する手段が
ない場合は、関係のない例を示す（あるいは奇妙な回避策を勧める）のではなく、適合例を省く **べきです（SHOULD）** 。
目的を達成する明確で慣用的な適合する方法がある場合は、対応する不適合例の後に適合例を示す
**べきです（SHOULD）** 。

::

      .. compliant_example::
         :id: compl_ex_k1CD6xoZxhXb
         :status: draft

         Checked division prevents division by zero from occurring.
         The programmer can then handle the returned :std:`std::option::Option`.

         .. rust-example::

            fn main() {
                // 検査付き除算の API を使う
                let _y = match 5i32.checked_div(0) {
                    None => 0,
                    Some(r) => r,
                };

                // 検査付き剰余の API を使う
                let _z = match 5i32.checked_rem(0) {
                    None => 0,
                    Some(r) => r,
                };
            }

``compliant_example`` の ``id``
-------------------------------

各 ``compliant_example`` の一意な識別子です。 ``compliant_example`` の識別子は
``compl_ex_`` で始まら **なければなりません（MUST）** 。

これらの識別子はリリースをまたいで **安定** しているものとみなされ、削除しては
**なりません（MUST NOT）** 。詳しくは後述の ``status`` を参照してください。

規則に従うことを確実にするため、 ``generate_guideline_templates.py`` スクリプトを使って生成
**しなければなりません（MUST）** 。

``compliant_example`` の ``status``
-----------------------------------

``compl_ex`` の ``status`` オプションは、親の ``guideline`` の ``status`` と一致し
**なければなりません（MUST）** 。

``compliant_example`` の内容
----------------------------

``compliant_example`` の内容セクションには、コードの説明とコード例の両方を含め **なければなりません（MUST）** 。

``compliant_example`` は規範的でもなく、網羅的でもありません。 ``guideline`` の内容が優先され **なければなりません（MUST）** 。

``compliant_example`` のコードの説明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`compliant_example` のコードの説明は、1 文以内の簡単な説明で **もかまいません（MAY）** 。

``compliant_example`` のコードの説明は、続くコード例を説明するのに必要な長さにとどめる
**べきです（SHOULD）** 。


``compliant_example`` のコード例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``compliant_example`` のコード例は ``.. rust-example::`` ディレクティブを使わ **なければなりません（MUST）**
（後述の `rust-example ディレクティブ`_ を参照）。

``compliant_example`` のコード例は、できるだけ短く、わかりやすくする **べきです（SHOULD）** 。

``compliant_example`` のコード例が複雑または長い場合は、わかりやすくするためのコメントを含める
**べきです（SHOULD）** 。

``compliant_example`` のコード例は、すべてのガイドラインに準拠し **なければなりません（MUST）** 。

``compliant_example`` のコード例は、ガイドラインに違反する寸前まで近づきつつも準拠の範囲にとどまることで、
ガイドラインを説明するよう努める **べきです（SHOULD）** 。

``bibliography``
================

各 ``guideline`` は、外部の文書や仕様を参照している場合、関連する ``bibliography`` （参考文献）を持つ
**べきです（SHOULD）** 。参考文献は、出典を引用するための構造化された方法を提供し、
読者が参照先の資料へ直接移動できるようにします。

::

      .. bibliography::
         :id: bib_7y0GAMmtMhch
         :status: draft

         .. list-table::
            :header-rows: 0
            :widths: auto
            :class: bibliography-table

            * - :bibentry:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
              - The Rust FLS. "Types and Traits - Integer Types." https://rust-lang.github.io/fls/types-and-traits.html#integer-types
            * - :bibentry:`gui_7y0GAMmtMhch:FLS-DIVISION-EXPR`
              - The Rust FLS. "Expressions - Syntax - DivisionExpression." https://rust-lang.github.io/fls/expressions.html#syntax_divisionexpression
            * - :bibentry:`gui_7y0GAMmtMhch:FLS-REMAINDER-EXPR`
              - The Rust FLS. "Expressions - Syntax - RemainderExpression." https://rust-lang.github.io/fls/expressions.html#syntax_remainderexpression

``bibliography`` の ``id``
--------------------------

各 ``bibliography`` の一意な識別子です。 ``bibliography`` の識別子は
``bib_`` で始まら **なければなりません（MUST）** 。

``bib_`` の後に続く部分は、ガイドラインの ID の後半部分と一致する **べきです（SHOULD）**
（たとえば、ガイドライン ``gui_7y0GAMmtMhch`` に対しては ``bib_7y0GAMmtMhch`` ）。

``bibliography`` の ``status``
------------------------------

``bibliography`` の ``status`` オプションは、親の ``guideline`` の ``status`` と一致し
**なければなりません（MUST）** 。

``bibliography`` の内容
-----------------------

参考文献は、正しくスタイルが適用されるように、ヘッダ行のない ``list-table`` として、
``bibliography-table`` クラスを付けて書式設定し **なければなりません（MUST）** 。

各行は 2 つの列を含ま **なければなりません（MUST）** 。

1. ``:bibentry:`` ロールを使った引用のアンカー
2. 著者、タイトル、URL を含む引用の説明

参考文献の検証
^^^^^^^^^^^^^^

ビルドシステムは、参考文献の項目について次の点を検証します。

* **引用キーの形式** - キーは ``UPPERCASE-WITH-HYPHENS`` の形で **なければなりません（MUST）** （例: ``FLS-INTEGER-TYPES`` 、 ``CERT-C-INT34`` ）
* **ガイドライン ID の一致** - ``:bibentry:`` ロール内のガイドライン ID は、それを含むガイドラインと一致し **なければなりません（MUST）**
* **URL の一貫性** - 異なるガイドラインで同じ URL を使う場合、同一の引用キーと説明を使わ **なければなりません（MUST）**
* **引用の参照** - すべての ``:cite:`` 参照には、対応する ``:bibentry:`` の定義がなければ **なりません（MUST）**

******************************
引用ロール
******************************

このドキュメントシステムには、引用を管理するための 2 つのロールがあります。本文中で引用を参照するための
``:cite:`` と、参考文献の中で引用のアンカーを定義するための ``:bibentry:`` です。

``:cite:`` ロール
=================

``:cite:`` ロールは、ガイドラインの本文中に、対応する参考文献の項目へリンクするクリック可能な参照を作ります。

**構文:** ``:cite:`gui_GUIDELINE_ID:CITATION-KEY```

**例:**

::

   As documented in :cite:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`, integer types have
   specific behaviors during division operations.

これは ``[FLS-INTEGER-TYPES]`` として表示され、参考文献の項目にリンクします。

ガイドライン ID の接頭辞（ ``gui_7y0GAMmtMhch:`` ）は、引用を含むガイドラインの ID と一致し
**なければなりません（MUST）** 。これにより、引用は正しく名前空間で区切られ、ビルド時に検証されます。

``:bibentry:`` ロール
=====================

``:bibentry:`` ロールは、参考文献の表の中に、 ``:cite:`` の参照がリンクできるアンカーを作ります。

**構文:** ``:bibentry:`gui_GUIDELINE_ID:CITATION-KEY```

**例:**

::

   * - :bibentry:`gui_7y0GAMmtMhch:FLS-INTEGER-TYPES`
     - The Rust FLS. "Types and Traits - Integer Types." https://rust-lang.github.io/fls/...

これは ``[FLS-INTEGER-TYPES] ↩`` として表示され、読者を本文中の引用箇所へ戻す
戻るボタンが付きます。

引用キーの形式
==============

引用キーは次の形式に従わ **なければなりません（MUST）** 。

* 大文字で始まる
* 大文字、数字、ハイフンのみを含む
* 大文字または数字で終わる
* 最大 50 文字

**正しい例:**

* ``FLS-INTEGER-TYPES``
* ``RUST-REF-UNION``
* ``CERT-C-INT34``
* ``ISO-26262-2018``

**誤った例:**

* ``fls-integer-types`` （小文字）
* ``123-KEY`` （数字で始まる）
* ``KEY_WITH_UNDERSCORE`` （アンダースコアを含む）

******************************
rust-example ディレクティブ
******************************

ガイドライン内のすべての Rust のコード例は、 ``.. code-block:: rust`` ではなく
``.. rust-example::`` ディレクティブを使わ **なければなりません（MUST）** 。このディレクティブは次の機能を提供します。

* Rust Playground による対話的な実行
* クリップボードへのコピー機能
* 未定義動作を検出するための Miri との統合
* ビルド時のコード例の検証
* 隠し行のサポート

基本的な使い方
==============

::

   .. rust-example::

      fn main() {
          println!("Hello, world!");
      }

これにより、コピー、実行、切り替えのボタンが付いた対話的なコードブロックが作られます。

rustdoc のアトリビュート
========================

このディレクティブは、コード例のコンパイルや実行のしかたを制御する、標準の rustdoc のアトリビュートをサポートしています。

``:ignore:``
------------

コード例はコンパイルもテストもされません。意図的に不完全にした説明用のコードに使います。

::

   .. rust-example::
      :ignore:

      // このコードは説明用です
      fn hypothetical_feature() { ... }

``:compile_fail:``
------------------

コード例はコンパイルに失敗する **べきです（SHOULD）** 。必要に応じて、想定されるエラーコードを指定できます。

::

   .. rust-example::
      :compile_fail: E0277

      fn example() {
          let x: i32 = "string"; // 型の不一致
      }

``:should_panic:``
------------------

コード例はコンパイルできるが、実行時にパニックする **べきです（SHOULD）** 。

::

   .. rust-example::
      :should_panic:

      fn main() {
          panic!("This is expected");
      }

``:no_run:``
------------

コード例はコンパイルされますが、実行はされません。特定の実行時の条件を必要とするコードや、
副作用のあるコードに使います。

::

   .. rust-example::
      :no_run:

      fn main() {
          std::process::exit(1);
      }

Miri との統合
=============

``:miri:`` オプションは、未定義動作を検出するための Miri によるチェックを有効にします。
``unsafe`` なコードを含むコード例は、 ``:miri:`` オプションを含め **なければなりません（MUST）** 。

``:miri:`` （デフォルト: check）
--------------------------------

Miri を実行し、未定義動作がないことを期待します。

::

   .. rust-example::
      :miri:

      fn main() {
          unsafe {
              let x: i32 = 42;
              let ptr = &x as *const i32;
              println!("{}", *ptr); // 安全: 有効なポインタ
          }
      }

``:miri: expect_ub``
--------------------

Miri を実行し、未定義動作が検出されることを期待します。 *してはいけない* ことを示すときに使います。

::

   .. rust-example::
      :miri: expect_ub

      fn main() {
          unsafe {
              let ptr: *const i32 = std::ptr::null();
              let _ = *ptr; // UB: ヌルポインタの参照外し
          }
      }

``:miri: skip``
---------------

Miri によるチェックをスキップします。コード例の中の操作を Miri がサポートしていない場合に使います。
スキップする理由は、周りの文章に記載する **べきです（SHOULD）** 。

::

   .. rust-example::
      :miri: skip

      // Miri はこの FFI 操作をサポートしていない
      fn main() {
          unsafe { /* FFI 呼び出し */ }
      }

*注:* Miri にはコンパイルでき実行できるコードが必要なので、 ``:miri:`` は ``:ignore:`` 、 ``:compile_fail:`` 、
``:no_run:`` と組み合わせることはできません。

警告の扱い
==========

``:warn:`` オプションは、コンパイラの警告をどう扱うかを制御します。

``:warn:`` または ``:warn: error``
----------------------------------

コンパイラの警告があると失敗にします。設定で ``rust_examples_fail_on_warnings = True`` の場合は、
これがデフォルトです。

``:warn: allow``
----------------

コンパイラの警告を、失敗にせずに許可します。

::

   .. rust-example::
      :warn: allow

      fn main() {
          let x = 42; // 警告: 使われていない変数（許可される）
      }

ツールチェーンのオプション
==========================

``:edition:``
-------------

Rust のエディションを指定します。デフォルトは ``2021`` です。

::

   .. rust-example::
      :edition: 2018

      // エディション 2018 固有のコード

``:channel:``
-------------

リリースチャネルを指定します。 ``stable`` 、 ``beta`` 、 ``nightly`` のいずれかです。

::

   .. rust-example::
      :channel: nightly

      #![feature(some_nightly_feature)]

``:version:``
-------------

対象とする Rust のバージョンを指定します。設定されたデフォルトとバージョンが大きく異なる場合は、
バッジが表示されます。

::

   .. rust-example::
      :version: 1.79

      use std::num::NonZero;

      fn main() {
          // Rust 1.79 のフィーチャを使うコード
      }

隠し行
======

``#`` とそれに続く空白で始まる行は、デフォルトでは隠されますが、コードのコンパイルと実行には含まれます。
これにより、コンパイルできる状態を保ちながら、コード例の関係する部分だけを見せることができます。

::

   .. rust-example::

      # use std::collections::HashMap;
      # fn main() {
      let mut map = HashMap::new();
      map.insert("key", "value");
      # }

隠し行は、表示された出力の切り替えボタン（目のアイコン）を使って表示できます。

隠し行は次のものに使う **べきです（SHOULD）** 。

* 定型的なインポート
* ``fn main() {}`` のラッパ
* コード例の目的から注意をそらすセットアップのコード
* ``#[allow(dead_code)]`` や、それに類するアトリビュート

表示オプション
==============

``:show_hidden:``
-----------------

隠し行を隠さずに、デフォルトで表示します。

``:name:``
----------

参照用に、コード例に名前を付けます。

******************************
標準ライブラリへのリンク
******************************

``:std:`` ロールは、Rust の標準ライブラリのドキュメントへのリンクを作ります。

**構文:** ``:std:`path::to::Item```

**例:**

::

   The type needs to implement :std:`core::marker::Copy`.

   See :std:`std::option::Option` for the return type.

   Use :std:`std::num::NonZero` for guaranteed non-zero values.

このロールは、指定されたパスで Rust のドキュメントを検索するリンクを生成します。
標準ライブラリのドキュメントへのインラインの URL の代わりに、これを使う **べきです（SHOULD）** 。

******************************
ビルド時の検証
******************************

ビルドシステムは、ガイドラインの品質を確保するため、いくつかの検証を行います。

必須のフィールド
================

各ガイドラインには、次のフィールドが記入されてい **なければなりません（MUST）** 。

* ``category``
* ``release``
* ``fls``
* ``decidability``
* ``scope``
* ``tags``

必須の子要素
============

各ガイドラインは、次の子要素を持た **なければなりません（MUST）** 。

* ``rationale``
* ``non_compliant_example``
* ``compliant_example``

``bibliography`` は **任意（OPTIONAL）** ですが、外部の出典を参照している場合は含める
**べきです（SHOULD）** 。

インラインの URL の検出
=======================

ビルドシステムは、ガイドラインの本文中のインラインの URL を検出し、エラーとして報告します。
URL をガイドラインの内容に直接書いては **なりません（MUST NOT）** 。代わりに次のようにします。

* Rust の標準ライブラリのドキュメントには、 ``:std:`` ロールを使う
* 外部の参照は ``bibliography`` に追加し、 ``:cite:`` を使う

**不適合:**

::

   See https://doc.rust-lang.org/std/num/struct.Wrapping.html for details.

**適合:**

::

   See :std:`std::num::Wrapping` for details.

外部の出典の場合は次のようにします。

::

   As documented in :cite:`gui_MyGuideline:SOURCE-NAME`, ...
