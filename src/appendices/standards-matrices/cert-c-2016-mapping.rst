.. _cert_c_mapping:

CERT-C 2016 と Rust の相互参照
=======================================

以下の表は、CERT-C 2025 のガイドラインと、それらの Rust への適用可能性との相互参照です。1 つ目の表は Rust 全般に適用されるガイドラインを扱います。2 つ目の表は、``unsafe`` なコードが存在する場合に追加で適用されるガイドラインを扱います。3 つ目の表は、Rust に適用されないガイドラインの一覧です。


表 1 – Rust 全般に適用されるガイドライン（安全な Rust、unsafe なコードが存在しない場合）
----------------------------------------------------------------------------------------------------

.. list-table::
   :widths: 14 14 14 14 14
   :header-rows: 1

   * - ガイドライン
     - タイトル
     - カテゴリ
     - セーフティクリティカル Rust ルール
     - コメント
   * - `FLP37 <https://wiki.sei.cmu.edu/confluence/display/c/FLP37-C.+Do+not+use+object+representations+to+compare+floating-point+values>`__
     - 浮動小数点数値の比較にオブジェクト表現を使用しない
     -
     -
     - これがどこで意図的に行われうるのかは、すぐにはわからないかもしれません。浮動小数点数を使う自己完結したシステムで起こるとは、まず考えにくいことです。しかし、 **生の浮動小数点データを受け取るとき** には、「とりあえずビットを比較する」ことに強く惹かれることがあります。さらに悪いことに、 **この方法は一部の値ではうまく動く** ため、正しくないことが隠れてしまうかもしれません。網羅性のために、エンコーディングは異なるのに標準のもとでは等しいと比較される FP 値の例を挙げるとよいでしょう。ここで最も単純な例は、おそらく ``0`` と ``-0`` です。これらはビット単位では異なると比較されますが、IEEE FP 標準のもとでは等しいとみなされます。 **追加のアイデア:** エンコーディングが同じでも ``NaN != NaN`` となるので、 ``NaN`` に関する浮動小数点数の等価性のガイドラインがあるとよいかもしれません。「この値は ``NaN`` か？」という問いに答えるには、
       ```is_nan`` <https://doc.rust-lang.org/std/primitive.f32.html?search=is_nan>`__
       メソッドがあります。
   * - `DCL39 <https://wiki.sei.cmu.edu/confluence/display/c/DCL39-C.+Avoid+information+leakage+when+passing+a+structure+across+a+trust+boundary>`__
     - 信頼境界を越えて構造体を渡すときに情報漏えいを避ける
     -
     -
     - Unsafe を使えば、技術的には実現できるようです。この議論の Zulip スレッド: `How should padding bits be
       erased for
       sending? <https://rust-lang.zulipchat.com/#narrow/channel/136281-t-opsem/topic/How.20should.20padding.20bits.20be.20erased.20for.20sending.3F/with/564800067>`__\ 重要な問題は、コピーされる構造体の未初期化ビットにあるかもしれない機密データをコピーせずに、信頼できないバッファへコピーすることのようです。構造体と列挙型については、内部のパディングビット以外のすべてをコピーする型を作ることで解決できそうです。共用体については、パディングをコピーしていないことを「本当に確実に」したく、できれば証明したいところです。共用体を安全に送ることは不可能かもしれません。\ *自分のプロセス内の機密メモリをゼロクリアし、プロセス終了後に機密情報が何も残らないようにしたい場合もあるでしょう。* **それは私たちの作業の範囲外です。**
       *これに関する Zulip スレッド:* `Are bits erased on drop? If not, how can
       I ensure they
       are? <https://rust-lang.zulipchat.com/#narrow/channel/122651-general/topic/Are.20bits.20erased.20on.20drop.3F.20If.20not.2C.20how.20can.20I.20ensure.20they.20are.3F/with/564800542>`__
   * - `INT36 <https://wiki.sei.cmu.edu/confluence/display/c/INT36-C.+Converting+a+pointer+to+integer+or+integer+to+pointer>`__
     - ポインタから整数へ、または整数からポインタへの変換
     -
     -
     - 修正を加えて: - ptr2int: 比較的重要ではないとみなす - int2ptr: 注意を促す\ **ポインタの来歴 (Pointer Provenance)** がこれに関係します。\ *あわせて参照:* `transmute: caution against
       int2ptr
       transmutation <http://github.com/rust-lang/rust/pull/122379>`__\ unsafe を多用する扱いになりうるため、精査に値するかもしれません
   * - `CON40 <https://wiki.sei.cmu.edu/confluence/display/c/CON40-C.+Do+not+refer+to+an+atomic+variable+twice+in+an+expression>`__
     - 1 つの式の中でアトミック変数を 2 回参照しない
     -
     -
     -
   * - `CON36 <https://wiki.sei.cmu.edu/confluence/display/c/CON36-C.+Wrap+functions+that+can+spuriously+wake+up+in+a+loop>`__
     - 見せかけの起床 (spurious wake-up) が起こりうる関数はループで囲む
     -
     -
     - これは
       ```std::sync::Condvar`` <https://doc.rust-lang.org/std/sync/struct.Condvar.html>`__
       の使い方、特にその
       ```wait`` <https://doc.rust-lang.org/std/sync/struct.Condvar.html#method.wait>`__
       メソッドと
       ```wait_timeout`` <https://doc.rust-lang.org/std/sync/struct.Condvar.html#method.wait_timeout>`__
       メソッドを使う場合に直接対応します。この見せかけの起床を正しくループで囲むために、それぞれ
       ```wait_while`` <https://doc.rust-lang.org/std/sync/struct.Condvar.html#method.wait_while>`__
       と
       ```wait_timeout_while`` <https://doc.rust-lang.org/std/sync/struct.Condvar.html#method.wait_timeout_while>`__
       というメソッドが用意されており、おそらくこちらを代わりに推奨すべきでしょう。
   * - `EXP40 <https://wiki.sei.cmu.edu/confluence/display/c/EXP40-C.+Do+not+modify+constant+objects>`__
     - 定数オブジェクトを変更しない
     -
     -
     - ``const`` アイテムは変更できません。これらは実際には使用箇所ごとにインライン化されるので、変更しても何の効果もありません。したがって、Rust の場合、これは ``const`` についての話ではありません。一方、 ``static`` アイテムは変更できますが、 ``unsafe`` なコードの中だけです。これは設計どおりです。「特別な場合に可変性を持つグローバルなアイテム」であるべきものもあるからです。ただし、 ``static`` 変数と ``static mut`` 変数は区別しなければなりません。
       `こちら <https://github.com/rust-lang/unsafe-code-guidelines/issues/269>`__
       で述べられているとおり、後者は非常に UB を起こしやすいからです。 ``static`` 変数は、
       `内部可変性パターン <https://doc.rust-lang.org/book/ch15-05-interior-mutability.html>`__
       を使えば、ずっと簡単に（それでも十分な注意は必要です！）使え、グローバルな可変状態のユースケースの大部分をカバーできます。\ *私の現在の考えでは、 ``static mut`` は Mandatory ルール（つまり例外なし）として、あるいはその安全性について十分に文書化された理由づけを伴う Required ルールとして、禁止すべきです。忘れないでください:* `usage of ``static mut`` very easily results
       in
       UB. <https://doc.rust-lang.org/nightly/edition-guide/rust-2024/static-mut-references.html#safe-references>`__\ \ *``static`` については、それを避けるよう推奨する Advisory ルールがあるとよいと思います。そうしたルールで、 ``static`` を安全に使う方法の例も示せるとよいでしょう。*\  **STR30 と対にする**
   * - `STR30 <https://wiki.sei.cmu.edu/confluence/display/c/STR30-C.+Do+not+attempt+to+modify+string+literals>`__
     - 文字列リテラルを変更しようとしない
     -
     -
     - Rust への対応付けにおいては **EXP40 と統合** して扱う
   * - `INT30 <https://wiki.sei.cmu.edu/confluence/display/c/INT30-C.+Ensure+that+unsigned+integer+operations+do+not+wrap>`__
     - 整数演算がラップアラウンドしないことを保証する
     -
     -
     - 実装済み。対応付けとカテゴリは引き続き要レビュー
   * - `INT33 <https://wiki.sei.cmu.edu/confluence/display/c/INT33-C.+Ensure+that+division+and+remainder+operations+do+not+result+in+divide-by-zero+errors>`__
     - 除算と剰余演算がゼロ除算エラーにならないことを保証する
     -
     -
     - 実装済み。対応付けとカテゴリは引き続き要レビュー
   * - `ARR30 <https://wiki.sei.cmu.edu/confluence/display/c/ARR30-C.+Do+not+form+or+use+out-of-bounds+pointers+or+array+subscripts>`__
     - 範囲外を指すポインタや配列の添字を作ったり使ったりしない
     -
     -
     - 安全な Rust では: - 直接のアドレス指定は範囲外ならパニックする - arr.get(index) は範囲外なら None を返す。unsafe な Rust では: - arr.get_unchecked(index) は範囲外なら UB。したがって、範囲外のインデックス指定は防ぐか、適切に処理すべきだと思います。unsafe なインデックス指定を使う場合は、形式的に検証すべきです。
   * - `FIO45 <https://wiki.sei.cmu.edu/confluence/display/c/FIO45-C.+Avoid+TOCTOU+race+conditions+while+accessing+files>`__
     - ファイルへのアクセス時に TOCTOU 競合状態を避ける
     -
     -
     - *根拠:*
       ```std::fs`` <https://doc.rust-lang.org/std/fs/index.html#time-of-check-to-time-of-use-toctou>`__
       は `TOCTOU について明示的に述べており <https://doc.rust-lang.org/std/fs/index.html#time-of-check-to-time-of-use-toctou>`__ 、
       *その避け方も説明しています。*
   * - `FLP30 <https://wiki.sei.cmu.edu/confluence/display/c/FLP30-C.+Do+not+use+floating-point+variables+as+loop+counters>`__
     - 浮動小数点変数をループカウンタとして使わない
     -
     -
     -
   * - `INT31 <https://wiki.sei.cmu.edu/confluence/display/c/INT31-C.+Ensure+that+integer+conversions+do+not+result+in+lost+or+misinterpreted+data>`__
     - 整数変換によってデータが失われたり誤って解釈されたりしないことを保証する
     -
     -
     - #185 で作業中。対応付けとカテゴリは引き続き要レビュー
   * - `FLP32 <https://wiki.sei.cmu.edu/confluence/display/c/FLP32-C.+Prevent+or+detect+domain+and+range+errors+in+math+functions>`__
     - 数学関数の定義域エラーと値域エラーを防止または検出する
     -
     -
     -
   * - `ENV33 <https://wiki.sei.cmu.edu/confluence/display/c/ENV33-C.+Do+not+call+system%28%29>`__
     - system() を呼び出さない
     -
     -
     - 対応付けは厳密ではありません（system() は関係しません）: コマンドを実行する場合、その入力は常にサニタイズしなければならず、その動作は移植可能にすべきです。このトピックに触れる `Clippy のリント <https://rust-lang.github.io/rust-clippy/master/index.html?search=command#suspicious_command_arg_space>`__
       があります。システムコマンドをどう呼び出してよいかを扱う一連のルールがおそらく必要でしょう。
   * - `MSC41 <https://wiki.sei.cmu.edu/confluence/display/c/MSC41-C.+Never+hard+code+sensitive+information>`__
     - 機密情報を決してハードコードしない
     -
     -
     -
   * - `MEM31 <https://wiki.sei.cmu.edu/confluence/display/c/MEM31-C.+Free+dynamically+allocated+memory+when+no+longer+needed>`__
     - 動的に割り当てたメモリは不要になったら解放する
     -
     -
     - Rust コンパイラは、ほとんどの場合これを自動的に強制します。安全な Rust における重要な例外は
       ```std::rc::Rc`` <https://doc.rust-lang.org/std/rc/struct.Rc.html>`__/```std::sync::Arc`` <https://doc.rust-lang.org/std/sync/struct.Arc.html>`__
       の循環です。ユーザは、対応する
       ```std::rc::Weak`` <https://doc.rust-lang.org/std/rc/struct.Weak.html>`__
       と
       ```std::sync::Weak`` <https://doc.rust-lang.org/std/sync/struct.Weak.html>`__
       を使って、これを避けるべきです。Rust では、メモリリークは（C での手動の ``malloc`` と ``free`` に比べて）比較的簡単に避けられますが、それでもなお起こりえます。さらに、Rust ではメモリリークは安全とみなされます。元のルールと同様に、私たちの対応付けは深刻度の低い欠陥を記述することになるでしょう。最悪の結果でも、メモリ枯渇によるサービス拒否だからです。
   * - `INT32 <https://wiki.sei.cmu.edu/confluence/display/c/INT32-C.+Ensure+that+operations+on+signed+integers+do+not+result+in+overflow>`__
     - 符号付き整数の演算がオーバーフローしないことを保証する
     -
     -
     - 実装済み。対応付けとカテゴリは引き続き要レビュー
   * - `INT34 <https://wiki.sei.cmu.edu/confluence/display/c/INT34-C.+Do+not+shift+an+expression+by+a+negative+number+of+bits+or+by+greater+than+or+equal+to+the+number+of+bits+that+exist+in+the+operand>`__
     - 負のビット数、またはオペランドのビット数以上のビット数で式をシフトしない
     -
     -
     - 実装済み。対応付けとカテゴリは引き続き要レビュー
   * - `CON33 <https://wiki.sei.cmu.edu/confluence/display/c/CON33-C.+Avoid+race+conditions+when+using+library+functions>`__
     - ライブラリ関数を使うときに競合状態を避ける
     -
     -
     -
   * - `CON41 <https://wiki.sei.cmu.edu/confluence/display/c/CON41-C.+Wrap+functions+that+can+fail+spuriously+in+a+loop>`__
     - 見せかけの失敗が起こりうる関数はループで囲む
     -
     -
     -
   * - `ARR37 <https://wiki.sei.cmu.edu/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object>`__
     - 配列ではないオブジェクトへのポインタに整数を加算または減算しない
     -
     -
     -
   * - `FLP34 <https://wiki.sei.cmu.edu/confluence/display/c/FLP34-C.+Ensure+that+floating-point+conversions+are+within+range+of+the+new+type>`__
     - 浮動小数点数の変換が変換先の型の範囲内に収まることを保証する
     -
     -
     -
   * - `FLP36 <https://wiki.sei.cmu.edu/confluence/display/c/FLP36-C.+Preserve+precision+when+converting+integral+values+to+floating-point+type>`__
     - 整数値を浮動小数点型に変換するときに精度を保つ
     -
     -
     -
   * - `FIO42 <https://wiki.sei.cmu.edu/confluence/display/c/FIO42-C.+Close+files+when+they+are+no+longer+needed>`__
     - 不要になったファイルは閉じる
     -
     -
     - これは人生にもあてはまる良い教訓だと思います。Rust では、こうしたリソースの扱いはずっと簡単です。その Drop 実装が自動的に閉じてくれるはずだからです。それでも心に留めておくのはよいことで、たとえば必要がないのにファイルに ``'static`` ライフタイムを持たせないようにしましょう。
   * - `CON38 <https://wiki.sei.cmu.edu/confluence/display/c/CON38-C.+Preserve+thread+safety+and+liveness+when+using+condition+variables>`__
     - 条件変数を使うときにスレッド安全性と活性を保つ
     -
     -
     - これまでのところ、次の推奨を行うという合意があります: -
       *できる限り ``Condvar`` を使わないことを検討する。*\ - *``Condvar`` を使い、活性が重要な場合は、 ``notify_one`` の代わりに ``notify_all`` を使うことを検討する。*\ Zulip での議論:
       `Recommending ``Condvar::notify_all`` instead of
       ``notify_one``? <https://rust-lang.zulipchat.com/#narrow/channel/122651-general/topic/Recommending.20.60Condvar.3A.3Anotify_all.60.20instead.20of.20.60notify_one.60.3F/with/572488048>`__\ より上位の方針レベルでの議論が予定されています
   * - `MSC32 <https://wiki.sei.cmu.edu/confluence/display/c/MSC32-C.+Properly+seed+pseudorandom+number+generators>`__
     - 擬似乱数生成器に適切なシードを与える
     -
     -
     -
   * - `MEM34 <https://wiki.sei.cmu.edu/confluence/display/c/MEM34-C.+Only+free+memory+allocated+dynamically>`__
     - 動的に割り当てたメモリだけを解放する
     -
     -
     -
   * - `CON35 <https://wiki.sei.cmu.edu/confluence/display/c/CON35-C.+Avoid+deadlock+by+locking+in+a+predefined+order>`__
     - あらかじめ決めた順序でロックしてデッドロックを避ける
     -
     -
     -

表 2 – unsafe なコードが存在する場合に Rust に適用されるガイドライン
----------------------------------------------------------------------------------------------------

.. list-table::
   :widths: 14 14 14 14 14
   :header-rows: 1

   * - ガイドライン
     - タイトル
     - カテゴリ
     - セーフティクリティカル Rust ルール
     - コメント
   * - `EXP39 <https://wiki.sei.cmu.edu/confluence/display/c/EXP39-C.+Do+not+access+a+variable+through+a+pointer+of+an+incompatible+type>`__
     - 互換性のない型のポインタを介して変数にアクセスしない
     -
     -
     -
   * - `CON32 <https://wiki.sei.cmu.edu/confluence/display/c/CON32-C.+Prevent+data+races+when+accessing+bit-fields+from+multiple+threads>`__
     - 複数のスレッドからビットフィールドにアクセスするときにデータ競合を防ぐ
     -
     -
     -
   * - `CON43 <https://wiki.sei.cmu.edu/confluence/display/c/CON43-C.+Do+not+allow+data+races+in+multithreaded+code>`__
     - マルチスレッドのコードでデータ競合を許さない
     -
     -
     - 要件が同じなので、Rust では **CON32 と対にして** 扱う
   * - `ARR36 <https://wiki.sei.cmu.edu/confluence/display/c/ARR36-C.+Do+not+subtract+or+compare+two+pointers+that+do+not+refer+to+the+same+array>`__
     - 同じ配列を指していない 2 つのポインタを減算したり比較したりしない
     -
     -
     -
   * - `MEM30 <https://wiki.sei.cmu.edu/confluence/display/c/MEM30-C.+Do+not+access+freed+memory>`__
     - 解放済みのメモリにアクセスしない
     -
     -
     -
   * - `EXP34 <https://wiki.sei.cmu.edu/confluence/display/c/EXP34-C.+Do+not+dereference+null+pointers>`__
     - ヌルポインタを参照外ししない
     -
     -
     -
   * - `DCL30 <https://wiki.sei.cmu.edu/confluence/display/c/DCL30-C.+Declare+objects+with+appropriate+storage+durations>`__
     - 適切な記憶域期間でオブジェクトを宣言する
     -
     -
     -
   * - `EXP42 <https://wiki.sei.cmu.edu/confluence/display/c/EXP42-C.+Do+not+compare+padding+data>`__
     - パディングデータを比較しない
     -
     -
     -
   * - `ENV31 <https://wiki.sei.cmu.edu/confluence/display/c/ENV31-C.+Do+not+rely+on+an+environment+pointer+following+an+operation+that+may+invalidate+it>`__
     - 環境ポインタを無効にしうる操作の後で、そのポインタに依存しない
     -
     -
     - 最近 unsafe の区分に移されました。このやり取りは、
       ```std::env::set_var`` <https://doc.rust-lang.org/std/env/fn.set_var.html>`__
       と
       ```std::env::remove_var`` <https://doc.rust-lang.org/std/env/fn.remove_var.html>`__
       の API を通じて Rust に対応します。Unix では、これらは **シングルスレッドのプログラムでのみ安全に使えます** 。執筆時点（2026 年 2 月 28 日）では、これがそれらのシステム API の実情です。このルールでは、Unix システム上で動くマルチスレッドのコードで両方の API を禁止することになるでしょう。
   * - `EXP33 <https://wiki.sei.cmu.edu/confluence/display/c/EXP33-C.+Do+not+read+uninitialized+memory>`__
     - 未初期化のメモリを読まない
     -
     -
     - *根拠: Rust では UB である*
   * - `MEM36 <https://wiki.sei.cmu.edu/confluence/display/c/MEM36-C.+Do+not+modify+the+alignment+of+objects+by+calling+realloc%28%29>`__
     - realloc() を呼び出してオブジェクトのアラインメントを変更しない
     -
     -
     -
   * - `CON34 <https://wiki.sei.cmu.edu/confluence/display/c/CON34-C.+Declare+objects+shared+between+threads+with+appropriate+storage+durations>`__
     - スレッド間で共有するオブジェクトを適切な記憶域期間で宣言する
     -
     -
     -
   * - `EXP36 <https://wiki.sei.cmu.edu/confluence/display/c/EXP36-C.+Do+not+cast+pointers+into+more+strictly+aligned+pointer+types>`__
     - ポインタを、より厳しくアラインされたポインタ型にキャストしない
     -
     -
     -
   * - `MEM35 <https://wiki.sei.cmu.edu/confluence/display/c/MEM35-C.+Allocate+sufficient+memory+for+an+object>`__
     - オブジェクトに十分なメモリを割り当てる
     -
     -
     -

表 3 – Rust に適用されないガイドラインのルール
----------------------------------------------------------------------------------------------------

.. list-table::
   :widths: 14 14 14 14 14
   :header-rows: 1

   * - ガイドライン
     - タイトル
     - カテゴリ
     - セーフティクリティカル Rust ルール
     - コメント
   * - `PRE31 <https://wiki.sei.cmu.edu/confluence/display/c/PRE31-C.+Avoid+side+effects+in+arguments+to+unsafe+macros>`__
     - 安全でないマクロの引数で副作用を避ける
     -
     -
     - **背景**\  *C における「安全でないマクロ」とは、パラメータの 1 つを 2 回以上評価する、またはまったく評価しないマクロを意味します。副作用を伴ってそうすると、その副作用が 2 回以上起こるか、まったく起こらないことになります。*\ C のマクロでは、この種のパターンを安全に使うことがとても難しくなっています。Rust ではそうではなく、このパターンを意図的に使って成功しているマクロの例がいくつもあります。実際、
       `Zulip のこちらのコメント <https://rust-lang.zulipchat.com/#narrow/channel/122651-general/topic/How.20common.20are.20macros.20that.20evaluate.20side.20effects.20twice.3F/near/564929315>`__
       で説明されているとおり、これにはさまざまなユースケースがあります。さらに、副作用が不要なら、
       `直後のコメント <https://rust-lang.zulipchat.com/#narrow/channel/122651-general/topic/How.20common.20are.20macros.20that.20evaluate.20side.20effects.20twice.3F/near/564929413>`__
       で説明されているとおり、マクロの作者は ``const`` ブロックを使ってそれを強制できます。マクロが副作用を持つ式を複数回展開するのを止める魔法のようなフィーチャは Rust にはありません。しかし、そうするにはずっと明示的に書く必要があります。したがって、実際にそうしたマクロ呼び出しに出会ったなら、ユーザはまさにそのつもりだった可能性が高いでしょう。\ *Zulip スレッド:* `Macros
       and insertion of side-effecting
       metavariables <https://rust-lang.zulipchat.com/#narrow/channel/404510-wg-macros/topic/Macros.20and.20insertion.20of.20side-effecting.20metavariables/with/564129698>`__
       マクロへの適用可能性についての現在の理由づけを要レビュー
   * - `EXP30 <https://wiki.sei.cmu.edu/confluence/display/c/EXP30-C.+Do+not+depend+on+the+order+of+evaluation+for+side+effects>`__
     - 副作用について評価順序に依存しない
     -
     -
     - C とは異なり、Rust では評価順序が完全に定義されています:
       `expressions -> expression
       precedence <https://doc.rust-lang.org/reference/expressions.html#expression-precedence>`__\ したがって、このルールは実際には必要ありません。少なくとも、C で必要とされるほど UB を止めるための緊急性はありません。\ *補足: 先例として、このトピックに触れる restriction リントがあります:*
       ```mixed_read_write_in_expression`` <https://rust-lang.github.io/rust-clippy/master/index.html?search=evaluation#mixed_read_write_in_expression>`__
   * - `EXP32 <https://wiki.sei.cmu.edu/confluence/display/c/EXP32-C.+Do+not+access+a+volatile+object+through+a+nonvolatile+reference>`__
     - volatile オブジェクトに非 volatile の参照を介してアクセスしない
     -
     -
     - この場合の volatile は C に固有のものです。さらに、このような不一致は型システムのルールに縛られるので、ユーザが transmute を使っていない限り検出されます。ただし、transmute に関するルールはこのルールの対応付けではなく、それ自体が独立したものです。
   * - `ARR39 <https://wiki.sei.cmu.edu/confluence/display/c/ARR39-C.+Do+not+add+or+subtract+a+scaled+integer+to+a+pointer>`__
     - スケーリングされた整数をポインタに加算または減算しない
     -
     -
     - Rust ではポインタ演算は API を通じて行います。 `それらのすべて <https://doc.rust-lang.org/std/primitive.pointer.html>`__
       が、 ``count: usize`` パラメータは基になる型 ``T`` を単位とすることを冒頭で述べています。名前自体も、それが特定の長さではなく **個数** であることを強く示しています。ARR39 のようなパターンが問題になるとわかれば、それについてのルールを作れます。しかし、私の知る限り問題にはなっていません。
   * - `FIO32 <https://wiki.sei.cmu.edu/confluence/display/c/FIO32-C.+Do+not+perform+operations+on+devices+that+are+only+appropriate+for+files>`__
     - ファイルにのみ適した操作をデバイスに対して行わない
     -
     -
     - **背景**\  *C では、開いているファイルがデバイス（ ``/dev/`` 以下のファイル）に対応する場合、それらの API は（現在のスレッドを）ブロックすることがあります。これは結果として、実質的にシステムをブロックすることでサービス拒否につながりえます。*
       ```std::fs`` <https://doc.rust-lang.org/std/fs/>`__ にある API は、C にあるものとはかなり異なります。このモジュールの一部の関数はブロックすることがあります: - ファイルシステムのロックを取得しようとするとき、 - ``Read::read`` を呼び出すとき（データを待ってブロックすることがある）、そして - ``Write::write`` を呼び出すとき（データが書き込まれるのを待ってブロックすることがある）。しかし、ファイルシステムに対するブロッキング動作の使用を制限するルールは、FIO32 とはかなり異なります。したがって、これは Rust には対応しないと考えます。関連するファイルシステムのガイダンスは、むしろ #392 の近くに置くべきかもしれません
   * - `FIO46 <https://wiki.sei.cmu.edu/confluence/display/c/FIO46-C.+Do+not+access+a+closed+file>`__
     - 閉じたファイルにアクセスしない
     -
     -
     - Rust では、ファイルを閉じると変数が消費され、それ以降のアクセスが無効になるので、これはすでに保証されています。\ *unsafe な Rust なら、いつものように、それでもこのようなことをやってのけられますが、それは UB であり、* **別のガイドラインで扱うべきです** *（「ライフタイムを有効な範囲を超えて延ばさない」といった趣旨のもの）*
   * - `ENV34 <https://wiki.sei.cmu.edu/confluence/display/c/ENV34-C.+Do+not+store+pointers+returned+by+certain+functions>`__
     - 特定の関数が返すポインタを保存しない
     -
     -
     - Rust では、生存中の参照は無効化できません。\ *unsafe な Rust でこうしたライブラリを実装する人は、自分が作る参照を無効にしないよう注意しなければなりません。それは UB になります。常にメモリモデルに違反します。しかし、これを強制するルールは ENV34 の文脈の外にあります。それは守らなければならない、より一般的なルールです。*
   * - `CON37 <https://wiki.sei.cmu.edu/confluence/display/c/CON37-C.+Do+not+call+signal%28%29+in+a+multithreaded+program>`__
     - マルチスレッドのプログラムで signal() を呼び出さない
     -
     -
     - Rust は ``std`` でシグナル処理をサポートしていません。その理由の詳細は
       `こちら <https://github.com/rust-lang/rfcs/issues/1368>`__ で確認できます。このフィーチャは Rust でサポートされていないので、このルールは適用されません。\ *とはいえ、シグナル処理のユーティリティを提供するクレートはあります。いくつかは上記のリンク先のスレッドで言及されています。読者の皆さん、それらのクレートは自己の判断で使ってください。*
   * - `MSC30 <https://wiki.sei.cmu.edu/confluence/display/c/MSC30-C.+Do+not+use+the+rand%28%29+function+for+generating+pseudorandom+numbers>`__
     - 擬似乱数の生成に rand() 関数を使わない
     -
     -
     - `core と std の擬似乱数生成 API <https://doc.rust-lang.org/std/random/index.html>`__ は、現在 nightly 限定です。\ *RNG だったかどうかはわかりませんが、記憶が正しければ、標準ライブラリのそれに似た何かも、衝突耐性がないという理由で Rust では非推奨とされていました。たしかハッシュ関数か何かだったと思います。*
   * - `STR31 <https://wiki.sei.cmu.edu/confluence/display/c/STR31-C.+Guarantee+that+storage+for+strings+has+sufficient+space+for+character+data+and+the+null+terminator>`__
     - 文字列の記憶域に、文字データとヌル終端文字のための十分な領域があることを保証する
     -
     -
     - Rust の文字列はヌル終端ではなく、その記憶域は保証されています
   * - `DCL40 <https://wiki.sei.cmu.edu/confluence/display/c/DCL40-C.+Do+not+create+incompatible+declarations+of+the+same+function+or+object>`__
     - 同じ関数やオブジェクトに対して互換性のない宣言を作らない
     -
     -
     - *根拠: Rust ではこれを行うことは不可能である。*
   * - `PRE30 <https://wiki.sei.cmu.edu/confluence/display/c/PRE30-C.+Do+not+create+a+universal+character+name+through+concatenation>`__
     - 連結によって国際文字名を作らない
     -
     -
     - *根拠: 単に C にとても固有のものである*
   * - `DCL38 <https://wiki.sei.cmu.edu/confluence/display/c/DCL38-C.+Use+the+correct+syntax+when+declaring+a+flexible+array+member>`__
     - フレキシブル配列メンバを宣言するときは正しい構文を使う
     -
     -
     - このルールは「未定義動作や、コンパイラ固有で標準化されていない構文に頼らず、このフィーチャの標準化された構文を使う」ことについてのものです。Rust にはコンパイラが 1 つしかなく、その構文が利用できる唯一のものなので、このような提案は Rust では意味をなしません。正しい構文はコンパイルでき、正しくない構文はコンパイルできません。
   * - `EXP35 <https://wiki.sei.cmu.edu/confluence/display/c/EXP35-C.+Do+not+modify+objects+with+temporary+lifetime>`__
     - 一時的なライフタイムを持つオブジェクトを変更しない
     -
     -
     - *根拠: 借用チェッカがこの問題をなくす*
   * - `EXP43 <https://wiki.sei.cmu.edu/confluence/display/c/EXP43-C.+Avoid+Undefined+Behavior+when+using+restrict-qualified+pointers>`__
     - restrict 修飾されたポインタを使うときに未定義動作を避ける
     -
     -
     - 借用チェッカによって、コード生成において構成上正しい restrict 修飾が得られます。基本的に「restrict」は型システムの一部であり、型チェック（特に借用チェック）によって、そのルールが常に守られることが強制されます。
   * - `ARR32 <https://wiki.sei.cmu.edu/confluence/display/c/ARR32-C.+Ensure+size+arguments+for+variable+length+arrays+are+in+a+valid+range>`__
     - 可変長配列のサイズ引数が有効な範囲内にあることを保証する
     -
     -
     - *根拠: Rust には VLA というものがない*
   * - `ARR38 <https://wiki.sei.cmu.edu/confluence/display/c/ARR38-C.+Guarantee+that+library+functions+do+not+form+invalid+pointers>`__
     - ライブラリ関数が無効なポインタを作らないことを保証する
     -
     -
     - このルールは基本的に、配列へのポインタと長さを受け取る関数を呼び出すときは常に、その長さが配列の境界を越えない（越えると無効なポインタを作ることになる）ようにすることをユーザに求めています。安全な Rust では、これは境界チェックで処理され、ARR30 の対応付けですでに扱っています。unsafe な Rust では、これを行うことは呼び出している API の安全性契約を破ることであり、それは `独自のルールが作られようとしています。 <https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines/issues/337>`__
       また、この ARR38（このルール）の対応付けの範囲外です。最近、適用対象の区分から移されました
   * - `STR32 <https://wiki.sei.cmu.edu/confluence/display/c/STR32-C.+Do+not+pass+a+non-null-terminated+character+sequence+to+a+library+function+that+expects+a+string>`__
     - 文字列を期待するライブラリ関数に、ヌル終端されていない文字の並びを渡さない
     -
     -
     - *根拠: Rust の文字列はヌル終端ではない*
   * - `FIO34 <https://wiki.sei.cmu.edu/confluence/display/c/FIO34-C.+Distinguish+between+characters+read+from+a+file+and+EOF+or+WEOF>`__
     - ファイルから読み取った文字と EOF または WEOF を区別する
     -
     -
     - `Read <https://doc.rust-lang.org/std/io/trait.Read.html>`__
       トレイトを参照してください。そこから EOF / WEOF を得ることはできません。ファイルの読み取り操作を Read トレイトを通じて行う限り、これは問題にならないはずです。
   * - `FIO37 <https://wiki.sei.cmu.edu/confluence/display/c/FIO37-C.+Do+not+assume+that+fgets%28%29+or+fgetws%28%29+returns+a+nonempty+string+when+successful>`__
     - fgets() や fgetws() が成功したときに空でない文字列を返すと仮定しない
     -
     -
     - このルールは、 ``fgets()`` と ``fgetws()`` の実行中に見つかったエラーを適切に処理する必要から生まれたものです。これらの関数のエラーは、その適切な処理をコンパイラが強制できない方法で報告されます。Rust の失敗しうる API は列挙型 ``Result<T, E>`` の値を返し、プログラムがコンパイルされるためには、そのエラーのケースを適切に処理しなければなりません。このフィーチャによって、Rust ではこの種のルールの必要性が完全になくなります。
   * - `FIO39 <https://wiki.sei.cmu.edu/confluence/display/c/FIO39-C.+Do+not+alternately+input+and+output+from+a+stream+without+an+intervening+flush+or+positioning+call>`__
     - フラッシュや位置決めの呼び出しを挟まずに、ストリームからの入力と出力を交互に行わない
     -
     -
     - これは、ストリーム処理 API の実装の詳細のために、C では UB です。Rust のこれらの API の実装は異なるので、このガイドラインは実際には対応しません。 \ *詳しくは、* `こちらのコメント <https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines/issues/336#issuecomment-3764046307>`__
       *と* `こちらの issue <https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines/issues/392>`__\ *を参照してください。*
   * - `FIO44 <https://wiki.sei.cmu.edu/confluence/display/c/FIO44-C.+Only+use+values+for+fsetpos%28%29+that+are+returned+from+fgetpos%28%29>`__
     - fsetpos() には fgetpos() が返した値だけを使う
     -
     -
     - Rust の ``std`` で、C の ``fsetpos`` と ``fgetpos`` に最も近い API は、
       ```std::io::Seek`` <https://doc.rust-lang.org/std/io/trait.Seek.html>`__
       トレイトが提供するものです。主な違いは 2 点にまとめられます: - このトレイトの基礎となるメソッド
       ```Seek::seek`` <https://doc.rust-lang.org/std/io/trait.Seek.html#tymethod.seek>`__
       は、与えられたバイト数だけシークします。このトレイトにはワイド指向のストリームという概念がないので、 ``fsetpos`` の ``fpos_t`` パラメータにエンコードされることがある「マルチバイトパーサの状態」を必要としません。 - ``Seek`` の API には未定義動作がありません。したがって、そこで使われるオフセットの出どころを制限する理由はありません。
   * - `ERR33 <https://wiki.sei.cmu.edu/confluence/display/c/ERR33-C.+Detect+and+handle+standard+library+errors>`__
     - 標準ライブラリのエラーを検出して処理する
     -
     -
     - std の失敗しうる関数は、結果を ``Result<T, E>`` 型で包みます。これにより、エラーのケースが起きたときは常に、呼び出し側がそれを処理することが強制されます。したがって、このルールはすでにコンパイラによって強制されています。
   * - `MSC37 <https://wiki.sei.cmu.edu/confluence/display/c/MSC37-C.+Ensure+that+control+never+reaches+the+end+of+a+non-void+function>`__
     - 制御が非 void 関数の終わりに決して到達しないことを保証する
     -
     -
     - *根拠: これはすでに型システムによって強制されている*
   * - `PRE32 <https://wiki.sei.cmu.edu/confluence/display/c/PRE32-C.+Do+not+use+preprocessor+directives+in+invocations+of+function-like+macros>`__
     - 関数形式マクロの呼び出しの中でプリプロセッサディレクティブを使わない
     -
     -
     - このルールは、C のプリプロセッサの動作のために C で必要とされるものです。C のプリプロセッサにとても固有のもので、Rust には対応しません
   * - `DCL31 <https://wiki.sei.cmu.edu/confluence/display/c/DCL31-C.+Declare+identifiers+before+using+them>`__
     - 識別子は使う前に宣言する
     -
     -
     - Rust の推論された型は、常に型チェックを通らなければなりません。そうでなければコンパイル時エラーになります。したがって、このルールのユースケースは Rust には存在しません。
   * - `DCL37 <https://wiki.sei.cmu.edu/confluence/display/c/DCL37-C.+Do+not+declare+or+define+a+reserved+identifier>`__
     - 予約済み識別子を宣言または定義しない
     -
     -
     - **背景** *C では、予約済み識別子にはキーワード以外のものも含まれます。ライブラリの関数や変数（例: ``memcpy``, ``errno``)）も含まれ、よく知られたもの（例: ``abs``）もあります。C はまた、多くの識別子を予約される可能性があるものとして挙げています（例: ``str`` で始まるものすべて）。C では、こうしたグローバルな識別子を再定義する（つまり上書きする）ことができます。たとえば、ヘッダに* ``#define errno my_errno`` *を追加するといったやり方で、これはほとんどの C プログラムを壊してしまいます。*\ Rust のスコープのルールは、この種の破壊をすでに防いでいます。どのような形の動作の上書きも局所的なスコープに限られ、影響を受ける部分が明示的にオプトインするものです。
   * - `EXP37 <https://wiki.sei.cmu.edu/confluence/display/c/EXP37-C.+Call+functions+with+the+correct+number+and+type+of+arguments>`__
     - 正しい数と型の引数で関数を呼び出す
     -
     -
     - これは型システムによって強制されます
   * - `EXP44 <https://wiki.sei.cmu.edu/confluence/display/c/EXP44-C.+Do+not+rely+on+side+effects+in+operands+to+sizeof%2C+_Alignof%2C+or+_Generic>`__
     - sizeof、\_Alignof、\_Generic のオペランドの副作用に依存しない
     -
     -
     - これは C に固有のものです
   * - `EXP47 <https://wiki.sei.cmu.edu/confluence/display/c/EXP47-C.+Do+not+call+va_arg+with+an+argument+of+the+incorrect+type>`__
     - 正しくない型の引数で va_arg を呼び出さない
     -
     -
     - 安全な Rust には可変長引数関数は存在しません。 **unsafe な Rust** には、C の可変長引数関数という形で存在します。これらは、FFI の文脈で可変長引数関数を宣言し実装する手段として存在します。とはいえ、このルールは Rust には対応しません。 `C の可変長引数関数を正しくなく実装することはありえます <https://doc.rust-lang.org/std/ffi/struct.VaList.html#safety>`__
       が、その問題は、 `unsafe fn の呼び出し側はその関数の安全性契約を守らなければならない <https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines/issues/337>`__
       という、より一般的なルールでカバーされるからです。\ *補遺: 「C の可変長引数関数は FFI の文脈以外では使うべきではない」といった趣旨のルールを主張するのは十分に妥当ですが、そのようなルールは CERT-C の EXP47 ルールの対応付けにはなりません。それは今後の作業とします。*
   * - `STR34 <https://wiki.sei.cmu.edu/confluence/display/c/STR34-C.+Cast+characters+to+unsigned+char+before+converting+to+larger+integer+sizes>`__
     - より大きなサイズの整数に変換する前に、文字を unsigned char にキャストする
     -
     -
     - Rust の文字は整数ではありません。 `Unicode スカラ値 <https://www.unicode.org/glossary/#unicode_scalar_value>`__ です。
   * - `FIO30 <https://wiki.sei.cmu.edu/confluence/display/c/FIO30-C.+Exclude+user+input+from+format+strings>`__
     - 書式文字列からユーザ入力を除外する
     -
     -
     - Rust では、書式文字列はコンパイル時に評価されます
   * - `FIO41 <https://wiki.sei.cmu.edu/confluence/display/c/FIO41-C.+Do+not+call+getc%28%29%2C+putc%28%29%2C+getwc%28%29%2C+or+putwc%28%29+with+a+stream+argument+that+has+side+effects>`__
     - 副作用を持つストリーム引数で getc()、putc()、getwc()、putwc() を呼び出さない
     -
     -
     - このルールは、これらの API が（C のマクロとして）どう実装されているかから生じるもので、状況のより一般的な側面から生じるものではありません。
   * - `ENV30 <https://wiki.sei.cmu.edu/confluence/display/c/ENV30-C.+Do+not+modify+the+object+referenced+by+the+return+value+of+certain+functions>`__
     - 特定の関数の戻り値が参照するオブジェクトを変更しない
     -
     -
     - Rust では、不変性は必要な場合に常に強制されます。\ *unsafe な Rust なら、いくつかの手順を踏めばこれをやってのけられると思います。しかし、それは「有効性不変条件によって不変であることが求められる値を変更しない」というルールの近くにあることになるでしょう。有効性不変条件を破るのは、もちろん UB です。*
   * - `ENV32 <https://wiki.sei.cmu.edu/confluence/display/c/ENV32-C.+All+exit+handlers+must+return+normally>`__
     - すべての終了ハンドラは正常にリターンしなければならない
     -
     -
     - パニックに UB はありません。パニック中のパニックは、最終的にプログラムをアボートさせます。
   * - `ERR34 <https://wiki.sei.cmu.edu/confluence/display/c/ERR34-C.+Detect+errors+when+converting+a+string+to+a+number>`__
     - 文字列を数値に変換するときにエラーを検出する
     -
     -
     - std の失敗しうる関数は、結果を ``Result<T, E>`` 型で包みます。これにより、エラーのケースが起きたときは常に、呼び出し側がそれを処理することが強制されます。したがって、このルールはすでにコンパイラによって強制されています。
   * - `DCL36 <https://wiki.sei.cmu.edu/confluence/display/c/DCL36-C.+Do+not+declare+an+identifier+with+conflicting+linkage+classifications>`__
     - 矛盾するリンケージ区分で識別子を宣言しない
     -
     -
     - これは C にとても固有の問題です
   * - `DCL41 <https://wiki.sei.cmu.edu/confluence/display/c/DCL41-C.+Do+not+declare+variables+inside+a+switch+statement+before+the+first+case+label>`__
     - switch 文の中で、最初の case ラベルより前に変数を宣言しない
     -
     -
     - C の ``switch`` 文のセマンティクスは、（幸いにも）Rust には存在しません
   * - `EXP45 <https://wiki.sei.cmu.edu/confluence/display/c/EXP45-C.+Do+not+perform+assignments+in+selection+statements>`__
     - 選択文の中で代入を行わない
     -
     -
     - Rust は式指向です。Rust では代入は ``()`` を返します。したがって、このルールのユースケースは Rust には実際には存在しません。
   * - `EXP46 <https://wiki.sei.cmu.edu/confluence/display/c/EXP46-C.+Do+not+use+a+bitwise+operator+with+a+Boolean-like+operand>`__
     - ブール値のようなオペランドにビット演算子を使わない
     -
     -
     - 型システムのおかげで、Rust ではこれらの演算子を使うのは妥当なことです。これらは明確に定義されており、正しく振る舞います。
   * - `FIO38 <https://wiki.sei.cmu.edu/confluence/display/c/FIO38-C.+Do+not+copy+a+FILE+object>`__
     - FILE オブジェクトをコピーしない
     -
     -
     - これは、 ``FILE`` オブジェクトが処理系定義の動作を持つことに関係しています。この場合、これは完全に C に固有のものです。
   * - `FIO40 <https://wiki.sei.cmu.edu/confluence/display/c/FIO40-C.+Reset+strings+on+fgets%28%29++or+fgetws%28%29+failure>`__
     - fgets() や fgetws() が失敗したときは文字列をリセットする
     -
     -
     - 元の ``fgets()`` と ``fgetws()`` の実装とは異なり、Rust の ``std`` の I/O 読み取り API におけるバッファ ``buf`` のバイトは、すべての場面で明確に定義された値を持ちます。それらは ``Read`` トレイトとその
       ```read`` <https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read>`__
       関数のルールに従います。
       ```Read::read_to_string`` <https://doc.rust-lang.org/std/io/trait.Read.html#method.read_to_string>`__
       のような関数は、これらの構成要素から作られています。この問題の根本原因は Rust には存在しないので、このルールは必要ありません。
   * - `SIG30 <https://wiki.sei.cmu.edu/confluence/display/c/SIG30-C.+Call+only+asynchronous-safe+functions+within+signal+handlers>`__
     - シグナルハンドラの中では非同期安全な関数だけを呼び出す
     -
     -
     - Rust は ``std`` でシグナル処理をサポートしていません。その理由の詳細は
       `こちら <https://github.com/rust-lang/rfcs/issues/1368>`__ で確認できます。このフィーチャは Rust でサポートされていないので、このルールは適用されません。\ *とはいえ、シグナル処理のユーティリティを提供するクレートはあります。いくつかは上記のリンク先のスレッドで言及されています。読者の皆さん、それらのクレートは自己の判断で使ってください。*
   * - `SIG31 <https://wiki.sei.cmu.edu/confluence/display/c/SIG31-C.+Do+not+access+shared+objects+in+signal+handlers>`__
     - シグナルハンドラの中で共有オブジェクトにアクセスしない
     -
     -
     - *SIG30 と同じ*
   * - `ERR30 <https://wiki.sei.cmu.edu/confluence/display/c/ERR30-C.+Take+care+when+reading+errno>`__
     - errno を読むときは注意する
     -
     -
     - Rust には errno は存在しません。さらに、失敗しうる関数は結果を ``Result<T, E>`` 型で包みます。これにより、エラーのケースが起きた場合に限り、呼び出し側がそれを処理することが強制されます。
   * - `ERR32 <https://wiki.sei.cmu.edu/confluence/display/c/ERR32-C.+Do+not+rely+on+indeterminate+values+of+errno>`__
     - errno の不定値に依存しない
     -
     -
     - *根拠: ERR30-C を参照*
   * - `CON30 <https://wiki.sei.cmu.edu/confluence/display/c/CON30-C.+Clean+up+thread-specific+storage>`__
     - スレッド固有の記憶域を後始末する
     -
     -
     - Rust におけるスレッド固有の記憶域とは、
       ```LocalKey`` <https://doc.rust-lang.org/std/thread/struct.LocalKey.html>`__
       の背後にある値を指します。この種の記憶域の後始末のされ方と、スレッドに局所的な他のリソース（スタック変数など）の後始末のされ方との間に、重要な違いはありません。したがって、独自のルールを設けるには値しません。
   * - `CON39 <https://wiki.sei.cmu.edu/confluence/display/c/CON39-C.+Do+not+join+or+detach+a+thread+that+was+previously+joined+or+detached>`__
     - すでに join または detach したスレッドを join または detach しない
     -
     -
     - これは次の理由で Rust には適用されません: -
       ```std::thread::JoinHandle::join`` <https://doc.rust-lang.org/std/thread/struct.JoinHandle.html#method.join>`__
       はスレッドを消費する。そして -
       ```JoinHandle`` <https://doc.rust-lang.org/std/thread/struct.JoinHandle.html>`__
       は ``Clone`` を実装していないので、2 回 join することは不可能である。不正にコピーするには ``unsafe`` なコードが必要になります。しかし、それも「所有権のセマンティクスに従う」という 1 つのルールで済ませられます。
   * - `INT35 <https://wiki.sei.cmu.edu/confluence/display/c/INT35-C.+Use+correct+integer+precisions>`__
     - 正しい整数の精度を使う
     -
     -
     - Rust では、すべてのプリミティブ整数型がその精度を保証しています
   * - `STR37 <https://wiki.sei.cmu.edu/confluence/display/c/STR37-C.+Arguments+to+character-handling+functions+must+be+representable+as+an+unsigned+char>`__
     - 文字処理関数の引数は unsigned char として表現可能でなければならない
     -
     -
     - Rust の文字は整数ではありません
   * - `STR38 <https://wiki.sei.cmu.edu/confluence/display/c/STR38-C.+Do+not+confuse+narrow+and+wide+character+strings+and+functions>`__
     - ナロー文字列とワイド文字列、およびそれらの関数を混同しない
     -
     -
     - Rust には文字型が 1 つしかありません
   * - `MEM33 <https://wiki.sei.cmu.edu/confluence/display/c/MEM33-C.++Allocate+and+copy+structures+containing+a+flexible+array+member+dynamically>`__
     - フレキシブル配列メンバを含む構造体は動的に割り当て、コピーする
     -
     -
     - Rust にはフレキシブル配列メンバというものはありません
   * - `FIO47 <https://wiki.sei.cmu.edu/confluence/display/c/FIO47-C.+Use+valid+format+strings>`__
     - 有効な書式文字列を使う
     -
     -
     - Rust では、書式文字列はコンパイル時に評価され、したがって検証されます
   * - `SIG34 <https://wiki.sei.cmu.edu/confluence/display/c/SIG34-C.+Do+not+call+signal%28%29+from+within+interruptible+signal+handlers>`__
     - 割り込み可能なシグナルハンドラの中から signal() を呼び出さない
     -
     -
     - Rust は ``std`` でシグナル処理をサポートしていません。その理由の詳細は
       `こちら <https://github.com/rust-lang/rfcs/issues/1368>`__ で確認できます。このフィーチャは Rust でサポートされていないので、このルールは適用されません。\ *とはいえ、シグナル処理のユーティリティを提供するクレートはあります。いくつかは上記のリンク先のスレッドで言及されています。読者の皆さん、それらのクレートは自己の判断で使ってください。*
   * - `SIG35 <https://wiki.sei.cmu.edu/confluence/display/c/SIG35-C.+Do+not+return+from+a+computational+exception+signal+handler>`__
     - 計算例外シグナルハンドラからリターンしない
     -
     -
     - *根拠: SIG34 を参照*
   * - `CON31 <https://wiki.sei.cmu.edu/confluence/display/c/CON31-C.+Do+not+destroy+a+mutex+while+it+is+locked>`__
     - ロック中のミューテックスを破棄しない
     -
     -
     - 所有権のセマンティクスにより、Rust ではこれを行うことは不可能です。破棄できるのは ``&mut`` でアクセスできるものだけです。ユーザは ``unsafe`` を使って不正にそのような参照を作れますが、そうすると所有権のセマンティクスに違反します（そして即座に未定義動作になります）。
   * - `MSC33 <https://wiki.sei.cmu.edu/confluence/display/c/MSC33-C.+Do+not+pass+invalid+data+to+the+asctime%28%29+function>`__
     - asctime() 関数に無効なデータを渡さない
     -
     -
     - ``core`` にも ``std`` にも、同等の時刻表示関数はありません。\ *執筆時点（2026-03-03）では、この種の機能は ``chrono``, ``hifitime``, ``icu``, ``jiff``, ``time`` などの外部ライブラリを通じて利用できます。*
   * - `MSC38 <https://wiki.sei.cmu.edu/confluence/display/c/MSC38-C.+Do+not+treat+a+predefined+identifier+as+an+object+if+it+might+only+be+implemented+as+a+macro>`__
     - マクロとしてのみ実装されているかもしれない定義済み識別子をオブジェクトとして扱わない
     -
     -
     - Rust にはマクロへのポインタというものはありません
   * - `MSC39 <https://wiki.sei.cmu.edu/confluence/display/c/MSC39-C.+Do+not+call+va_arg%28%29+on+a+va_list+that+has+an+indeterminate+value>`__
     - 不定値を持つ va_list に対して va_arg() を呼び出さない
     -
     -
     - Rust には可変長引数関数は存在しません
   * - `MSC40 <https://wiki.sei.cmu.edu/confluence/display/c/MSC40-C.+Do+not+violate+constraints>`__
     - 制約に違反しない
     -
     -
     - *MSC40 の背景: このルールは、C 標準に準拠しないコードを書かないことについてのものである。*\ Rust コンパイラはそのようなプログラムをすべて拒否します。有効な Rust コードでないプログラムはコンパイルされません。
