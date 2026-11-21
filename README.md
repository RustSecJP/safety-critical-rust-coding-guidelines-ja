# Safety-Critical Rust Coding Guidelines（日本語訳）

[![原文](https://img.shields.io/badge/原文-Safety--Critical--Rust--Consortium-blue)](https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines)
[![日本語訳](https://img.shields.io/badge/日本語訳-非公式-orange)](https://rustsecjp.github.io/safety-critical-rust-coding-guidelines-ja/)

> [!NOTE]
> これは Safety-Critical Rust Consortium の [Safety-Critical Rust Coding Guidelines](https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines) の**非公式な日本語訳**です。Safety-Critical Rust Consortium による確認や公認を受けたものではありません。内容が食い違う場合は原文が優先されます。
> This is an unofficial Japanese translation, not endorsed by the Safety-Critical Rust Consortium.

安全性が重要な分野（自動車、航空、医療など）で Rust を使うための、コーディングガイドラインです。

**読むのはこちら: https://rustsecjp.github.io/safety-critical-rust-coding-guidelines-ja/**

## 訳した版

| | |
|---|---|
| 原文の版 | 2026-11-18 時点（コミット [`924aa78`](https://github.com/Safety-Critical-Rust-Consortium/safety-critical-rust-coding-guidelines/commit/924aa78c82f21dd037f5d9c61ec9ce1c3fbe3f1a)） |
| 翻訳日 | 2026-11-21 |

- 訳したのは `src/` の本文です。取り下げ済みのガイドライン（`src/retired-guidelines/`）は英語のままです。
- ガイドラインの ID、状態、タグなど、機械が読む項目は原文のままです。
- 組み立ては原文と同じ Sphinx の仕組み（`make.py`）を、GitHub Actions で動かしています。
- 原文の README は [README.en.md](README.en.md) にあります。

## ライセンス

原文と同じです。文書は CC BY 4.0、コードは MIT または Apache-2.0 です（[LICENSES/](LICENSES)、[COPYRIGHT](COPYRIGHT)）。誤訳の指摘や訳語の提案は、Issue か Pull Request でお願いします。

RustSecJP は非公式のコミュニティです。Safety-Critical Rust Consortium、Rust Project、Rust Foundation とは提携していません。
