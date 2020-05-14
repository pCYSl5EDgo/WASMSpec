# 私家版 WebAssembly version 1.1 仕様書日本語訳

この文書はW3CのWebAssembly Community Groupにより策定されたWebAssemblyのバージョン1.1の仕様書を私的に日本語訳したものです。

# もくじ

 - [はじめに](https://pcysl5edgo.github.io/WASMSpec/Introduction)
   - [はじめに](https://pcysl5edgo.github.io/WASMSpec/Introduction#はじめに)
   - [概論](https://pcysl5edgo.github.io/WASMSpec/Introduction#概論)
 - [構造](https://pcysl5edgo.github.io/WASMSpec/Structure)
   - [表記上のお約束](https://pcysl5edgo.github.io/WASMSpec/Structure#表記上のお約束)
   - [値](https://pcysl5edgo.github.io/WASMSpec/Structure#値)
   - [型](https://pcysl5edgo.github.io/WASMSpec/Structure#型)
   - [命令](https://pcysl5edgo.github.io/WASMSpec/Structure#命令)
   - [モジュール](https://pcysl5edgo.github.io/WASMSpec/Structure#モジュール)
 - [検証](https://pcysl5edgo.github.io/WASMSpec/Validation)
   - [表記上のお約束](https://pcysl5edgo.github.io/WASMSpec/Validation#表記上のお約束)
   - [型](https://pcysl5edgo.github.io/WASMSpec/Validation#型)
   - [命令](https://pcysl5edgo.github.io/WASMSpec/Validation#命令)
   - [モジュール](https://pcysl5edgo.github.io/WASMSpec/Validation#モジュール)
 - [実行](https://pcysl5edgo.github.io/WASMSpec/Execution)
   - [表記上のお約束](https://pcysl5edgo.github.io/WASMSpec/Execution#表記上のお約束)
   - [ランタイムの構造](https://pcysl5edgo.github.io/WASMSpec/Execution#ランタイムの構造)
   - [数値](https://pcysl5edgo.github.io/WASMSpec/Execution#数値)
   - [命令](https://pcysl5edgo.github.io/WASMSpec/Execution#命令)
   - [モジュール](https://pcysl5edgo.github.io/WASMSpec/Execution#モジュール)
 - [Binary Format](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat)
   - [表記上のお約束](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat#表記上のお約束)
   - [値](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat#値)
   - [型](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat#型)
   - [命令](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat#命令)
   - [モジュール](https://pcysl5edgo.github.io/WASMSpec/BinaryFormat#モジュール)
 - [Text Format(工事中)](https://pcysl5edgo.github.io/WASMSpec/TextFormat)
 - [補遺](https://pcysl5edgo.github.io/WASMSpec/Appendix)

# LICENSE

 - [ライセンス本文](./LICENSE)
 - [原文ライセンス本文](https://www.w3.org/Consortium/Legal/2015/copyright-software-and-document)

# 英文原著リポジトリ

[英文原著リポジトリ](https://github.com/WebAssembly/spec)のみが厳密に正確な仕様書です。
この翻訳は不正確な可能性があります。