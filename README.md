# AI Staff Combini - AI店員シミュレーター

コンビニに配置されたAI店員を3Dで実装・制御するWebベースのインタラクティブシステムです。Google Gemini APIを用いた自然言語対話と、WebカメラでのリアルタイムFace Detectionに対応しています。

## 概要

このプロジェクトは、ブラウザ上で3Dモデルのコンビニ店員を表示し、以下の機能を備えたAIシステムです：

- **リアルタイム音声対話**: Google Gemini APIを通じた双方向音声コミュニケーション
- **3Dアニメーション**: 複数のポーズとジェスチャーによる表現力豊かな対話
- **顔検出と反応**: カメラを使用した来店客への自動応答
- **店長への報告機能**: 客からの要望を店長にリアルタイムで通知
- **抽選ゲーム**: ガラガラ抽選との連携

## フォルダ構成

```
ai_staff_combini/
├── index.html                 # メインアプリケーション
├── README.md                  # このファイル
└── static/
    ├── fonts/                 # テキスト表示用フォント
    │   ├── Dosis_Bold.json
    │   ├── Koruri_Bold_Bold.json
    │   └── Noto_Sans_JP_Bold.json
    ├── gltf/                  # 3Dモデルファイル（GLTF形式）
    ├── js/
    │   ├── three.module.js    # Three.jsエンジン本体
    │   └── pico.js            # 顔検出ライブラリ
    └── jsm/                   # Three.js標準モジュール
        ├── geometries/        # ジオメトリ（RoundedBox, Text等）
        ├── loaders/           # 各種ファイルフォーマットローダー
        ├── postprocessing/    # ポストプロセッシング効果
        ├── shaders/           # シェーダープログラム
        └── utils/             # ユーティリティ関数
```

## 使用している技術

### フロントエンド
- **Three.js** - WebGL 3Dグラフィックスエンジン
- **JavaScript (ES6+)** - フロントエンドロジック
- **WebGL** - GPU 3D描画
- **Web Audio API** - 音声入出力処理
- **MediaRecorder API** - マイク音声キャプチャ
- **WebSocket** - サーバーとのリアルタイム通信

### AI・機械学習
- **Google Gemini API** - 自然言語処理と音声対話
- **pico.js** - 顔検出（カスケード分類器）

### 3D・グラフィックス
- **GLTF/GLB フォーマット** - 3Dモデル
- **テクスチャマッピング** - 環境表現
- **スキンニングアニメーション** - キャラクター動作

### 通信
- **WebSocket (WSS)** - Google Generative AI APIとの双方向通信
- **HTTP** - モデル・リソースのロード

## 処理の流れ

### 初期化フロー
```
1. ページロード
   ↓
2. Three.jsシーン・カメラ・レンダラーの初期化
   ↓
3. 3Dモデル(GLTF)のロード
   ↓
4. フォント・テクスチャの読み込み
   ↓
5. 顔検出モデルのロード
   ↓
6. Webカメラアクセス権の取得（Startボタン押下時）
```

### リアルタイム対話フロー
```
1. Startボタン押下
   ↓
2. AudioContext初期化 + マイク入力開始
   ↓
3. 顔検出処理（100msごと）
   ↓
4. 音声キャプチャ（PCM 16bit, 16kHz）
   ↓
5. WebSocket経由でGemini APIへ送信
   ↓
6. Gemini APIが応答
   ↓
7. ツール呼び出し解析
   - ジェスチャー実行（ポーズ変更）
   - 音声再生
   - 店長への通知送信
   ↓
8. 3Dアニメーション実行
   ↓
9. 応答音声の再生
```

### ポーズシステム
AI店員は以下のポーズ/ジェスチャーを実行可能：

| ポーズ | 説明 |
|--------|------|
| `home` | ニュートラルポジション |
| `sleep` | 睡眠状態（Z字アニメーション） |
| `ojigi` | お辞儀（顧客への挨拶） |
| `baibai` | バイバイ（手を振る） |
| `shake_head` | 首振り |
| `shake_body` | 身体の振動 |
| `garagara` | 抽選ガラガラ実行 |

### 顔検出と応答
- カメラフレーム: 400×300px
- 検出間隔: 100ms
- 最大の顔を検出時、カメラ追従・自動起動

### 音声処理
1. **入力**: マイク → AudioWorklet → PCM 16bit → Gemini API
2. **出力**: Gemini API → Base64 PCM → AudioBuffer → 再生

## 機能詳細

### 主要な操作ボタン
- **Start**: 音声対話開始（初期化）
- **home**: ニュートラルポジション復帰
- **baibai / ojigi**: 各ポーズ実行
- **sleep**: 睡眠状態ON/OFF
- **shake_head / shake_body**: 動き
- **garagara**: 抽選ゲーム実行

### 自動応答機能
- **睡眠状態**: 15秒以上音声なしで自動開始
- **自動起動**: 180秒以上操作なし + 顔検出で自動開始
- **顔追従**: 検出された顔位置にカメラ視点が自動追従

### Slackなど外部連携
- 顧客からの要望を `sendMessageStaffツール` で店長に通知
- 抽選結果の自動報告

## セットアップと実行

### 必須条件
- モダンブラウザ（Chrome, Firefox, Edge等）
- マイク・カメラ付きデバイス
- Google Gemini API キー
- HTTPSまたはlocalhostでの実行（セキュリティ要件）

### 実行方法

1. **API Keyの取得**
   - [Google AI Studio](https://aistudio.google.com/app/apikey)からAPIキーを取得

2. **サーバー起動**
   ```bash
   # シンプルな静的サーバーで実行
   python3 -m http.server 8000
   # または
   npx http-server
   ```

3. **ブラウザアクセス**
   ```
   http://localhost:8000/?apiKey=YOUR_API_KEY
   ```

4. **[Start]ボタン押下**
   - マイク・カメラのアクセス許可
   - 対話開始

## 主要なJavaScript処理

### WebSocket管理
```javascript
function startWebSocket(wakeup_text='')
```
Google Gemini APIとのWebSocket接続管理

### ポーズ制御
```javascript
function pose_update()
function pose_home()
```
キャラクターの骨格アニメーション更新

### 音声処理
```javascript
function startStreaming()           // マイク入力開始
function resampleAudio()            // リサンプリング
function convertFloat32ToInt16()    // 形式変換
function playPCMData()              // 音声再生
```

### 顔検出
```javascript
function detectFace()
```
100ms間隔での顔検出と大きさ追跡

## Gemini API プロンプト & ツール仕様

### システムプロンプト

AI店員に以下のシステムプロンプト（System Instruction）を与えています：

```
あなたはコンビニのAIスタッフです。フランクに話してください。
発話するときは、ジェスチャーをツールを使ってどれか選んでください。

コンビニの利用方法について聞かれたら、QRコードを読み取ってスマホに
専用アプリを入れるよう促してください。
決済手段は、dポイント、PayPay、クレジットカードなど各種電子決済が使えます。

お客様からなにか要望を受けた場合は、気軽にsendMessageStaffツールで
店長に教えてください。店長はすぐには対応できませんが、後日対応します。

抽選を求められたらツールでガラガラ(garagara)を実行してください。
ユーザがガラガラを回して結果を伝えてきます。
当たっても特に景品はないので、適当にコメントしてください。

雑談の場合は、お店に増やしてほしい商品など要望がないか聞き出してください。

おすすめの商品は、たらこおにぎりです。
```

### ツール（Function Calling）定義

#### 1. gesture ツール
AI店員がジェスチャーを実行するためのツール

```javascript
{
    "name": "gesture",
    "description": "ジェスチャーをします。",
    "parameters": {
        "type": "object",
        "properties": {
            "pose": {
                "type": "string",
                "enum": ["ojigi", "baibai", "sleep", "shake_head", "shake_body", "garagara"]
            }
        },
        "required": ["pose"]
    }
}
```

**実行ポーズ一覧**:
- **ojigi** - お辞儀（いらっしゃいませ時に実行）
- **baibai** - バイバイ（さよなら時に実行）
- **sleep** - 居眠り（話が不明な場合）
- **shake_head** - 首を横に振る（否定する場合）
- **shake_body** - 体を揺らす（デフォルト・何もすることがない場合）
- **garagara** - 抽選ガラガラ（抽選要求時に実行）

#### 2. sendMessageStaff ツール
顧客からの要望を店長に通知するツール

```javascript
{
    "name": "sendMessageStaff",
    "description": "店長にメッセージを送ります。「お客様が〜と言っています」のような形式で送信してください。",
    "parameters": {
        "type": "object",
        "properties": {
            "text": {
                "type": "string"
            }
        },
        "required": ["text"]
    }
}
```

**使用例**:
- 「お客様が暖かいご飯を置いてほしいと言っています」
- 「お客様がグミを増やしてほしいと言っています」

### Audio設定

Gemini APIとの音声通信設定：

```javascript
generationConfig: {
    responseModalities: ["AUDIO"],          // 音声応答
    speechConfig: {
        voiceConfig: {
            prebuiltVoiceConfig: {
                voiceName: 'Aoede'  // 利用可能: Aoede, Charon, Fenrir, Kore, Puck
            }
        }
    }
}
```

使用している音声：**Aoede**（自然な日本語音声）

### モデル選択

```javascript
model: 'models/gemini-2.5-flash-native-audio-preview-12-2025'
```

Gemini 2.5 Flash with Native Audio対応モデルを使用し、リアルタイムの音声処理を実現

## 環境変数・設定

### URLパラメータ
- `apiKey`: Google Gemini API キー（必須）

### タイムアウト設定
```javascript
const SLEEP_TIMEOUT_MILLSECOND_TH = 15000;      // 15秒
const AUTO_CATCH_START_MILLSECOND_TH = 180000; // 3分
const AUDIO_ACTIVE_TH = 1200;                   // 音声検出閾値
```

## トラブルシューティング

| 問題 | 解決策 |
|------|--------|
| 顔が検出されない | 照明確認、カメラ解像度確認 |
| 音声が出ない | マイク/スピーカー権限確認、オーディオコンテキスト再開 |
| WebSocket接続失敗 | HTTPS使用確認、APIキー有効性確認 |
| 3Dモデル未表示 | GLTF/GLBファイル存在確認、パス確認 |

## 今後の拡張案

- [ ] 複数の3Dモデルキャラクター対応
- [ ] モーション・キャプチャデータ対応
- [ ] 表情制御（BlendShape/MorphTarget）
- [ ] ARモード（WebAR）
- [ ] マルチユーザー対応
- [ ] カスタマイズ機能（衣装変更等）
- [ ] 抽選の景品管理機能
- [ ] 営業時間設定
- [ ] 多言語対応

## ライセンス

（記載してください）

## 参考リンク

- [Three.js Documentation](https://threejs.org/docs/)
- [Google Generative AI API](https://ai.google.dev/)
- [Web Audio API](https://developer.mozilla.org/ja/docs/Web/API/Web_Audio_API)
- [Face Detection Library (pico.js)](https://github.com/nenadmarkus/pico)

---

**最終更新**: 2026年2月25日