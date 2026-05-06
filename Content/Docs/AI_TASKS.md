# AI Tasks

## 完了

### パッケージ化対応（2026-05-06）

- [x] `/Game/Stylized_Room` 参照箇所の特定
  - `Content/StylizedKitchen/Levels/Stylized_Interior.umap` がバイナリ内で `/Game/Stylized_Room` を参照（親Level/ストリーミングLevel）
  - Config内に参照なし。バイナリ(.umap)のみ
- [x] `Config/DefaultGame.ini` に `[/Script/UnrealEd.ProjectPackagingSettings]` セクションを追加
  - `MapsToCook` でゲーム本編マップ10本を明示指定
  - `Stylized_Interior.umap` など ShowcaseマップをCook対象から除外
- [x] `.gitignore` に `images/` を追加（すでに未追跡だったのでgit rmは不要）
- [x] UE5.4 / BuildCookRun によるパッケージ化コマンドを実行 → **BUILD SUCCESSFUL（エラー0件）**
- [x] `Builds\Windows\SausageHunter.exe` と `SausageHunter-Windows.pak`（~706MB）の生成を確認
- [x] `Docs/PACKAGING_REPORT.md` を作成

## パッケージ化後の残タスク（手動確認が必要）

- [ ] `Builds\Windows\SausageHunter.exe` を実行してタイトル画面が出るか確認
- [ ] UINavigation プラグインの動作確認
  - `GlobalDefaultGameMode=/UINavigation/UINavGameMode.UINavGameMode_C` が未解決（Warningのみ）
  - 必要なら UINavigation plugin を `Plugins/` に配置して再Cook
- [ ] ParagonFX マテリアルの見た目確認（多数がSM6/SM5コンパイル失敗）
- [ ] `Soseji_Hunter.uproject` 削除 / `SausageHunter.uproject` 追加のgitコミット
- [ ] Shipping設定での最終パッケージ化（`-clientconfig=Shipping`）

### 旧来の作業（以前のスレッド）

- 新スレッドで既存Markdownを再読込。
- `images/` のスクリーンショット156枚をコンタクトシート化して画面種別を分類。
- 代表画像を目視確認し、`AIC_Enemy_Base`, `BB_Enemy_Base`, 敵タイプ別BT, SubTree, EQS, `BP_Enemy_Base` の追加根拠を整理。
- `Docs/AI_SCREENSHOT_REVIEW.md` を追加。
- `Soseji_Hunter.uproject` をUE 5.4でロード。
- AI関連の Blueprint / Behavior Tree / Blackboard / EQS / Enum をUE経由でロードして解析。
- `BB_Enemy_Base` の key 型を確認。
- `AIC_Enemy_Base` の主要Event / Function / Blackboard key name / Perception参照を確認。
- `BP_Enemy_Base` の `BPI_EnemyAI` / `BPI_Damagable` 実装グラフ名を確認。
- `ReserveAttackToken` / `ReturnAttackToken` / `StoreAttackTokens` / `GetIdealRange` 参照を確認。
- `BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss` の参照SubTree / Decorator / Service / Task / EQSを確認。
- `EQS_FindCover`, `EQS_FindIdealRangedLocation`, `EQS_Strafe`, `EQS_Teleport`, `EQS_Seeking`, `EQS_FurthestPOIActor` のGenerator/Test系参照を確認。
- `Spawner_BP` のWave / EnemyType / SpawnAIFromClass / Boss map参照を確認。
- Stage / Boss maps の ExternalActors に `Spawner_BP`, `BP_PatrolRoute`, `BP_POI`, `BP_Enemy_Boss` 参照があることを確認。
- `Config/DefaultEngine.ini` の `GameName` を `Soseji_Hunter` に修正。
- `Config/DefaultGame.ini` の `ProjectName` を `Soseji_Hunter` に修正。
- `Config/DefaultGame.ini` の `StartupActions` を `bAddPacks=False` に修正。
- `README.md` をポートフォリオ向けに上書き。

## 追加で目視確認するとさらに強くなること

0. 今回のスクショレビューで追加確認したいこと
   - `BTT_RangedAttack` のTarget Keyが `SelfActor` に見える理由。
   - `ReserveAttackToken` / `ReturnAttackToken` のトークン数と返却漏れ対策。
   - `E_AIState` の6番目の表示名。
   - EQS Testの重み/Filter/Scoreの詳細。
   - どの `BTT_*`, `BTD_*`, `BTS_*` を自分が作ったかの記憶確認。

1. `AIC_Enemy_Base` Event Graph のスクリーンショット
   - `OnPerceptionUpdated`
   - `HandleSensedSight`
   - `HandleSensedSound`
   - `HandleSensedDamage`
   - `HandleLostSight`
   - `SeekAttackTarget`

2. `BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss` のスクリーンショット
   - Root直下の左から右の分岐順
   - Decorator の条件
   - Observer Abort
   - Service Tick間隔
   - Taskの key selector / threshold

3. `Spawner_BP` のDetails / Wave配列
   - Waveごとの敵タイプ
   - 生成数
   - 生成位置
   - Boss map遷移条件

4. `EQS_*` 画面
   - Generatorの範囲/半径/点数
   - Distance / Trace / Pathfinding Test の重みとFilter

## ES/READMEで避ける表現

- 敵AIを完全に一から全部自作。
- Unreal Engine のAI PerceptionやEQSそのものを自作。
- 参考資料を使っていないように見せる表現。
- 現場レベルの高度な戦術AIや最適化を実装。
- Root順やDecorator設定を、未確認の数値込みで断言。

## ES/READMEで使いやすい表現

- 敵AIを Blueprint ベースで実装・調整した。
- Behavior Tree / Blackboard / AI Controller / AI Perception / EQS を用いた敵AIを担当した。
- 近接・遠距離・魔法・BossでBehavior Treeを分け、共通Blackboardで状態とターゲットを管理した。
- 視覚・聴覚・被ダメージをきっかけに戦闘状態へ遷移し、見失った場合は調査・探索状態へ移る構成にした。
- EQSを使い、カバー、理想距離、ストレイフ、テレポート、探索地点を選ぶ仕組みを組み込んだ。
- UE5の解説記事や動画も参考にしながら、本作の敵タイプやステージ構成に合わせてAIを組み込み・調整した。
