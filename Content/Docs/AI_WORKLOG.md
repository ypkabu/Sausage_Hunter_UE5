# AI Worklog

## 2026-05-06 パッケージ化対応

- プロジェクト全体を調査（uproject, Config/*.ini, Content一覧）
- `/Game/Stylized_Room` 参照箇所を特定:
  - 参照元: `Content/StylizedKitchen/Levels/Stylized_Interior.umap`（バイナリ内）
  - Config内には参照なし
  - ExternalActors 内の "Stylized_Room" ラベルのアクター（Stage1〜3, Boss1〜3）は  
    アクター表示名のみで `/Game/Stylized_Room` パッケージへの参照はなし
- `Config/DefaultGame.ini` に `[/Script/UnrealEd.ProjectPackagingSettings]` セクションを追加し、  
  ゲーム本編マップ10本の `MapsToCook` を明示指定。これにより `Stylized_Interior.umap` が除外される
- `.gitignore` に `images/` を追加（すでに未追跡だったのでgit rm不要）
- UE5.4 / BuildCookRun でパッケージ化を実行 → **BUILD SUCCESSFUL (エラー0件)**
- `Builds\Windows\SausageHunter.exe` と `SausageHunter-Windows.pak` (~706MB) を確認
- `Docs/PACKAGING_REPORT.md` を新規作成

## 2026-05-06 AIスクリーンショット・Markdownレビュー

- 新スレッドで再開し、既存Markdownをすべて再確認。
- `images/` の156枚を俯瞰するため、`Saved/ImageContactSheets/contact_sheet_01.jpg` から `contact_sheet_08.jpg` と `manifest.tsv` を作成。
- 代表スクリーンショットを目視確認。
  - `BB_Enemy_Base` のキー一覧。
  - `AIC_Enemy_Base` の Possess / Perception / Sight / Hearing / Damage / 忘却 / 状態設定。
  - `BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss`, SubTree群。
  - `EQS_FindCover`, `EQS_FindIdealRangedLocation`, `EQS_Teleport`, `EQS_Context_AttackTarget`。
  - `BP_Enemy_Base` の死亡、被弾、攻撃トークン関連。
- スクショ読解の追加レポートとして `Docs/AI_SCREENSHOT_REVIEW.md` を作成。

## 2026-05-06 earlier

- プロジェクト内のMarkdownを確認。
  - 既存は `README.md`。
  - `Docs/AI_CONTEXT.md`, `Docs/AI_TASKS.md`, `Docs/AI_WORKLOG.md`, `Docs/AI_PORTFOLIO_ANALYSIS.md` を作成・更新。
- `Soseji_Hunter.uproject` を確認。
  - Unreal Engine 5.4。
  - `MeleeTrace` plugin は disabled。
- `Config` を確認・修正。
  - `DefaultEngine.ini` の古いGameNameを `GameName=Soseji_Hunter` に修正。
  - `DefaultEngine.ini` の `ActiveGameNameRedirects` を `/Script/Soseji_Hunter` 向けに整理。
  - `DefaultGame.ini` のテンプレート由来ProjectNameを `ProjectName=Soseji_Hunter` に修正。
  - `DefaultGame.ini` の `StartupActions` を `bAddPacks=False` に修正。
- Unreal Editor 5.4 / Commandlet を使ってAI関連アセットをロード。
  - 調査用スクリプトとJSONは `Saved/` 配下。
  - `Saved/` はGit ignored。
  - `open_editor_for_assets` は headless / nullrhi ではクラッシュするため、ロード済みUObject/CDOと文字列抽出で解析。
- `BB_Enemy_Base` のkeyと型を確認。
  - `SelfActor`: Object / Actor
  - `AttackTarget`: Object / Actor
  - `State`: Enum / `E_AIState`
  - `PointOfInterest`: Vector
  - `AttackRadius`: Float
  - `DefendRadius`: Float
  - `DistanceToAttackTarget`: Float
- `AIC_Enemy_Base` を確認。
  - `AIPerceptionComponent`
  - `AISense_Sight`
  - `AISense_Hearing`
  - `AISense_Damage`
  - `OnPerceptionUpdated`
  - `HandleSensedSight`
  - `HandleSensedSound`
  - `HandleSensedDamage`
  - `HandleLostSight`
  - `SeekAttackTarget`
  - `TimeToSeekAfterLosingSight = 3.0`
- Enemy BP のAIController / BehaviorTree を確認。
  - `BP_Enemy_Base` -> `AIC_Enemy_Base`, `BT_Enemy_Base`
  - `BP_Enemy_Melee` -> `AIC_Enemy_Base`, `BT_Enemy_Melee`
  - `BP_Enemy_Ranged` -> `AIC_Enemy_Base`, `BT_Enemy_Ranged`
  - `BP_Enemy_Mage` -> `AIC_Enemy_Base`, `BT_Enemy_Mage`
  - `BP_Enemy_Boss` -> `AIC_Enemy_Boss`, `BT_Enemy_Boss`
- `BP_Enemy_Base` のInterface実装名を確認。
  - `BPI_EnemyAI`: `GetPatrolRoute`, `SetMovementSpeed`, `GetIdealRange`, `AttackStart`
  - `BPI_Damagable`: `IsDead`, `TakeDamage`, `Heal`, `GetMaxHealth`, `GetCurrentHealth`, `IsAttacking`, `ReserveAttackToken`, `GetTeamNumber`, `GetMaxGage`, `GetCurrentGage`
- Behavior Tree の参照関係を確認。
  - Melee: 距離条件、武器装備、理想距離、近接攻撃、ストレイフ。
  - Ranged: 射線、カバー、理想距離、射撃、回復。
  - Mage: 射線、理想距離、テレポート、魔法攻撃、回復。
  - Boss: 複数攻撃、テレポート、浮遊、無敵/割り込み制御。
- EQS のGenerator/Test系参照を確認。
  - `EQS_FindCover`: SimpleGrid + Distance / Pathfinding / Trace。
  - `EQS_FindIdealRangedLocation`: SimpleGrid + DistanceTo / Distance / Pathfinding / Trace。
  - `EQS_Strafe`: OnCircle + Distance / Pathfinding。
  - `EQS_Teleport`: Cone + Distance / Pathfinding / Trace。
  - `EQS_Seeking`: Cone + Pathfinding / Trace。
  - `EQS_FurthestPOIActor`: ActorsOfClass + Distance。
- `Spawner_BP` とマップ配置を確認。
  - `SpawnAIFromClass`
  - `Wave1`, `wave2`, `Wave3`
  - `Boss1`, `Boss2`, `Boss3`
  - `Melee`, `Ranged`, `Mage`
  - `Stage1/2/3` と `Boss1/2/3` のExternalActorsに `Spawner_BP` 等の参照。
- `README.md` をポートフォリオ向けに上書き。

## 注意

- Behavior Tree のRoot直下順、DecoratorのObserver Abort、Service Tick間隔、Taskの全パラメータは、最終的にはエディタ画面のスクリーンショットで補強した方がよい。
- 現在のGit状態には、以前から存在したと思われるuasset/umap変更も残っている。今回の主なテキスト成果物は `Config/`, `README.md`, `Docs/`。
