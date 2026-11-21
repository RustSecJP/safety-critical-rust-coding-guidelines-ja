.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

``unsafe`` なコードでは ``unsafe`` キーワードが見えるようにする
===============================================================

.. guideline:: ``unsafe`` なコードでは ``unsafe`` キーワードが見えるようにする
   :id: gui_ZDLZzjeOwLSU
   :category: required
   :status: draft
   :release: 1.85-latest
   :fls: fls_8kqo952gjhaf
   :decidability: decidable
   :scope: crate
   :tags: readability, reduce-human-error

   安全性の保証に違反するおそれのあるすべてのコードには、目に見える ``unsafe`` キーワードで印を付けてください
   :cite:`gui_ZDLZzjeOwLSU:RUST-REF-UNSAFE-KEYWORD`。

   次の構文では、``unsafe`` を明示的に見えるようにする必要があります。

   * ``extern`` ブロックは ``unsafe extern`` として宣言しなければなりません
   * ``#[no_mangle]`` アトリビュートは ``#[unsafe(no_mangle)]`` と書かなければなりません
   * ``#[export_name]`` アトリビュートは ``#[unsafe(export_name)]`` と書かなければなりません
   * ``#[link_section]`` アトリビュートは ``#[unsafe(link_section)]`` と書かなければなりません

   .. note::

      Rust 2024 エディションからは、これらの場面で ``unsafe`` の使用が必須です。
      ``#[link]`` と ``#[link_ordinal]`` アトリビュートは ``extern`` ブロックに付ける必要があるので、
      ``unsafe extern`` の要件によって暗黙のうちにカバーされます。
      unsafe なアトリビュートに関するトラッキングイシューは rust-lang/rust#82499 を参照してください。

   .. rationale::
      :id: rat_eQV3s9ggNegr
      :status: draft

      * 監査可能性とレビュー

        * ``unsafe`` ブロックは明確な監査の境界を作ります。レビュアは、Rust の安全性の保証に
          違反するおそれのあるコードに集中できます :cite:`gui_ZDLZzjeOwLSU:RUSTNOMICON-MEET-SAFE`
        * ISO 26262 :cite:`gui_ZDLZzjeOwLSU:ISO-26262` や
          DO-178C :cite:`gui_ZDLZzjeOwLSU:DO-178C` のようなセーフティクリティカルな規格は、危険な操作のトレーサビリティを要求しています
        * 安全性の要件を手作業で守らなければならない箇所をすべて列挙し、文書化するのに役立ちます
        * ISO 26262 Part 6, Table 1, objective 1c（言語サブセットの使用）を満たします
        * DO-178C Section 6.3.4.f（ソースコードのトレーサビリティ）を満たします

      * 責任の明示的な承認

        * ``unsafe`` キーワードは、コンパイラが検証できない不変条件を守る責任を
          プログラマが引き受けていることを示します
        * 意識的な判断なしに ``unsafe`` な操作を誤って使うことを防ぎます
        * セーフティクリティカルなシステムにおける多層防御の原則に沿っています

      * 静的解析とツール

        * ``cargo-geiger`` :cite:`gui_ZDLZzjeOwLSU:CARGO-GEIGER`、``unsafe-inspect``、
          独自のリンタなどのツールで、unsafe ブロックを自動的に見つけて数えることができます
        * 安全性評価のための「unsafe 密度」のような指標を使えるようにします
        * 認証規格が要求する適格性確認のエビデンスを支えます

      * 認証のためのトレーサビリティ

        * セーフティクリティカルな認証では、危険な操作が特定され管理されていることを
          示す必要があります
        * 目に見える ``unsafe`` トークンは、セーフティケースやハザード分析と直接結びつきます
        * 各 unsafe な操作がレビューされ正当化されたことを示す、必要な文書の作成を
          容易にします

   .. non_compliant_example::
      :id: non_compl_ex_FdmuPXGZr4EP
      :status: draft

      ``#[no_mangle]`` アトリビュートは unsafe です。既存のシンボルと衝突する関数の識別子を
      宣言するのに使えてしまうからです。この不適合例は、``convert`` という名前のマングルされない関数を
      宣言していますが、Rust 2024 で必要な unsafe のラッパがありません。

      この不適合例をコンパイルするには、Rust 2021 エディション以前が必要です。
      Rust 2024 エディションでは、``unsafe`` のラッパがないとコンパイルエラーになります。

      .. rust-example::
          :compile_fail:
          :edition: 2024

          // 別の 'convert' 関数が定義されていると、
          // リンカやローダによる未定義動作が起こりうる。
          #[no_mangle]
          fn convert() {}

          fn main() {
              convert();
          }

   .. compliant_example::
      :id: compl_ex_wR1FEyLRKmrr
      :status: draft

      この適合例に示すように、Rust 2024 エディションでは ``no_mangle`` アトリビュートに
      ``unsafe`` キーワードが必須になります。

      注意: ``convert`` 関数のシンボルが複数回定義されている場合、このコードでも
      未定義動作が起こりえます。

      .. rust-example::
          :miri: skip
          :edition: 2024

          #[unsafe(no_mangle)] // 適合
          fn convert() {}

          fn main() {
              convert();
          }

   .. non_compliant_example::
      :id: non_compl_ex_FdmuPXGZr4EO
      :status: draft

      この不適合例は、``extern "C"`` ブロックの中で ``size`` パラメータの型を ``usize`` ではなく
      ``f32`` と指定しており、``malloc`` 関数を誤って宣言しています。

      ``extern`` ブロックは unsafe です。型や関数を誤って宣言すると、未定義動作が起こりうるからです。
      これは宣言が使われていない場合でも当てはまります。この不適合例をコンパイルするには、
      Rust 2021 エディション以前が必要です。

      .. rust-example::
          :compile_fail:
          :edition: 2024

          use std::ffi;

          extern "C" {
              // 'malloc' が別の場所で 'usize' の引数で定義されている場合、コンパイラは
              // この互換性のない宣言を使ってこの関数の呼び出しのコードを生成することがあり、
              // その結果、未定義動作になる。
              fn malloc(size: f32) -> *mut ffi::c_void;
          }

          fn main() {}

   .. compliant_example::
      :id: compl_ex_wR1FEyLRKmrq
      :status: draft

      この適合例に示すように、Rust 2024 エディションでは ``extern "C"`` ブロックに
      ``unsafe`` キーワードが必須になります。

      注意: 宣言した ``malloc`` 関数が実際の定義と互換性がない場合、このコードでも
      未定義動作が起こりえます。この未定義動作をなくすため、この適合例で使っている
      ``malloc`` の宣言は ``usize`` 型の引数を 1 つ受け取ります。

      .. rust-example::
          :miri: skip
          :edition: 2024

          use std::ffi;

          unsafe extern "C" {
              // ここでは、malloc は C の stdlib.h で定義されたものであり、
              // size_of::<usize>() == size_of::<size_t>() であると仮定している
              fn malloc(size: usize) -> *mut ffi::c_void;
              fn free(ptr: *mut ffi::c_void);
          }

          fn main() {
              unsafe {
                  let ptr = malloc(1024);
                  if !ptr.is_null() {
                      free(ptr);
                  }
              }
          }

   .. non_compliant_example::
      :id: non_compl_ex_Hk3mNp5qRs7t
      :status: draft

      ``#[export_name]`` と ``#[link_section]`` アトリビュートは、リンク時のシンボル解決と
      メモリレイアウトに影響するため、誤って使うと未定義動作を引き起こすことがあります。
      ``unsafe`` キーワードがなければ、これらの危険はレビュアやツールから見えません。

      この不適合例には、別々の問題が 2 つあります。1 つ目は、unsafe のラッパなしで ``#[export_name]``
      アトリビュートを使っていることです。このアトリビュートは、リンク時に使われるシンボル名を
      制御します。同じ名前の別のシンボルが存在すると、未定義動作を引き起こします。
      Rust 2024 では、これに ``unsafe`` の印を付ける必要があります。

      2 つ目の問題は、この不適合例が unsafe のラッパなしで ``#[link_section]`` アトリビュートを
      使っていることです。このアトリビュートは、アイテムを特定のリンカセクションに配置します。
      セクションの配置を誤ると、未定義動作を引き起こすことがあります（例: 可変なデータを
      読み取り専用セクションに置く、``.init`` のような特別なセクションに干渉する）。

      この不適合例をコンパイルするには、Rust 2021 エディション以前が必要です。

      .. rust-example::
          :compile_fail:
          :edition: 2024

          // C ライブラリの 'printf' 関数と衝突する
          #[export_name = "printf"]  // 不適合

          // unsafe の印がない - Rust 2024 では不適合
          #[link_section = ".init_array"] // 不適合
          static DATA: u32 = 42;  // 初期化テーブルを壊す！

          fn main() {
              println!("DATA = {DATA}");
          }

   .. compliant_example::
      :id: compl_ex_xY2zAb3cDe4f
      :status: draft

      ``#[export_name]`` と ``#[link_section]`` アトリビュートは、安全性への影響が見えるように
      ``unsafe()`` のラッパを使わなければなりません。

      .. rust-example::
          :miri: skip
          :edition: 2024

          // SAFETY: 'custom_symbol' は他のどのシンボルとも衝突しない
          #[unsafe(export_name = "custom_symbol")]
          pub fn my_function() {}

          // SAFETY: 組み込みシステム向けに、データを特定のセクションに配置する
          #[unsafe(link_section = ".noinit")]
          static mut PERSISTENT_DATA: [u8; 256] = [0; 256];

          // SAFETY: 共有メモリ用の独自セクション
          #[unsafe(link_section = ".shared")]
          static SHARED_BUFFER: [u8; 4096] = [0; 4096];

          fn main() {
              my_function();
              println!("shared buffer size = {}", SHARED_BUFFER.len());
              unsafe { let _ = PERSISTENT_DATA[0]; }
          }

   **強制**
   このガイドラインは、次の仕組みによって強制できます。

   * **Rust 2024 エディション**: Rust 2024 エディションに移行すると、``extern`` ブロックと
     unsafe なアトリビュートについて、このガイドラインへの違反がコンパイルエラーになります。

   * **コンパイラのリント**: 次のリントを有効にします。

     * ``#![deny(unsafe_code)]`` - すべての unsafe なコードを拒否します（正当な例外には ``#[allow(unsafe_code)]`` を使います）
     * ``#![deny(unsafe_op_in_unsafe_fn)]`` - unsafe な関数の中で明示的な unsafe ブロックを要求します
     * ``#![warn(unsafe_attr_outside_unsafe)]`` - ``unsafe()`` のラッパのない unsafe なアトリビュートについて警告します（2024 より前）

   * **静的解析ツール**:

     * ``cargo-geiger`` - unsafe なコードの使用を数えて報告します
     * ``cargo-audit`` - 依存関係にある既知の脆弱性を検査します
     * プロジェクト固有の要件のための独自の Clippy リント

   * **コードレビュー**: ``unsafe`` トークンを含むすべてのコードを手作業でレビューすることを
     開発プロセスに組み込み、使用箇所ごとに正当化の理由を文書化するべきです。

   **関連するガイドライン**
   * unsafe ブロックのスコープを最小限にする
   * すべての unsafe なコードについて、安全性の不変条件を ``// SAFETY:`` コメントで文書化する
   * 生の unsafe なコードよりも安全な抽象化を優先する
   * 可能な場合はクレートレベルで ``#![forbid(unsafe_code)]`` を使い、例外は明示する

   .. bibliography::
      :id: bib_n8YJHvQf4mWx
      :status: draft

      .. list-table::
         :header-rows: 0
         :widths: auto
         :class: bibliography-table

         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-EDITION-GUIDE`
           - The Rust Edition Guide. "Rust 2024." https://doc.rust-lang.org/edition-guide/rust-2024/index.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-REF-UNSAFE-KEYWORD`
           - The Rust Reference. "Unsafe Keyword." https://doc.rust-lang.org/reference/unsafe-keyword.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-LINT-UNSAFE`
           - Rust Compiler Lint Documentation. "unsafe_code." https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#unsafe-code.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUSTNOMICON-MEET-SAFE`
           - The Rustonomicon. "Meet Safe and Unsafe." https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:ISO-26262`
           - International Organization for Standardization. "ISO 26262 - Road vehicles - Functional safety." https://www.iso.org/standard/68383.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:DO-178C`
           - RTCA, Inc. "DO-178C: Software Considerations in Airborne Systems and Equipment Certification." https://store.accuristech.com/standards/rtca-do-178c.
         * - :bibentry:`gui_ZDLZzjeOwLSU:CARGO-GEIGER`
           - cargo-geiger contributors. "cargo-geiger: Detects usage of unsafe Rust." https://github.com/geiger-rs/cargo-geiger.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-REF-EXTERN`
           - The Rust Reference. "External blocks." https://doc.rust-lang.org/reference/items/external-blocks.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-REF-UNSAFE-ATTR`
           - The Rust Reference. "Unsafe attributes." https://doc.rust-lang.org/reference/attributes.html#unsafe-attributes.
         * - :bibentry:`gui_ZDLZzjeOwLSU:FLS`
           - The Rust Project. "FLS." https://rust-lang.github.io/fls/.
         * - :bibentry:`gui_ZDLZzjeOwLSU:RUST-REF-UNION`
           - The Rust Reference. "Unions." https://doc.rust-lang.org/reference/items/unions.html.
         * - :bibentry:`gui_ZDLZzjeOwLSU:UCG-VALIDITY`
           - Rust Unsafe Code Guidelines. "Validity and Safety Invariant." https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#validity-and-safety-invariant.
