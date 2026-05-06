# Soseji_Hunter

ホットドッグになろうと襲ってくるソーセージたちから、パンの主人公が身を守る3Dアクションゲームです。  
3つのステージを攻略し、各ステージのウェーブ戦とボス戦を突破するとクリアになります。

## プロジェクト概要

| 項目 | 内容 |
| --- | --- |
| タイトル | Soseji_Hunter (リポジトリ名 `Sausage_Hunter_UE5`) |
| ジャンル | 3Dアクション |
| プレイ人数 | 1人 |
| 想定プレイ時間 | 約10分 |
| 開発環境 | Unreal Engine 5.4 / Blueprint |
| 言語 | Blueprint Visual Scripting のみ (C++ 不使用) |
| 入力デバイス | Xboxコントローラー |
| 制作期間 | 大学1年時、4人チームで制作 |

## デモ

- プレイ動画: URL
- 写真 / GIF: 追加予定

## ゲームの流れ

1. タイトル画面でステージと難易度を選択する。
2. ステージ内に出現する敵を倒しながら進む。
3. 1ステージは通常ウェーブ3回とボス戦で構成される。
4. 武器、魔法、スキル、回避を使い分けて敵を倒す。
5. HPが0になるとゲームオーバー、ボスを倒すとリザルトへ遷移する。

## 操作方法

| 操作 | 内容 |
| --- | --- |
| 左スティック | 移動 |
| 右スティック | カメラ操作 |
| 右スティック押し込み | ロックオン |
| 右トリガー | 攻撃 |
| Aボタン | ジャンプ |
| Bボタン | 回避 |
| LB + X/Y/A/B | スキル使用 |

## 自分の担当

4人チームで制作し、私は主に敵AI関連を担当しました。

- 共通AI Controller (`AIC_Enemy_Base`) と Blackboard (`BB_Enemy_Base`) の構成
- AI Perception による視覚・聴覚・被ダメージ検知と状態遷移
- 敵タイプ別Behavior Tree (近接 / 遠距離 / 魔法 / ボス) の構成
- 共通行動 (待機 / 調査 / 探索 / 凍結) のSubTree化
- EQS によるカバー、理想距離、ストレイフ、テレポート、探索地点の候補選択
- ウェーブスポナーと敵AIの連携

UE5のAI機能(AI Perception、Behavior Tree、Blackboard、EQS)については、解説記事や動画も参考にしながら学習しました。基本構成を本作の敵タイプに合わせて組み込み・調整し、近接、遠距離、魔法、ボスごとの攻撃距離、探索、回復、テレポートに合わせて分岐とパラメータを作りました。

| 担当者 | 主な担当 |
|--------|----------|
| 自分 | 敵AI全般、ウェーブスポナーと敵AIの連携 |
| メンバーA | 環境ダメージエフェクト(HeatDistortion)、剣攻撃エフェクト(スラッシュ)、ステージクリアやリザルト・ゲームオーバーへの遷移処理、移動時の焦げ跡演出、インベントリのアイテム使用・消費処理 |
| メンバーB | UI/演出全般 (操作方法表示、HP/Heatゲージ、タイトル/リザルトウィジェット遷移、ホーム/リトライボタン) |
| メンバーC | 3Dモデル制作、アニメーション、企画 |

チーム制作のため、他メンバーが制作した素材・演出・UI・モデルを自分の成果として誤解されないよう、担当範囲を分けて記載しています。

## 設計判断

敵AIは、共通の判断情報(検知、状態管理、ターゲット、距離)を共有しながら、敵タイプごとの戦闘行動だけを分けて実装することを意識しました。具体的には、`AIC_Enemy_Base` でAI Perceptionと状態遷移を扱い、`BB_Enemy_Base` で敵タイプ間の共通情報(攻撃対象、状態、探索地点、距離)を保持し、`BT_Enemy_Melee / Ranged / Mage / Boss` で各敵タイプの戦闘行動を切り替える構成にしました。

待機・調査・探索・凍結のように敵タイプによらず共通する行動はSubTree化し、`BT_SubTree_Passive` などとして複数の敵で再利用しています。これにより、敵タイプを増やすときに状態管理を作り直す必要がなく、戦闘行動のBTだけを追加すればよい構成になっています。

位置取りは固定座標ではなくEQSで候補を選ぶ形にしました。`EQS_FindCover`、`EQS_FindIdealRangedLocation`、`EQS_Strafe`、`EQS_Teleport`、`EQS_Seeking` といった用途別のEQSを用意し、敵ごとに異なる戦い方を作っています。

## Unreal Engine のバージョン

- **UE 5.4**
- ビルド構成: Blueprint のみ (C++ 不使用)

## 敵AIシステム

敵AIは、共通のAI ControllerとBlackboardを中心に構成しています。プレイヤーを視覚・聴覚・被ダメージで検知し、Blackboardへ現在状態、攻撃対象、探索地点、攻撃距離などを保存します。その情報をBehavior Treeが参照し、巡回、探索、追跡、攻撃、回復、テレポートなどを切り替えます。

### AI構成

| アセット | 役割 |
| --- | --- |
| `AIC_Enemy_Base` | 通常敵の共通AI Controller。AI Perceptionでプレイヤーを検知し、Blackboardを更新する |
| `AIC_Enemy_Boss` | ボス専用AI Controller。ボスHP UIやボス用挙動と連携する |
| `BB_Enemy_Base` | 敵AI共通のBlackboard。ターゲット、状態、探索地点、距離情報を保持する |
| `BT_Enemy_Base` | 共通敵AIの状態分岐。待機、調査、凍結などの基礎行動を扱う |
| `BT_Enemy_Melee` | 近接敵用BT。接近、武器装備、近接攻撃、理想距離への移動を行う |
| `BT_Enemy_Ranged` | 遠距離敵用BT。射線確保、カバー探索、射撃、回復を行う |
| `BT_Enemy_Mage` | 魔法敵用BT。魔法攻撃、回復、テレポート、距離維持を行う |
| `BT_Enemy_Boss` | ボス用BT。近接攻撃、ジャンプ攻撃、AOE、テレポート、浮遊などを行う |
| `BT_SubTree_Passive` | 巡回または待機の共通行動 |
| `BT_SubTree_Investigating` | 最後に感知した地点へ移動して確認する |
| `BT_SubTree_Seeking` | 見失った対象を探索する |
| `BT_SubTree_Frozen` | 凍結・行動不能状態用 |
| `BP_Enemy_Base` | HP、被ダメージ、攻撃トークン、Interface、AIController/BTの共通基底 |
| `Spawner_BP` | ウェーブごとに敵を生成し、ステージ進行やボス戦へつなげる |

### Blackboard

`BB_Enemy_Base` には以下のキーを用意しています。

| Key | 型 | 用途 |
| --- | --- | --- |
| `SelfActor` | Object / Actor | 自分自身 |
| `AttackTarget` | Object / Actor | 攻撃対象のプレイヤー |
| `State` | Enum (`E_AIState`) | 現在のAI状態 |
| `PointOfInterest` | Vector | 調査・探索・移動先 |
| `AttackRadius` | Float | 攻撃に入る距離 |
| `DefendRadius` | Float | 距離維持・防御寄りの判断に使う距離 |
| `DistanceToAttackTarget` | Float | ターゲットとの現在距離 |

### AIの状態遷移

基本的な考え方は以下です。

1. 通常時は `Passive` として待機または巡回する
2. プレイヤーを視認、音で感知、または攻撃を受けると `AttackTarget` を設定する
3. `State` を攻撃状態へ変更し、敵タイプごとのBTで攻撃行動を選ぶ
4. プレイヤーを見失った場合は `PointOfInterest` を使って最後に感知した地点を調べる
5. 一定時間探して見つからない場合は待機・巡回状態へ戻る (`TimeToSeekAfterLosingSight = 3.0`)
6. 凍結などの状態異常が入った場合は `Frozen` 用のSubTreeへ切り替える

### 敵タイプ別の挙動

| 敵タイプ | プレイヤー目線の挙動 | 実装のポイント |
| --- | --- | --- |
| 近接敵 | プレイヤーへ接近し、近距離で武器攻撃を行う。距離が離れると再接近する | `BT_Enemy_Melee` で `DistanceToAttackTarget` を見て、接近・武器装備・近接攻撃・理想距離への移動を切り替える。`EQS_Strafe` で横移動候補を持つ |
| 遠距離敵 | プレイヤーから距離を取り、射線が通る位置から攻撃する。低HP時には回復する | `EQS_FindCover` / `EQS_FindIdealRangedLocation` を利用。`BTD_CanSeeTarget`、`BTD_IsHealthBelowThreshold` で行動を切り替える |
| 魔法敵 | 魔法攻撃、回復、テレポートを行う。近づかれた時に距離を作る | `BT_Enemy_Mage` と `EQS_Teleport` で移動先を選ぶ。近距離時には GroundSmash 系の分岐がある |
| ボス | 複数の攻撃パターンを持ち、ジャンプ攻撃、範囲攻撃、テレポート、浮遊、無敵/割り込み制御を行う | `BT_Enemy_Boss` と `BP_Enemy_Boss` で攻撃演出・HP UI・特殊行動を制御 |

## 実装上の工夫

- 敵の共通情報を `BB_Enemy_Base` に集約し、敵タイプごとの差分を個別のBehavior Treeで分けることで、共通の状態管理を使いながら敵ごとの個性を出せる構成にしました
- `AIC_Enemy_Base` の `OnPerceptionUpdated` で、視覚・聴覚・被ダメージをまとめて扱い、`HandleSensedSight` / `HandleSensedSound` / `HandleSensedDamage` に分けて処理する形で、戦闘開始の入口を統一しました
- `KnownSeenActors` と `CheckForgottenSeenActor` のTimerで、見失い処理をPerceptionイベントだけに依存させない構成にしています
- `BPI_EnemyAI` を使い、BT Taskから敵キャラクター側の `AttackStart`、`SetMovementSpeed`、`GetIdealRange`、`GetPatrolRoute` を呼び出せるようにしました
- `BPI_Damagable` を使い、HP、被ダメージ、死亡判定、回復処理、チーム番号、攻撃トークンをAI側から扱えるようにしました
- `On Same Team` の判定で、自分のControlled Pawnと相手Actorの `Get Team Number` を比較し、味方を攻撃対象から除外しています
- `BTS_UpdateDistanceToTarget` でターゲットとの距離を更新し、攻撃・接近・距離維持の判断に使いました
- `BTS_StopAttackingIfTargetIsDead` により、ターゲットが死亡した場合に攻撃状態を解除します
- `BT_SubTree_Passive`、`BT_SubTree_Investigating`、`BT_SubTree_Seeking`、`BT_SubTree_Frozen` を分け、複数の敵で共通行動を再利用しました
- `EQS_FindCover`、`EQS_FindIdealRangedLocation`、`EQS_Strafe`、`EQS_Teleport`、`EQS_Seeking` を使い、固定座標ではなく環境クエリで位置取りや探索地点を選びました
- 攻撃処理は `OnAttackEnd`、`OnWeaponEquipped`、`OnTeleportEnd`、`OnHealOverTimeEnd` などのイベント完了を待ってBT Taskを終了する構成にし、アニメーション・攻撃処理・AI判断を分離しました
- `ReserveAttackToken` / `ReturnAttackToken` により、複数の敵が同時に攻撃しすぎないように制御する仕組みを用意しました

## ウェーブ・スポーン

`Spawner_BP` でステージごとの敵生成を管理しています。通常ステージではウェーブごとに近接・遠距離・魔法敵を出現させ、各敵に対応するBehavior Treeを割り当てます。ステージ終盤ではボス用マップへ遷移し、ボス専用AI ControllerとBehavior Treeでボス戦を行います。

## チームメンバーが担当した実装

私の担当範囲ではないため概要のみ記載しますが、本作には以下の機能が他メンバーによって実装されています。

### ゲームプレイ系 (メンバーA)

- Niagara/パーティクルを使った環境ダメージエフェクト (HeatDistortion)
- 剣攻撃エフェクト (スラッシュ)
- ステージクリアによるウェーブ進行、リザルト遷移、ゲームオーバー遷移処理
- 移動時に発生する足跡 (焦げ跡) 演出
- アイテム取得、インベントリ格納、使用時の消費処理 (数が0になると表示が消える)

### UI / 演出 (メンバーB)

- ゲーム開始時の左下に操作方法を表示
- HPゲージ、ヒートゲージ、ボスHPゲージ
- タイトル、ステージ選択、リザルト、ゲームオーバー画面のウィジェット手順遷移
- ホームボタン、リトライボタン

### 3Dモデル / アニメーション / 企画 (メンバーC)

- キャラクター・敵の3Dモデル制作
- アニメーション制作
- ゲーム企画

## 既知の課題

- 攻撃エフェクトのサイズや位置が一部ずれている
- アイテムが使用できなくなる場合がある
- スキル使用直後など、一部アニメーション遷移が不自然になる場合がある
- AI関連はBlueprint中心のため、TaskやDecoratorが増えると全体像を追いにくい。今後はDataAsset化やデバッグ表示の追加で調整しやすくしたい

## ライセンス

このリポジトリ内のコードおよび自作素材のライセンスは未設定です。第三者アセットは各配布元・購入元のライセンスに従います。

### このリポジトリに含まれていないアセット

再配布制限のあるアセットは `.gitignore` でリポジトリから除外しています。これらはローカルで開発時に使用していますが、GitHub には含めていません。

| 種別 | 除外したパス | 理由 |
|------|-------------|------|
| Marketplace / Fab エフェクトパック | `Content/FXVarietyPack/`, `Content/StylizedKitchen/`, `Content/sA_SplatterofbloodFx/`, `Content/Effects/tool/` | Marketplace Standard License により、アセット単体の再配布が禁止されているため |
| Mixamo アニメーション・キャラクター | `Content/Animations/Melee/`, `Content/Animations/Axe/`, `Content/Animations/Rifle/`, `Content/Animations/Magic/`, `Content/Characters/Mage/Mesh/`, `Content/Enemies/*/Animation/` | Mixamo (Adobe) からダウンロードしたアセット。利用規約上、`.uasset` としてのリポジトリ公開に制限がある可能性があるため |
| Epic Games 無料公開アセット | `Content/ParagonFengMao/`, `Content/ParagonGideon/`, `Content/ParagonSparrow/`, `Content/InfinityBladeEffects/`, `Content/StarterContent/` | Epic Marketplace Standard License により、アセット単体の再配布制限の懸念があるため |

これらのアセットを使用してゲームをビルドするには、各配布元から個別にダウンロードして該当パスに配置する必要があります。

### 使用アセット・外部素材

| 種別 | 配置場所 | 出典・ライセンス |
|------|---------|----------------|
| Unreal Engine テンプレート / Starter Content | `Content/StarterContent/` 等 (除外済み) | Epic Games / Unreal Engine 付属コンテンツ。Unreal Engine EULA の範囲で使用 |
| Epic Games 無料公開アセット (Paragon系) | `Content/ParagonFengMao/`, `Content/ParagonGideon/`, `Content/ParagonSparrow/` (除外済み) | Epic Games が Marketplace で無料公開した Paragon シリーズ |
| Epic Games 無料公開アセット (Effects系) | `Content/InfinityBladeEffects/` (除外済み) | Epic Games が無料公開した Infinity Blade: Effects |
| Lyra Starter Game | `Content/Weapons/Meshes/Rifle/SM_Rifle.uasset` | Epic Games の Lyra Starter Game サンプル。Unreal Engine EULA の範囲で使用 |
| Mixamo アニメーション・キャラクターモデル | `Content/Animations/`, `Content/Characters/Mage/Mesh/` (除外済み) | Mixamo (Adobe)。Adobe アカウントでダウンロードし、ゲームへの組み込みは許可される範囲で利用 |
| Marketplace / Fab エフェクトパック | `Content/FXVarietyPack/`, `Content/StylizedKitchen/`, `Content/sA_SplatterofbloodFx/`, `Content/Effects/tool/` (除外済み) | Marketplace / Fab のパックを使用 |
| 自作 3D モデル・武器・アイテム | `Content/Pan/Mesh/`, `Content/Enemies/hurank/Mesh/`, `Content/Enemies/Sarami/Mesh/`, `Content/Enemies/pork/mesh/`, `Content/ThirdPerson/MapAssets/`, `Content/Inventory/Mesh/`, `Content/Weapons/Meshes/fork/`, `Content/Weapons/Meshes/Knife/` | 自作。著作権はチームメンバー(モデル制作担当)に帰属 |
| 自作 Blueprint / マップ | `Content/Enemies/AI/`, `Content/ThirdPerson/Blueprints/`, `Content/Player/`, `Content/Title/`, `Content/Widgets/`, `Content/ThirdPerson/Maps/` | 自作。著作権はプロジェクト制作チームに帰属 |
| 生成AI画像素材 | `Content/Title/Picture/ChatGPT_Image_*`, `Content/Title/Picture/Gemini_Generated_Image_*`, `Content/ThirdPerson/image/ChatGPT_Image_*` | OpenAI (ChatGPT/DALL-E) / Google Gemini の生成AIで作成 |
| BGM / SE / ボイス | `Content/sound/BGM/`, `Content/sound/SE/`, `Content/sound/boss/`, `Content/sound/pan/`, `Content/sound/sosage/`, `Content/Audio/`, `Content/Projectiles/Audio/` | 出典の確認を進めている |

### 参照した主なライセンス情報

- Unreal Engine EULA: https://www.unrealengine.com/eula
- Epic Marketplace Standard License: https://www.unrealengine.com/marketplace-distribution-agreement
- Fab Standard License: https://www.fab.com/eula
- Mixamo Terms of Use: https://helpx.adobe.com/jp/creative-cloud/faq/mixamo-faq.html
- OpenAI Terms of Use: https://openai.com/policies/terms-of-use/
- Google Gemini Terms: https://policies.google.com/terms/generative-ai

## 生成AIの利用について

本作の制作過程で、以下のように生成AIを利用しました。

| 用途 | 使用したAI | 利用範囲 |
|------|------------|---------|
| タイトル・UI 画像素材 | ChatGPT / DALL-E (OpenAI)、Google Gemini | `Content/Title/Picture/`、`Content/ThirdPerson/image/` 配下の画像素材作成 |
| コード生成・調査支援 | Claude Code (Anthropic) | パッケージ化エラーの原因調査・修正案立案、`Config/DefaultGame.ini` への MapsToCook 追記、`.gitignore` 修正、ドキュメント作成支援 |

最終的な仕様判断、Blueprint の設計・実装、Behavior Tree 構造の設計、EQS 設定、マップ配置、ゲームプレイの仕様決定は自分(およびチームメンバー)で行っています。生成素材については、提出物で使用可能な範囲か確認したうえで利用しています。
