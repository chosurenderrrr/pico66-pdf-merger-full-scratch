# PDF Merger フルスクラッチ実装計画

## 概要

pure JavaScript で PDF 結合機能を再実装する。外部ライブラリ（pdf-merger-js）を使用せず、ブラウザ内で完結する実装とする。

## 実装フェーズ

### Phase 1: ネットワーク隔離・セキュリティ ✅
- [x] CSP (Content Security Policy) の設定
  - `connect-src 'none'`: 外部ネットワーク接続を完全に遮断
  - `font-src 'none'`: 外部フォント読み込みを禁止
- [x] JavaScriptによるネットワークAPIの上書き
  - `fetch()`, `XMLHttpRequest`, `WebSocket`, `sendBeacon` を監視・遮断

### Phase 2: PDF Parser ✅
- [x] PDFバイナリ読み込み機能
- [x] PDFヘッダー検証（%PDF-x.y形式チェック）
- [x] startxref位置の特定
- [x] XREFテーブル解析（PDF 1.4形式）
  - オブジェクトオフセットマップの構築（`Map<number, offset>`）
- [x] Trailer辞書のパース（/Root, /Info, /Size）
- [x] 破損PDF対応：スキャンによるXREF再構築機能

### Phase 3: ページ抽出（圧縮ストリームをそのままコピー）✅
- [x] /Catalog → /Pages → /Kidsツリーの辿り
- [x] Pageオブジェクトの特定（/Type /Page）
- [x] /Contentsストリームの収集（圧縮されたまま）
  - FlateDecode等のフィルタ情報を保持
  - Uint8Arrayとしてバイナリデータを抽出
- [x] /MediaBox、/Resourcesの抽出

**設計変更**: ZLIB/DEFLATE解凍を行わず、元の圧縮データをそのままコピー
- メリット: 解凍処理のバグを回避、高速化、元の圧縮率を維持
- 実装: `collectPageContent()`が`{ bytes, filter, length }`を返す

### Phase 4: PDFビルダー ✅
- [x] 新規PDFヘッダー出力（%PDF-1.4）
- [x] オブジェクトのリナンバリング
  - 1 = Catalog
  - 2 = Pages
  - 3+ = コンテンツストリーム（必要に応じて）
  - N+ = ページオブジェクト
- [x] XREFテーブルの再構築
  - 正確なバイトオフセット計算
  - 使用/未使用オブジェクトの追跡
- [x] Trailer辞書の生成（/Size, /Root）
- [x] startxrefと%%EOFの出力
- [x] すべてのパートを結合してUint8Arrayとして返却

### Phase 5: UI統合・テスト 🔄
- [x] ファイル選択（ドラッグ＆ドロップ、クリック）
- [x] ファイルリスト表示（順序変更可能）
- [x] プログレス表示
- [x] ログ出力（日本語）
- [x] ダウンロード機能
- [ ] サンプルPDF（1.pdf ~ 4.pdf）での動作確認
- [ ] エラーハンドリングの最適化

## 実装アーキテクチャ

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: セキュリティレイヤー                               │
│  - CSPヘッダーによる外部接続遮断                             │
│  - JSによるfetch/XHR/WebSocket遮断                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: PDF Parser                                        │
│  - verifyHeader()        → %PDF-x.y検証                     │
│  - findStartXref()       → startxref位置特定                │
│  - parseXrefTable()      → XREFテーブル解析                 │
│  - parseTrailer()        → /Root, /Info抽出                 │
│  - getObject()           → オブジェクト取得                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: ページ抽出と解凍                                   │
│  - extractPages()        → /Pagesツリーを辿る               │
│  - collectPagesFromTree()→ 再帰的ページ収集                 │
│  - extractStreamData()   → ストリーム抽出                   │
│  - zlibInflate()         → ZLIB解凍                         │
│  - collectPageContent()  → コンテンツ収集                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: PDFビルダー                                        │
│  - buildMergedPDF()      → PDF構築                          │
│    ├─ Header出力 (%PDF-1.4)                                 │
│    ├─ オブジェクトリナンバリング                             │
│    │   1: Catalog, 2: Pages, 3+: Contents, N+: Pages        │
│    ├─ XREFテーブル構築                                       │
│    └─ Trailer + startxref + %%EOF                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: UI                                                │
│  - ファイル選択/ドラッグ＆ドロップ                          │
│  - 進捗表示（プログレスバー）                               │
│  - ログ出力（日本語）                                       │
│  - ダウンロード（Blob URL）                                 │
└─────────────────────────────────────────────────────────────┘
```

## 技術仕様

### 対応フォーマット
- PDF 1.4（伝統的な XREF テーブル形式）
- FlateDecode 圧縮ストリーム

### 非対応（将来の拡張）
- PDF 1.5+ の XREF ストリーム
- Object ストリーム
- 暗号化 PDF

### セキュリティ対策
```
┌─────────────────────────────────────────┐
│  CSP (HTTP Header)                      │
│  - connect-src 'none'                   │
│  - font-src 'none'                      │
│  - object-src 'none'                    │
├─────────────────────────────────────────┤
│  JavaScript によるAPI上書き             │
│  - fetch() → 外部URL遮断                │
│  - XMLHttpRequest → 外部URL遮断         │
│  - WebSocket → 完全遮断                 │
│  - sendBeacon → 外部URL遮断             │
└─────────────────────────────────────────┘
```

## ファイル構成

```
pdf-merger/
├── pdf-merger.html      # メインアプリケーション
├── IMPLEMENTATION_PLAN.md  # 本計画書
├── AGENTS.md            # プロジェクトドキュメント
└── [サンプル PDF files]
```

## 注意事項

1. **メモリ管理**: 大きな PDF ファイルはブラウザメモリに注意
2. **エラーハンドリング**: 破損 PDF や未対応形式の検出
3. **パフォーマンス**: 大きな配列操作はチャンク処理を検討
4. **セキュリティ**: ファイル名のエスケープ、XSS 対策
5. **ネットワーク隔離**: 外部へのデータ漏洩を完全に防止
