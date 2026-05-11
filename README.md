# HEXACO Field Checklist

HEXACO-PI-R に基づく対人分析チェックリスト。完全オフライン動作 (PWA)。

🔗 **Live**: https://shimatoshi.github.io/hexaco-checklist/

## 何ができるか

- 観察可能な行動 cue (101項目) をチェック → HEXACO 6因子 + Dark Triad 3因子を推定
- 日本人 prior (性別×年代別) で事前分布を初期化、cue で事後更新
- Tokiwa 2024 (N=13,668) の7クラスタとマッチング
- 推定プロファイル × クラスタ → 5領域戦略 (話題/言葉遣い/立ち振舞い/服装/対話) と「憑依推奨キャラクターシート」を自動生成
- JSON エクスポート / インポート / LocalStorage 保存
- Service Worker でオフライン動作、ホーム画面追加可能

## 設計思想

- 推論層は完全に決定論。同一入力 → 同一出力
- AI/LLM 推論は実行時に **使わない** (オフラインで Claude/ChatGPT 等不要)
- 全ロジックとデータが単一 HTML に内包

## アーキテクチャ

```
Layer 1: ステータス入力 (性別/年代/関係性/環境)
   ↓ 日本人 prior 適用
Layer 2: cue チェックリスト (101項目, 6カテゴリ)
   ↓ Bayesian-like 事後更新 (各 cue の delta を加算 + クリッピング)
Layer 3: 推定プロファイル → クラスタ判定 → 戦略合成
   = [クラスタテンプレ A] + [因子モディファイア B] + [Dark Triad オーバーレイ C]
```

## データソース

### 心理測定モデル
- **HEXACO-PI-R 100**: Lee & Ashton, https://hexaco.org/
- **日本語版**: 若林明雄 (2014). *Japanese Psychological Research*, 56(3), 211-223
- **採点キー**: hexaco.org/downloads/ScoringKeys_100.pdf

### 日本人 prior データ
- **Big Five 性×年代**: Kawamoto et al. (2015). *発達心理学研究*, 26(2), 107-122
  ([J-Stage open access](https://www.jstage.jst.go.jp/article/jjdp/26/2/26_107/_article/-char/ja/))
- **Dark Triad**: 下司・川本・小塩 (2017). *日本心理学会大会発表論文集*, 81, 3B-004
  ([J-Stage open access](https://www.jstage.jst.go.jp/article/pacjpa/81/0/81_3B-004/_pdf))
- **クラスタ**: Tokiwa, E. (2024). Cultural Influences on Personality Types: A Cluster Analysis of HEXACO Traits in Japan. Research Square preprint, doi:10.21203/rs.3.rs-5323735/v1

### 補助データソース
- Yoshino & Oshio (2021, 2022): 日本人地域差・BFI-2-J
- Lee & Ohtake (2016): 学歴/年収係数 (日本男性 Agreeableness が最大効果)
- Oshio (2014): 自尊感情メタ分析 N=48,927
- 観察 cue weight: Gosling 2002 (環境)、Park 2015 (Facebook 言語)、Tanaka 2022 (日本人 SNS) 他

詳細citation は `research/*/findings.json` を参照。

## ファイル構成

```
hexaco-checklist/
├── index.html                    # 単一HTMLアプリ (全ロジック・JSONデータ内包)
├── manifest.json                 # PWA マニフェスト
├── sw.js                         # Service Worker
├── icon-{192,512,180}.png        # アイコン
│
├── hexaco-pi-r-100.json          # 項目100×3言語+採点キー
├── priors_jp_tipi.json           # 日本人 Big Five prior (Kawamoto 2015)
├── priors_jp_darktriad.json      # 日本人 Dark Triad prior (Shimotsukasa 2017)
├── strategies/
│   ├── layerA_clusters.json      # 7クラスタテンプレ
│   ├── layerB_factor_modifiers.json
│   ├── layerC_darktriad_overlays.json
│   └── cue_weights.json          # 101 cue × 因子delta
│
├── research/                     # 抽出データと参考PDF (各findings.json参照)
│   ├── A_demographics/
│   ├── B_culture/
│   ├── C_clusters/
│   ├── D_clinical/
│   └── E_cues/
└── raw/                          # HEXACO 原典 .doc + ScoringKeys PDF
```

## 精度の現実

- **接触前モード** (見ず知らずの相手を thin-slice 観察): Self-Observer 相関は文献値で 0.15〜0.35。**過信は禁物**
- **接触済みモード** (知人): 0.30〜0.50 程度
- 観察可能性: O (Openness) と X (Extraversion) は **読みやすい**、E と A は **観察困難**
- 出力は決定論的だが、入力の解釈ノイズが大きい
- 複数回観察 + 異なる文脈での再評価を推奨

## 倫理上の注意

このツールは **対人理解とコミュニケーション最適化** を目的とします。

- 短期 compliance を得やすいプロファイル (低H・高Dark Triad) ほど、操作的アプローチの後で関係が壊れるリスクが高い
- 推定結果はあくまで確率的な傾向。個別の人物への決めつけや差別的判断には用いるべきでない
- 「憑依キャラクター」機能は初期信頼形成のための rapport building であり、長期関係では素を出さないと続かない

## ローカル開発

```bash
git clone https://github.com/shimatoshi/hexaco-checklist.git
cd hexaco-checklist
# 単に index.html を開く (file:// だと SW は無効、それ以外は動く)
xdg-open index.html

# またはローカルサーバ (SW を有効化したい場合):
python3 -m http.server 8000
# → http://localhost:8000/
```

## ZIM パッケージ化 (Kiwix オフライン配布)

```bash
mkdir pkg
cp index.html manifest.json sw.js icon-*.png pkg/
zimwriterfs --welcome=index.html --language=jpn \
  --title="HEXACO Field Checklist" \
  --description="対人分析チェックリスト" \
  --creator="shimatoshi" --publisher="self" \
  --name="hexaco-checklist" pkg hexaco-checklist.zim
```

## ライセンス

コード (index.html, sw.js, manifest.json, アイコン等): MIT
データ JSON (priors, cue_weights, strategies): CC-BY-4.0
研究PDF (research/): 各論文の著作権は各出版社/著者に帰属。本リポジトリは研究目的のアーカイブとして格納

問題があれば issue を立ててください。
