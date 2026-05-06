# VietPay デザインシステム

VietPayモバイル + Web向けデザイントークンです。色、タイポグラフィ、スペーシング、角丸、エレベーションのソース・オブ・トゥルースとなります。

## ブランドカラー

```
--brand-primary:   #00B074   /* VietPay green */
--brand-secondary: #0EA5E9   /* sky blue */
--success:         #16A34A
--warning:         #F59E0B
--danger:          #DC2626
--info:            #2563EB
```

## ニュートラル

```
--bg-base:   #FFFFFF / #0A0E1A (dark)
--bg-card:   #F8FAFC / #0F1629 (dark)
--text-pri:  #0F172A / #E2E8F0 (dark)
--text-sec:  #64748B / #94A3B8 (dark)
--border:    #E2E8F0 / rgba(56,189,248,0.12) (dark)
```

## タイポグラフィ

```
--font-display: "SF Pro Display", system-ui  (見出し、ディスプレイ)
--font-body:    "Inter", system-ui            (本文、UI)
--font-mono:    "JetBrains Mono", monospace   (コード、数値)
```

タイプスケール（モジュラー1.25）:
- xs `12px` · sm `14px` · base `16px` · lg `20px` · xl `25px` · 2xl `32px` · 3xl `40px`

## スペーシング（4pxグリッド）

`4 · 8 · 12 · 16 · 24 · 32 · 48 · 64`

## 角丸（Radius）

`sm 4px` · `md 8px` · `lg 12px` · `xl 16px` · `2xl 24px` · `full 9999px`

## エレベーション（影）

```
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05)
--shadow-md: 0 4px 6px rgba(0,0,0,0.08), 0 2px 4px rgba(0,0,0,0.06)
--shadow-lg: 0 10px 15px rgba(0,0,0,0.10), 0 4px 6px rgba(0,0,0,0.05)
--shadow-xl: 0 20px 25px rgba(0,0,0,0.12), 0 10px 10px rgba(0,0,0,0.04)
```

## コンポーネントアンカー

- ボタン: タッチターゲット44px、角丸`lg`、プライマリ塗り `--brand-primary`
- インプット: 高さ48px、角丸`md`、フォーカスリング `--brand-secondary`
- カード: 角丸`xl`、パディング`16-24px`、シャドウ`md`
- モーダル: 最大幅 モバイル`560px` / Web`720px`、角丸`2xl`

## モーション

- 持続時間: `fast 120ms` · `base 200ms` · `slow 320ms`
- イージング: `cubic-bezier(0.4, 0, 0.2, 1)`（標準）
