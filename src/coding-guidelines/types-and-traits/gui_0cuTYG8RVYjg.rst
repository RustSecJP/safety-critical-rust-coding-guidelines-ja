.. SPDX-License-Identifier: MIT OR Apache-2.0
   SPDX-FileCopyrightText: The Coding Guidelines Subcommittee Contributors

.. default-domain:: coding-guidelines

共用体フィールドの読み取りがそのフィールドの型として有効な値を生成するようにする
================================================================================

.. guideline:: 共用体フィールドの読み取りがそのフィールドの型として有効な値を生成するようにする
   :id: gui_0cuTYG8RVYjg
   :category: required
   :status: draft
   :release: unknown
   :fls: fls_oFIRXBPXu6Zv
   :decidability: undecidable
   :scope: system
   :tags: defect, safety, undefined-behavior

   共用体フィールドから読み取るときは、基になるバイト列がそのフィールドの型として有効な値になっていることを保証してください。
   バイト列がフィールドの型として有効な値を表していない共用体フィールドを読み取ることは、未定義動作です。

   共用体フィールドにアクセスする前に、その共用体が次のいずれかであることを確認してください。

   * 最後にそのフィールドを通じて書き込まれた
   * 対象のフィールドの型として再解釈したときに有効なバイト列を持つフィールドを通じて書き込まれた

   アクティブなフィールドが不確かな場合は、明示的な有効性検査を使ってください。

   .. rationale::
      :id: rat_8QeimyAvM7cH
      :status: draft

      C と同様に、共用体では複数のフィールドが同じメモリを占めることができます。
      列挙型とは異なり、共用体はどのフィールドが現在アクティブかを追跡しません。
      フィールドを読み取るときには、基になるバイト列が
      そのフィールドの型として有効であることを保証しなければなりません :cite:`gui_0cuTYG8RVYjg:RUST-REF-UNION`。

      すべての型には *有効性不変条件* があります。これは、その型のすべての値が
      満たさなければならない制約の集まりです :cite:`gui_0cuTYG8RVYjg:UCG-VALIDITY`。
      共用体フィールドの読み取りは *型付きの読み取り* を行います。
      これは、バイト列が対象の型として有効であることを表明するものです。

      よく使われる型の有効性の要件の例を次に示します。

      * **bool**: ``0``（false）か ``1``（true）でなければなりません。それ以外の値（例: ``3``）は無効です。
      * **char**: 有効な Unicode スカラー値（``0x0`` から ``0xD7FF``、または ``0xE000`` から ``0x10FFFF``）でなければなりません。
      * **参照**: ヌルでなく、適切にアラインされていなければなりません。
      * **列挙型**: 有効な判別子の値を保持していなければなりません。
      * **浮動小数点数**: ``f32`` 型や ``f64`` 型では、すべてのビットパターンが有効です。
      * **整数**: 整数型では、すべてのビットパターンが有効です。

      無効な値を読み取ることは、未定義動作です。

   .. non_compliant_example::
      :id: non_compl_ex_ecHYRXb4Ncpu
      :status: draft

      この不適合例は、真偽値の共用体フィールドから無効なビットパターンを読み取ります。
      値 ``3`` は ``bool`` 型の有効な値ではありません（``0`` と ``1`` だけが有効です）。

      .. rust-example::
         :miri: expect_ub

         union IntOrBool {
             i: u8,
             b: bool,
         }

         fn main() {
             let u = IntOrBool { i: 3 };

             // 'bool' 型の共用体フィールドから無効な値を読み取る未定義動作
             unsafe { u.b };  // 不適合
         }

   .. non_compliant_example::
      :id: non_compl_ex_8bloNOcsLEKX
      :status: draft

      この不適合例は、``char`` 型の ``union`` フィールドから無効な Unicode 値を読み取ります。

      .. rust-example::
         :miri: expect_ub

         union IntOrChar {
             i: u32,
             c: char,
         }

         fn main() {
             // '0xD800' はサロゲートであり、有効な Unicode スカラー値ではない
             let u = IntOrChar { i: 0xD800 };

             // 'char' 型の共用体フィールドから無効な Unicode 値を読み取る
             unsafe { u.c };  // 不適合
         }

   .. non_compliant_example::
      :id: non_compl_ex_PsJAB4WglRZl
      :status: draft

      この不適合例は、'Color' 列挙型の共用体フィールドから無効な判別子を読み取ります。

      .. rust-example::
         :miri: expect_ub

         #[repr(u8)]
         #[derive(Copy, Clone)]
         #[allow(dead_code)]
         enum Color {
             Red = 0,
             Green = 1,
             Blue = 2,
         }

         union IntOrColor {
             i: u8,
             c: Color,
         }

         fn main() {
             let u = IntOrColor { i: 42 };

             // 'Color' 列挙型から無効な判別子を読み取る未定義動作
             unsafe { u.c };  // 不適合
         }

   .. non_compliant_example::
      :id: non_compl_ex_aEx4HnDD8xIp
      :status: draft

      この不適合例は、ヌルポインタを含む共用体から参照を読み取ります。
      アラインされていないポインタを読み取る場合にも、同様の問題が起こります。

      .. rust-example::
         :miri: expect_ub

         union PtrOrRef {
             p: *const i32,
             r: &'static i32,
         }

         fn main() {
             let u = PtrOrRef { p: std::ptr::null() };

             //  共用体の参照フィールドからヌル値を読み取る未定義動作
             unsafe { u.r };  // 不適合
         }

   .. compliant_example::
      :id: compl_ex_x27meeLDMZNI
      :status: draft

      この適合例は、有効な読み取りを保証するために、アクティブなフィールドを明示的に追跡します。

      .. rust-example::
         :miri:

         #[repr(C)]
         #[derive(Copy, Clone)]
         union IntOrBoolData {
             i: u8,
             b: bool,
         }

         /// 共用体のどのフィールドが現在アクティブかを追跡する。
         #[derive(Clone, Copy, PartialEq, Eq)]
         enum ActiveField {
             Int,
             Bool,
         }

         /// 実行時にアクティブなフィールドを追跡する共用体のラッパ。
         pub struct IntOrBool {
             data: IntOrBoolData,
             active: ActiveField,
         }

         impl IntOrBool {
             pub fn from_int(value: u8) -> Self {
                 Self {
                     data: IntOrBoolData { i: value },
                     active: ActiveField::Int,
                 }
             }

             pub fn from_bool(value: bool) -> Self {
                 Self {
                     data: IntOrBoolData { b: value },
                     active: ActiveField::Bool,
                 }
             }

             pub fn set_int(&mut self, value: u8) {
                 self.data.i = value;
                 self.active = ActiveField::Int;
             }

             pub fn set_bool(&mut self, value: bool) {
                 self.data.b = value;
                 self.active = ActiveField::Bool;
             }

             /// そのフィールドがアクティブなら整数値を返す。
             pub fn as_int(&self) -> Option<u8> {
                 match self.active {
                     // SAFETY: 最後に `i` として書き込まれたとわかっているときにだけ `i` を読み取る
                     ActiveField::Int => Some(unsafe { self.data.i }), // 適合
                     ActiveField::Bool => None,
                 }
             }

             /// そのフィールドがアクティブなら真偽値を返す。
             pub fn as_bool(&self) -> Option<bool> {
                 match self.active {
                     // SAFETY: 最後に `b` として書き込まれたとわかっているときにだけ `b` を読み取る
                     ActiveField::Bool => Some(unsafe { self.data.b }), // 適合
                     ActiveField::Int => None,
                 }
             }
         }

         fn main() {
             let mut value = IntOrBool::from_bool(true);
             assert_eq!(value.as_bool(), Some(true));
             assert_eq!(value.as_int(), None);

             value.set_int(42);
             assert_eq!(value.as_bool(), None);
             assert_eq!(value.as_int(), Some(42));
         }

   .. compliant_example::
      :id: compl_ex_Y7xaYuD2xdmq
      :status: draft

      この適合例は、書き込んだのと同じフィールドから読み取ります。

      .. rust-example::
         :miri:

         #[repr(C)]
         #[derive(Copy, Clone)]
         union IntBytes {
             i: u32,
             bytes: [u8; 4],
         }

         fn get_int() -> u32 {
             let u = IntBytes { i: 0x12345678 };

             // SAFETY: [u8; 4] ではすべてのビットパターンが有効である
             // 注意: バイト順はターゲットのエンディアンに依存する
             assert_eq!(unsafe { u.bytes }, 0x12345678_u32.to_ne_bytes()); // 適合

             let u2 = IntBytes {
                 bytes: [0x11, 0x22, 0x33, 0x44],
             };

             // SAFETY: 'u32' ではすべてのビットパターンが有効である
             assert_eq!(unsafe { u2.i }, u32::from_ne_bytes([0x11, 0x22, 0x33, 0x44])); // 適合

             unsafe { u2.i } // 適合
         }

         fn main() {
            println!("{}", get_int());
         }

   .. compliant_example::
      :id: compl_ex_Jsxenev7lNf0
      :status: draft

      この適合例は、すべてのビットパターンが有効な別の型として値を再解釈します。

      .. rust-example::
         :miri:

         #[repr(C)]
         #[derive(Copy, Clone)]
         union IntBytes {
             i: u32,
             bytes: [u8; 4],
         }

         fn get_bytes() -> [u8; 4] {
             let u = IntBytes { i: 0x12345678 };

             // SAFETY: '[u8; 4]' ではすべてのビットパターンが有効である
             // 注意: バイト順はターゲットのエンディアンに依存する
             assert_eq!(unsafe { u.bytes }, 0x12345678_u32.to_ne_bytes()); // 適合
             unsafe { u.bytes }  // 適合
         }

         fn get_u32() -> u32 {
             let u = IntBytes {
                 bytes: [0x11, 0x22, 0x33, 0x44],
             };

             // SAFETY: 'u32' ではすべてのビットパターンが有効である
             assert_eq!(unsafe { u.i }, u32::from_ne_bytes([0x11, 0x22, 0x33, 0x44])); // 適合
             unsafe { u.i }  // 適合
         }

         fn main() {
             println!("{:#04x?}", get_bytes());
             println!("{}", get_u32());
         }

   .. compliant_example::
      :id: compl_ex_vIITtPAeKHrp
      :status: draft

      この適合例は、制約のある型として読み取る前にバイト列を検証します。

      .. rust-example::
         :miri:

         #[repr(C)]
         union IntOrBool {
             i: u8,
             b: bool,
         }

         fn try_read_bool(u: &IntOrBool) -> Option<bool> {
             // SAFETY: `u8` ではすべてのビットパターンが有効なので、どのフィールドが
             // 最後に書き込まれたかに関係なく、`u8` としての読み取りは常に有効である。
             let raw = unsafe { u.i }; // 適合

             // `bool` として解釈する前に検証する（0 と 1 だけが有効）
             match raw {
                 0 => Some(false),
                 1 => Some(true),
                 _ => None,
             } // 適合
         }

         fn main() {
             let u1 = IntOrBool { i: 1 };
             let u2 = IntOrBool { i: 3 };

             assert_eq!(try_read_bool(&u1), Some(true));
             assert_eq!(try_read_bool(&u2), None);
         }

   .. compliant_example::
      :id: compl_ex_4Z8tmqYLLjtw
      :status: draft

      次のことを示す複雑な例です。

      * ジェネリクスを使った、有効な型のコンパイル時検査の使い方
      * FFI に面したコードと、安全な Rust のコードベースの残りの部分とを隔てる方法

      .. rust-example::
         :miri:

         use std::marker::PhantomData;
         use std::mem::size_of;

         /// アクティブなフィールドを表すマーカ型。
         pub struct AsInt;
         pub struct AsBool;

         /// FFI 境界を越えたやり取りに使える共用体型。
         #[repr(C)]
         #[derive(Copy, Clone)]
         pub union IntOrBoolData {
             pub i: u8,
             pub b: bool,
         }

         /// C のコードから共用体と一緒に送られるタグ。
         #[repr(u8)]
         #[derive(Copy, Clone, PartialEq, Eq)]
         pub enum IntOrBoolTag {
             Int = 0,
             Bool = 1,
         }

         /// FFI から届きうる、C 互換のタグ付き共用体。
         #[repr(C)]
         #[derive(Copy, Clone)]
         pub struct CIntOrBool {
             pub tag: IntOrBoolTag,
             pub data: IntOrBoolData,
         }

         // ============================================================================
         // Rust のコードベースの残りの部分で使う安全なラッパ型
         // ============================================================================

         /// 型パラメータがアクティブなフィールドを静的に追跡する共用体のラッパ。
         /// これはゼロコストである: 生の共用体と同じサイズ。
         #[repr(C)]
         pub struct IntOrBool<T> {
             data: IntOrBoolData,
             _marker: PhantomData<T>,
         }

         impl IntOrBool<AsInt> {
             pub fn from_int(value: u8) -> Self {
                 Self {
                     data: IntOrBoolData { i: value },
                     _marker: PhantomData,
                 }
             }

             pub fn get(&self) -> u8 {
                 // SAFETY: 型パラメータ `AsInt` が、整数フィールドがアクティブであることを保証する
                 unsafe { self.data.i }
             }

             /// 真偽値の表現に変換する。
             /// 整数値が 0 か 1 のときにだけ有効。
             pub fn try_into_bool(self) -> Option<IntOrBool<AsBool>> {
                 match self.get() {
                     0 | 1 => Some(IntOrBool {
                         data: IntOrBoolData { b: self.get() == 1 },
                         _marker: PhantomData,
                     }),
                     _ => None,
                 }
             }
         }

         impl IntOrBool<AsBool> {
             pub fn from_bool(value: bool) -> Self {
                 Self {
                     data: IntOrBoolData { b: value },
                     _marker: PhantomData,
                 }
             }

             pub fn get(&self) -> bool {
                 // SAFETY: 型パラメータ `AsBool` が、真偽値フィールドがアクティブであることを保証する
                 unsafe { self.data.b }
             }

             /// 整数の表現に変換する。bool は u8 の部分集合なので常に有効。
             pub fn into_int(self) -> IntOrBool<AsInt> {
                 IntOrBool {
                     data: self.data,
                     _marker: PhantomData,
                 }
             }
         }

         // ============================================================================
         // FFI 境界: C の表現から安全な Rust の型に変換する
         // ============================================================================

         /// C のタグ付き共用体を安全な Rust の型に変換した結果。
         /// 呼び出し側は両方のバリアントを処理しなければならず、これにより型安全性が保証される。
         pub enum SafeIntOrBool {
             Int(IntOrBool<AsInt>),
             Bool(IntOrBool<AsBool>),
         }

         impl CIntOrBool {
             /// FFI 境界で、C の表現から安全な Rust の型に変換する。
             /// これ以降、すべてのコードは型安全なラッパを使う。
             pub fn into_safe(self) -> SafeIntOrBool {
                 match self.tag {
                     IntOrBoolTag::Int => {
                         // SAFETY: タグが、整数フィールドがアクティブであることを保証する
                         let value = unsafe { self.data.i };
                         SafeIntOrBool::Int(IntOrBool::from_int(value))
                     }
                     IntOrBoolTag::Bool => {
                         // SAFETY: タグが、真偽値フィールドがアクティブであることを保証する
                         let value = unsafe { self.data.b };
                         SafeIntOrBool::Bool(IntOrBool::from_bool(value))
                     }
                 }
             }
         }

         // ============================================================================
         // FFI 境界: 安全な Rust の型から C の表現に戻す
         // ============================================================================

         impl From<IntOrBool<AsInt>> for CIntOrBool {
             fn from(val: IntOrBool<AsInt>) -> Self {
                 CIntOrBool {
                     tag: IntOrBoolTag::Int,
                     data: IntOrBoolData { i: val.get() },
                 }
             }
         }

         impl From<IntOrBool<AsBool>> for CIntOrBool {
             fn from(val: IntOrBool<AsBool>) -> Self {
                 CIntOrBool {
                     tag: IntOrBoolTag::Bool,
                     data: IntOrBoolData { b: val.get() },
                 }
             }
         }

         // ============================================================================
         // 例: 安全な型を使うアプリケーションのコード
         // ============================================================================

         /// 真偽値を処理する。この関数は IntOrBool<AsBool> しか受け取れない
         /// ので、無効な bool のバイト列を読み取る可能性はない。
         fn process_bool(val: IntOrBool<AsBool>) -> &'static str {
             if val.get() { "yes" } else { "no" }
         }

         /// 整数値を処理する。
         fn process_int(val: IntOrBool<AsInt>) -> u8 {
             val.get().saturating_mul(2)
         }

         // 通常は C で定義される FFI 関数を模したもの。
         // 実際のコードでは、これらは C ライブラリにリンクされる `extern "C"` 宣言になる。

         /// C からデータを「受け取る」C 関数を模したもの。
         extern "C" fn receive_from_ffi() -> CIntOrBool {
             CIntOrBool {
                 tag: IntOrBoolTag::Bool,
                 data: IntOrBoolData { b: true },
             }
         }

         /// C にデータを「送る」C 関数を模したもの。
         extern "C" fn send_to_ffi(data: CIntOrBool) {
             // 実際のコードでは、これは C で実装される
             match data.tag {
                 IntOrBoolTag::Int => {
                     let i = unsafe { data.data.i };
                     assert_eq!(i, 84);
                 }
                 IntOrBoolTag::Bool => {
                     let b = unsafe { data.data.b };
                     assert!(b);
                 }
             }
         }

         fn main() {
             // ゼロコストであることを証明する: PhantomData はサイズを増やさない
             assert_eq!(size_of::<IntOrBoolData>(), size_of::<IntOrBool<AsInt>>());
             assert_eq!(size_of::<IntOrBoolData>(), size_of::<IntOrBool<AsBool>>());
             assert_eq!(size_of::<IntOrBoolData>(), 1); // わずか 1 バイト

             // === FFI 境界: C から受け取る ===
             let from_c = receive_from_ffi();
             let safe_value = from_c.into_safe();

             // === アプリケーションのコード: 完全に型安全で、unsafe なし ===
             match safe_value {
                 SafeIntOrBool::Bool(b) => {
                     // process_bool は IntOrBool<AsBool> でしか呼び出せない
                     assert_eq!(process_bool(b), "yes");
                 }
                 SafeIntOrBool::Int(i) => {
                     // process_int は IntOrBool<AsInt> でしか呼び出せない
                     let _ = process_int(i);
                 }
             }

             // === Rust 内での型安全な変換 ===
             let int_val = IntOrBool::from_int(1);

             // IntOrBool<AsInt> を process_bool に渡すことはできない - コンパイルできない:
             // process_bool(int_val); // エラー: IntOrBool<AsBool> を期待したが、IntOrBool<AsInt> が見つかった

             // 明示的に変換しなければならず、その際に値が検証される
             if let Some(bool_val) = int_val.try_into_bool() {
                 assert_eq!(process_bool(bool_val), "yes");
             }

             // 無効な変換は変換の時点で捕捉される
             let int_val = IntOrBool::from_int(42);
             assert!(int_val.try_into_bool().is_none()); // 42 は有効な bool ではない

             // === FFI 境界: C に送り返す ===
             let int_val = IntOrBool::from_int(42);
             let doubled = IntOrBool::from_int(process_int(int_val));
             send_to_ffi(doubled.into());
         }
   .. bibliography::
      :id: bib_WNCi5njUWLuZ
      :status: draft

      .. list-table::
         :header-rows: 0
         :widths: auto
         :class: bibliography-table

         * - :bibentry:`gui_0cuTYG8RVYjg:RUST-REF-UNION`
           - The Rust Reference. "Unions." https://doc.rust-lang.org/reference/items/unions.html

         * - :bibentry:`gui_0cuTYG8RVYjg:UCG-VALIDITY`
           - Rust Unsafe Code Guidelines. "Validity and Safety Invariant." https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#validity-and-safety-invariant.
