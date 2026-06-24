# キーボードなし文字入力ソフト

椅子にもたれかかった楽な姿勢のまま、キーボードに手を伸ばさずマウス操作（フリック等）だけで快適・高速に文字入力を行うためのWebアプリケーションです。

## 1. 要件定義（5要素）

* **【目的】** 椅子にもたれかかった楽な姿勢のまま、キーボードに手を伸ばさずマウス操作（フリック等）だけで快適・高速に文字入力を行う。
* **【利用者の入出力】** 画面上のUIパネル（文字盤）を見ながら、マウスの動きで文字を選択・入力する ➔ 入力されたひらがながテキストエリア等に表示される。
* **【制約】** Webブラウザ上で動作するWebアプリであること。
* **【受け入れ基準】** 最低限、ひらがな50音の入力と表示ができること。
* **【非目標】** カタカナや英数字モードの実装、漢字変換などは今回は作らない（スコープ外）。

## 2. 機能一覧と優先度

### コア機能（アプリの核となる部分）

* **マウスジェスチャー（フリック）認識機能** 【優先度：高】
* マウスの動いた方向（上下左右など）を検知して、どの文字が選ばれたかを判定する機能。


* **文字盤UI表示機能** 【優先度：高】
* スマホのフリック画面のような、視覚的に分かりやすい入力パネルを画面に表示する機能。


* **テキスト出力エリア** 【優先度：高】
* 入力されたひらがなを画面上に次々と表示していくエリア。



### サポート機能（あると便利な部分）

* **一字削除（バックスペース）機能** 【優先度：高】
* 入力を間違えたときに、マウス操作で1文字消せる機能。


* **テキストコピー機能** 【優先度：中】
* 入力したひらがなを一発でクリップボードにコピーして、他のサイトやアプリに貼り付けられるようにする機能。


* **入力モード切り替え（特定の場合のみ起動）** 【優先度：中】
* 常にマウス入力を受け付けると疲れるため、特定のボタンを押している間だけ、または特定の範囲内だけでフリック入力を有効にする切り替え機能。



### 共通機能（システム全体のベース）

* **画面レイアウト（HTML/CSS）** 【優先度：高】
* 文字盤とテキストエリアをブラウザ上にきれいに配置するベース画面。



## 3. 機能要求と非機能要求の分類

### 機能要求（システムが提供する機能）

* マウスジェスチャー（フリック）認識機能
* 文字盤UI表示機能
* テキスト出力エリア表示機能
* 一字削除（バックスペース）機能
* テキストコピー機能
* 入力モード切り替え機能（特定時のみ有効化）

### 非機能要求（システムが満たすべき品質や制約）

* **性能（Performance）**：マウスの動きに対して、遅延（タイムラグ）なくスムーズに文字盤の選択や入力が反映されること。
* **セキュリティ（Security）**：入力された文字データを外部サーバーに送信せず、ブラウザ内（ローカル）だけで安全に処理すること。
* **ユーザビリティ（Usability）**：椅子にもたれて画面から少し離れても文字盤が見やすいよう、UIの大きさや配色に配慮すること。疲労防止のため、特定の操作時のみフリックが反応する誤操作・疲労防止策を入れること。
* **保守性（Maintainability）**：将来的に「カタカナ」や「英数字」のモードを追加したくなったとき、コードを大きく書き直さずに拡張できるようなシンプルな設計にしておくこと。

## 4. 設計図（Mermaid）

### ① ユースケース図風の図

```mermaid
flowchart LR
    User(("ユーザー"))

    subgraph app ["キーボードなし文字入力ソフト"]
        UC1["入力モードを切り替える"]
        UC2["ひらがなをフリック入力する"]
        UC3["マウスフリックを認識する"]
        UC4["文字盤UIを表示する"]
        UC5["1文字削除する"]
        UC6["テキストをコピーする"]
    end

    User --> UC1
    User --> UC2
    User --> UC5
    User --> UC6

    UC2 -.->|include| UC3
    UC2 -.->|include| UC4

```

### ② クラス図

```mermaid
classDiagram
    class InputApp {
        -isInputModeActive: boolean
        +init() void
        +toggleInputMode() void
        +copyToClipboard() void
    }

    class CharacterPanel {
        -panelElement: Object
        -currentCenterChar: string
        +render() void
        +showFlickGuide(direction: string) void
        +hideFlickGuide() void
    }

    class GestureRecognizer {
        -isTracking: boolean
        -startX: number
        -startY: number
        +startTracking(x: number, y: number) void
        +detectDirection(currentX: number, currentY: number) string
        +stopTracking() string
    }

    class TextBuffer {
        -text: string
        +addCharacter(char: string) void
        +deleteLastCharacter() void
        +getText() string
    }

    InputApp "1" --> "1" CharacterPanel : 画面表示を制御
    InputApp "1" --> "1" GestureRecognizer : マウスの動きを検知
    InputApp "1" --> "1" TextBuffer : 入力文字列を保持

```

### ③ シーケンス図

```mermaid
sequenceDiagram
    autonumber
    actor User as ユーザー
    participant UI as CharacterPanel (UI)
    participant Ctrl as InputApp (コントローラ)
    participant Model as TextBuffer (モデル)

    Note over User, Model: 入力モードが有効な状態でのフリック入力ループ
    loop 文字入力の繰り返し
        User->>UI: マウスをドラッグ（フリック開始）
        UI->>Ctrl: マウス移動イベント通知
        Ctrl->>UI: フリック方向のガイド画面を表示

        User->>UI: マウスを離す（フリック終了）
        UI->>Ctrl: マウスアップイベント通知
        
        Ctrl->>Ctrl: ジェスチャーから文字（ひらがな）を確定
        Ctrl->>Model: addCharacter(確定文字)
        Model-->>Ctrl: 最新の文字列データ
        Ctrl->>UI: テキスト表示エリアを更新(最新文字列)
    end

    alt 1文字消したい場合（バックスペース操作）
        User->>UI: 削除ボタン（または特定のジェスチャー）
        UI->>Ctrl: 削除要求を通知
        Ctrl->>Model: deleteLastCharacter()
        Model-->>Ctrl: 1文字削除後の文字列データ
        Ctrl->>UI: テキスト表示エリアを更新(最新文字列)
    end

```

### ④ 状態遷移図

```mermaid
stateDiagram-v2
    [*] --> Disabled : アプリ起動（初期状態）

    state Disabled {
        [*] --> 無効状態 : マウス入力OFF
    }

    state Enabled {
        [*] --> Idle : 待機中（マウス静止）
        Idle --> Tracking : マウスのドラッグ開始（フリック検知）
        Tracking --> Tracking : マウス移動中（方向を判定・ガイド表示）
        Tracking --> Idle : マウスを離す（文字確定・入力完了）
    }

    無効状態 --> Idle : 特定の起動トリガー（特定エリアへの進入 or 特定キー押下）
    Enabled --> 無効状態 : 終了トリガー（エリア外への移動 or 解除操作）

    Disabled --> [*] : ブラウザを閉じる（終了状態）

```

## 5. 機能規模見積もり（COSMIC法）

主要機能におけるデータの移動を分析し、機能規模を見積もった結果は以下の通りです。

* **① アプリ初期化と画面表示 (4 CFP)**
* アプリがブラウザで起動する（Entry: 1）
* ローカル設定を読み込む（Read: 1）
* 文字盤UIとテキストエリアを表示する（Exit: 1）
* 初期文字盤配置データを保持する（Write: 1）


* **② 入力モード切り替え (3 CFP)**
* ユーザーが切り替えトリガーを操作する（Entry: 1）
* モード状態を更新する（Write: 1）
* 文字盤UIの表示状態を切り替える（Exit: 1）


* **③ 文字フリック操作（ドラッグ中）(3 CFP)**
* ユーザーがマウスを動かす（Entry: 1）
* 現在のマウス座標に対応する文字データを読み込む（Read: 1）
* 選択中方向のガイドをハイライト表示する（Exit: 1）


* **④ 文字確定（フリック終了）(4 CFP)**
* ユーザーがマウスを離す（Entry: 1）
* 確定したひらがなを入力バッファに書き込む（Write: 1）
* 最新の文字列を入力バッファから読み出す（Read: 1）
* テキストエリアを最新文字列に更新する（Exit: 1）


* **⑤ 1文字削除機能 (4 CFP)**
* ユーザーが削除操作を行う（Entry: 1）
* 現在の文字列を読み出す（Read: 1）
* 末尾の1文字を削除してバッファを書き換える（Write: 1）
* テキストエリアの表示を更新する（Exit: 1）


* **⑥ 全文字削除機能 (3 CFP)**
* ユーザーがクリアボタンを押す（Entry: 1）
* 入力バッファを空にリセットする（Write: 1）
* テキストエリアの表示を空にする（Exit: 1）


* **⑦ テキストコピー機能 (4 CFP)**
* ユーザーがコピーボタンを押す（Entry: 1）
* 入力バッファから文字列データを読み出す（Read: 1）
* クリップボードへ文字列を出力する（Exit: 1）
* 画面にコピー完了の通知を表示する（Exit: 1）



**【合計機能規模】：25 CFP**

## 6. 開発環境・起動方法

### 使用技術（予定）

* HTML5
* CSS3
* JavaScript (Vanilla JS)

### 起動方法

1. 本リポジトリをクローンまたはダウンロードします。
2. `index.html` を任意のWebブラウザで開きます。
