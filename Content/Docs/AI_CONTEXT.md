# AI Context

## 現状

- Unreal Engine 5.4 / Blueprint 中心の3Dアクションゲーム。
- プロジェクト名は `Soseji_Hunter`。
- `Config/DefaultEngine.ini` の `[URL] GameName` は `Soseji_Hunter` に修正済み。
- `Config/DefaultGame.ini` の `ProjectName` は `Soseji_Hunter` に修正済み。
- `DefaultGame.ini` の `StartupActions` は `bAddPacks=False` に修正済み。エディタ起動時に StarterContent を再追加しないため。

## 今回の確認方法

- `Soseji_Hunter.uproject` を Unreal Editor 5.4 でロード。
- Unreal Editor Python / Commandlet で Blueprint / Behavior Tree / Blackboard / EQS / Enum / CDO をロードして確認。
- `BB_Enemy_Base` の key 型、敵BPの `AIControllerClass` / `BehaviorTree`、主要アセットの参照文字列を取得。
- Behavior Tree の Root直下順や Blueprint Event Graph のピン接続は、UE Python からは protected で完全取得できない部分がある。そのため、確定できる情報は「ロード済みUObject/CDO情報」と「uasset内の参照文字列」に分けて扱う。
- Headless / nullrhi で `open_editor_for_assets` を使うとクラッシュするため、ビジュアルなエディタ画面のスクリーンショット取得は未実施。アセット自体はUE経由でロード済み。

## AI関連の中核アセット

### AI Controller

- `/Game/Enemies/AI/AIC_Enemy_Base`
  - 通常敵共通のAI Controller。
  - `AIPerceptionComponent` を持つ。
  - `AISense_Sight`, `AISense_Hearing`, `AISense_Damage` の参照を確認。
  - `OnPerceptionUpdated`, `HandleSensedSight`, `HandleSensedSound`, `HandleSensedDamage`, `HandleLostSight`, `SeekAttackTarget` を確認。
  - Blackboard key name:
    - `AttackTarget`
    - `AttackRadius`
    - `DefendRadius`
    - `PointOfInterest`
    - `State`
  - `TimeToSeekAfterLosingSight = 3.0` を確認。

- `/Game/Enemies/AI/AIC_Enemy_Boss`
  - Boss用 Controller。
  - `BP_Enemy_Boss`, `W_BossHealthBar`, `W_PlayerHUD` 参照あり。

### Blackboard

- `/Game/Enemies/AI/BehaviorTrees/BB_Enemy_Base`
  - `SelfActor`: Object / Actor
  - `AttackTarget`: Object / Actor
  - `State`: Enum / `/Game/Enemies/AI/Enums/E_AIState`
  - `PointOfInterest`: Vector
  - `AttackRadius`: Float
  - `DefendRadius`: Float
  - `DistanceToAttackTarget`: Float

### Enum

- `E_AIState`
  - `Passive`
  - `Attacking`
  - `Frozen`
  - `Investigating`
  - `Seeking`
  - もう1要素は表示名を文字列抽出だけでは確定しきれていないが、AIC側に `SetStateAsDead` があるため `Dead` 相当の状態がある可能性が高い。

- `E_AISense`
  - `Sight`
  - `Hearing`
  - `Damage`

- `E_EnemyType`
  - `Melee`
  - `Ranged`
  - `Mage`

### Behavior Tree

- `BT_Enemy_Base`
  - `BB_Enemy_Base` を使用。
  - 共通の状態分岐用。
  - `BT_SubTree_Passive`, `BT_SubTree_Investigating`, `BT_SubTree_Frozen` 参照あり。

- `BT_Enemy_Melee`
  - 近接敵用。
  - `BT_SubTree_Passive`, `BT_SubTree_Investigating`, `BT_SubTree_Seeking`, `BT_SubTree_Frozen` 参照あり。
  - `BTS_UpdateDistanceToTarget`, `BTS_StopAttackingIfTargetIsDead` 参照あり。
  - `BTT_EquipWeapon`, `BTT_MoveToIdealRange`, `BTT_MeleeAttack`, `BTT_Focus`, `BTT_ClearFocus`, `BTT_UnequipWeapon`, `BTT_SetMovementSpeed` 参照あり。
  - `BTD_IsWithinIdealRange`, `BTD_IsWieldingWeapon` 参照あり。
  - `DistanceToAttackTarget <= 200`, `DistanceToAttackTarget >= 350` の条件文字列を確認。
  - `EQS_Strafe` 参照あり。

- `BT_Enemy_Ranged`
  - 遠距離敵用。
  - `BT_SubTree_Passive`, `BT_SubTree_Investigating`, `BT_SubTree_Seeking`, `BT_SubTree_Frozen` 参照あり。
  - `BTT_RangedAttack`, `BTT_Heal`, `BTT_Focus`, `BTT_ClearFocus`, `BTT_SetMovementSpeed` 参照あり。
  - `BTD_CanSeeTarget`, `BTD_IsHealthBelowThreshold` 参照あり。
  - `EQS_FindCover`, `EQS_FindIdealRangedLocation` 参照あり。
  - `Move to Line of Sight`, `FindCover`, `HealthThreshold`, `HealPercentage` の文字列を確認。

- `BT_Enemy_Mage`
  - 魔法敵用。
  - `BT_SubTree_Passive`, `BT_SubTree_Investigating`, `BT_SubTree_Seeking`, `BT_SubTree_Frozen` 参照あり。
  - `BTT_Mage_Attack`, `BTT_Mage_Heal`, `BTT_Mage_Teleport`, `BTT_Focus`, `BTT_ClearFocus`, `BTT_SetMovementSpeed` 参照あり。
  - `BTD_CanSeeTarget`, `BTD_IsHealthBelowThreshold` 参照あり。
  - `EQS_FindIdealRangedLocation`, `EQS_Teleport` 参照あり。
  - `BasicAttack`, `BarrageAttack`, `Heal`, `ShouldTeleport`, `TeleportLocation` 文字列を確認。
  - `DistanceToAttackTarget <= 200` の条件文字列を確認。

- `BT_Enemy_Boss`
  - Boss用。
  - `BT_SubTree_Passive`, `BT_SubTree_Frozen` 参照あり。
  - `BTT_Boss_Attack`, `BTT_Boss_Teleport`, `BTT_Boss_StartFloat`, `BTT_Boss_StopFloat`, `BTT_EquipWeapon`, `BTT_Focus`, `BTT_ClearFocus`, `BTT_SetIsInvincible`, `BTT_SetIsInterruptible`, `BTT_SetMovementSpeed` 参照あり。
  - `BTD_Chance`, `BTD_IsWieldingWeapon` 参照あり。
  - `EQS_FurthestPOIActor` 参照あり。
  - `Quick Attack`, `Jump Attack`, `AOE Trail`, `DistanceToAttackTarget >= 250`, `DistanceToAttackTarget <= 300` 文字列を確認。

### SubTrees

- `BT_SubTree_Passive`
  - `BTD_HasPatrolRoute`, `BTT_MoveAlongPatrolRoute`, `BTT_ClearFocus`, `BTT_SetMovementSpeed`, `Wait` 参照あり。
  - 待機・巡回用。

- `BT_SubTree_Investigating`
  - `BTT_SetMovementSpeed`, `BTT_SetStateAsPassive`, `MoveTo`, `Wait`, `PointOfInterest` 参照あり。
  - 感知地点の調査用。

- `BT_SubTree_Seeking`
  - `EQS_Seeking`, `BTT_Focus`, `BTT_SetMovementSpeed`, `BTT_SetStateAsPassive`, `MoveTo`, `Wait`, `PointOfInterest` 参照あり。
  - 見失った対象の探索用。

- `BT_SubTree_Frozen`
  - 凍結・行動不能状態用と推測。

### Enemy Character

- `BP_Enemy_Base`
  - `AIControllerClass = AIC_Enemy_Base`
  - `BehaviorTree = BT_Enemy_Base`
  - `BPI_EnemyAI`, `BPI_Damagable` を実装。
  - `BPI_EnemyAI` 側:
    - `GetPatrolRoute`
    - `SetMovementSpeed`
    - `GetIdealRange`
    - `AttackStart`
  - `BPI_Damagable` 側:
    - `IsDead`
    - `TakeDamage`
    - `Heal`
    - `GetMaxHealth`
    - `GetCurrentHealth`
    - `IsAttacking`
    - `ReserveAttackToken`
    - `GetTeamNumber`
    - `GetMaxGage`
    - `GetCurrentGage`
  - `ReserveAttackToken`, `ReturnAttackToken`, `StoreAttackTokens`, `ReservedAttackTokens`, `TokensUsedInCurrentAttack`, `TokensNeeded` を確認。
  - `TryToBlock`, `StartBlock`, `EndBlock`, `BlockChance`, `OnAttackEnd`, `OnWeaponEquipped`, `OnWeaponUnequipped`, `OnBlocked`, `OnBlockEnded`, `OnDamageResponse` を確認。

- `BP_Enemy_Melee`
  - `AIControllerClass = AIC_Enemy_Base`
  - `BehaviorTree = BT_Enemy_Melee`
  - `ShortRangeAttack`, `LongRangeAttack`, `SpinningAttack`, `GroundSmashAttack` を確認。

- `BP_Enemy_Ranged`
  - `AIControllerClass = AIC_Enemy_Base`
  - `BehaviorTree = BT_Enemy_Ranged`
  - Rifle攻撃、`AttackStart`, `AttackEnd`, `SnapToTarget`, `SetMovementSpeed` 参照あり。

- `BP_Enemy_Mage`
  - `AIControllerClass = AIC_Enemy_Base`
  - `BehaviorTree = BT_Enemy_Mage`
  - `BasicAttack`, `BarrageAttack`, `GroundSmashAttack`, `Heal`, `Teleport`, `HealOverTime` 参照あり。

- `BP_Enemy_Boss`
  - `AIControllerClass = AIC_Enemy_Boss`
  - `BehaviorTree = BT_Enemy_Boss`
  - `QuickAttack`, `JumpAttack`, `AOETrail`, `Teleport`, `SpawnEnemy`, `TakeDamage`, `UpdateHealthPercentage` 参照あり。

### Spawner / Map

- `Spawner_BP`
  - `SpawnAIFromClass` を使用。
  - `BP_Enemy_Melee`, `BP_Enemy_Ranged`, `BP_Enemy_Mage` と各BTを参照。
  - `Wave1`, `wave2`, `Wave3`, `Boss1`, `Boss2`, `Boss3`, `EnemyType`, `SpawnRateRatio`, `DeathEnemy`, `RandomPointInBoundingBox`, `GetCurrentLevelName`, `MapRangeClamped`, `RandomIntegerInRange` 文字列を確認。

- Stage / Boss maps
  - `Stage1`, `Stage2`, `Stage3` と `Boss1`, `Boss2`, `Boss3` の ExternalActors に `Spawner_BP`, `BP_PatrolRoute`, `BP_POI` 参照あり。
  - Boss maps には `BP_Enemy_Boss` と `Spawner_ref` 参照あり。

## 確定と推測の扱い

### 確定と言ってよい

- `BB_Enemy_Base` の key と型。
- 各 Enemy BP の AIController / BehaviorTree 割り当て。
- `AIC_Enemy_Base` が Sight / Hearing / Damage の Perception 系処理を持つこと。
- `State` を Blackboard enum としてBT分岐に使っていること。
- 近接・遠距離・魔法・BossでBehavior Treeが分かれていること。
- EQS がカバー、理想距離、ストレイフ、テレポート、探索、Boss用POI選択に使われていること。
- `Config` 上の古いGameName表記は現在残っていないこと。

### 推測として言うべき

- Root直下の厳密な左から右の実行順。
- Decorator の全Observer Abort設定、Service Tick間隔、Taskの細かい数値。
- 攻撃トークンが「同時攻撃数制御」として完全に機能しているか。名前と参照からその意図は強く推測できる。
- `E_AIState` の6個目の表示名。`SetStateAsDead` があるため Dead相当と推測。
