# PDF Merger フルスクラッチ実装計画

## 概要

pure JavaScript で PDF 結合機能を再実装する。外部ライブラリ（pdf-merger-js）を使用せず、ブラウザ内で完結する実装とする。

## 実装フェーズ

### Phase 1: プロジェクト構造の準備
- [ ] 既存 `pdf-merger.html` をバックアップ（または新規作成）
- [ ] 基本 HTML 構造の作成
- [ ] CSS スタイル（日本語 UI、ダークテーマ）の実装

### Phase 2: PDF パーサーの実装
- [ ] PDF バイナリ読み込み機能
- [ ] PDF ヘッダー解析（%PDF-x.y）
- [ ] XREF テーブル解析（PDF 1.4 形式）
- [ ] オブジェクト構造の解析
- [ ] Trailer 辞書の解析（/Root, /Info, /Size）

### Phase 3: ページ抽出機能
- [ ] /Catalog → /Pages → /Kids ツリーの辿り
- [ ] Page オブジェクトの特定（/Type /Page）
- [ ] コンテンツストリームの抽出（/Contents）
- [ ] リソース（/Resources）の処理
- [ ] MediaBox の取得

### Phase 4: ストリーム処理
- [ ] FlateDecode（ZLIB/DEFLATE）デコード実装
  - RFC 1950（ZLIB）対応
  - RFC 1951（DEFLATE）対応
  - 固定ハフマン、動的ハフマン、非圧縮ブロック
- [ ] ストリームのエンコード（結合用）

### Phase 5: PDF ビルダー
- [ ] 新規 PDF ヘッダー出力
- [ ] オブジェクトのリナンバリング
- [ ] XREF テーブルの再構築
- [ ] Trailer の生成
- [ ] %%EOF の出力

### Phase 6: UI 実装
- [ ] ファイル選択（ドラッグ＆ドロップ、クリック）
- [ ] ファイルリスト表示（順序変更可能）
- [ ] プログレス表示
- [ ] ログ出力
- [ ] ダウンロード機能

### Phase 7: テスト・検証
- [ ] サンプル PDF（1.pdf ~ 4.pdf）での動作確認
- [ ] 複数ページ PDF の結合テスト
- [ ] エラーハンドリングの確認

## 技術仕様

### 対応フォーマット
- PDF 1.4（伝統的な XREF テーブル形式）
- FlateDecode 圧縮ストリーム

### 非対応（将来の拡張）
- PDF 1.5+ の XREF ストリーム
- Object ストリーム
- 暗号化 PDF

### アーキテクチャ
```
PDFParser (入力 PDF 解析)
  ↓
PageExtractor (ページ抽出)
  ↓
StreamDecoder (必要に応じてデコード)
  ↓
PDFBuilder (出力 PDF 構築)
  ↓
ダウンロード
```

## ファイル構成

```
pdf-merger/
├── pdf-merger.html      # メインアプリケーション
├── IMPLEMENTATION_PLAN.md  # 本計画書
└── [サンプル PDF files]
```

## 注意事項

1. **メモリ管理**: 大きな PDF ファイルはブラウザメモリに注意
2. **エラーハンドリング**: 破損 PDF や未対応形式の検出
3. **パフォーマンス**: 大きな配列操作はチャンク処理を検討
4. **セキュリティ**: ファイル名のエスケープ、XSS 対策
