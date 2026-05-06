# AI Portfolio Analysis

## 結論

このプロジェクトには、敵AI専用の実装群がかなりまとまって存在する。中核は `AIC_Enemy_Base`, `BB_Enemy_Base`, 各敵タイプ別のBehavior Tree、`BP_Enemy_Base`、`Spawner_BP`、複数のEQS。

ポートフォリオでは、次のように書くのが一番安全で強い。

> 敵AIを主に担当し、Behavior Tree / Blackboard / AI Controller / AI Perception / EQS を用いて、近接・遠距離・魔法・Bossの敵タイプごとに検知、追跡、探索、攻撃、回復、テレポート、ウェーブ生成を制御しました。

「AIシステムを全部完全自作」と抽象的に盛るより、アセット名と処理単位を出して説明した方が現場エンジニアには強く見える。

一部機能をUE5の解説記事やYouTube動画を参考に実装した場合は、隠さずに次のように言うのが安全。

> UE5のAI機能は解説記事や動画も参考にしながら学び、Behavior Tree / Blackboard / AI Perception / EQS の基本構成を本作の敵タイプに合わせて組み込み・調整しました。参考実装をそのまま置くのではなく、近接、遠距離、魔法、Bossごとの攻撃距離、探索、回復、テレポートに合わせて分岐とパラメータを作りました。

## 1. AI関連アセット

| 種別 | アセット | 確認できた役割 |
| --- | --- | --- |
| AI Controller | `AIC_Enemy_Base` | Perceptionでプレイヤーを検知し、Blackboardの `AttackTarget` / `State` / `PointOfInterest` などを更新する中核 |
| AI Controller | `AIC_Enemy_Boss` | Boss専用。Boss BP、Boss HP UI と連携 |
| Blackboard | `BB_Enemy_Base` | ターゲット、状態、探索地点、攻撃距離、防御距離、ターゲット距離を共有 |
| BT | `BT_Enemy_Base` | 共通敵AI。待機、調査、凍結など |
| BT | `BT_Enemy_Melee` | 近接敵。接近、理想距離、武器装備、近接攻撃、ストレイフ |
| BT | `BT_Enemy_Ranged` | 遠距離敵。射線確保、カバー、理想距離、射撃、回復 |
| BT | `BT_Enemy_Mage` | 魔法敵。距離維持、テレポート、魔法攻撃、回復 |
| BT | `BT_Enemy_Boss` | Boss。近接/ジャンプ/AOE/テレポート/浮遊/無敵・割り込み制御 |
| SubTree | `BT_SubTree_Passive` | 巡回または待機 |
| SubTree | `BT_SubTree_Investigating` | 最後に感知した地点へ移動して確認 |
| SubTree | `BT_SubTree_Seeking` | 見失った対象を探索 |
| SubTree | `BT_SubTree_Frozen` | 凍結・行動不能状態用 |
| Enemy BP | `BP_Enemy_Base` | HP/被ダメージ/攻撃トークン/Interface/AIController/BT の共通基底 |
| Enemy BP | `BP_Enemy_Melee` | 近接攻撃・ブロック・武器装備 |
| Enemy BP | `BP_Enemy_Ranged` | 銃撃・距離管理 |
| Enemy BP | `BP_Enemy_Mage` | 魔法攻撃・テレポート・回復 |
| Enemy BP | `BP_Enemy_Boss` | Boss攻撃・召喚・HP UI・AOE |
| Spawner | `Spawner_BP` | Waveごとの敵生成。Melee/Ranged/Mage とBTを参照 |
| EQS | `EQS_FindCover` | 遠距離敵などのカバー地点探索 |
| EQS | `EQS_FindIdealRangedLocation` | 射線や距離を考慮した遠距離用位置取り |
| EQS | `EQS_Strafe` | 近接敵の回り込み/横移動候補 |
| EQS | `EQS_Teleport` | 魔法敵/Boss系のテレポート候補 |
| EQS | `EQS_Seeking` | 見失ったターゲットの探索候補 |
| EQS | `EQS_FurthestPOIActor` | Boss用のPOI選択。名前上は遠いPOIを選ぶ用途 |

## 2. AIの判断条件

### 確定情報

- `AIC_Enemy_Base` は `Sight`, `Hearing`, `Damage` の Perception を使う。
- `OnPerceptionUpdated` から感知情報を処理する構成。
- 感知種類ごとに `HandleSensedSight`, `HandleSensedSound`, `HandleSensedDamage` がある。
- 見失い処理として `HandleLostSight`, `SeekAttackTarget` がある。
- `TimeToSeekAfterLosingSight = 3.0`。
- Blackboardの `State` は `E_AIState` enum。
- `E_AIState` には `Passive`, `Attacking`, `Frozen`, `Investigating`, `Seeking` がある。
- `AttackTarget` は攻撃対象。
- `PointOfInterest` は調査・探索地点。
- `DistanceToAttackTarget` は `BTS_UpdateDistanceToTarget` が更新する。
- `BTS_StopAttackingIfTargetIsDead` がターゲット死亡時の攻撃停止を担う。
- Melee BT には `DistanceToAttackTarget <= 200`, `>= 350` の条件がある。
- Mage BT には `DistanceToAttackTarget <= 200`, `ShouldTeleport`, `TeleportLocation` がある。
- Boss BT には `DistanceToAttackTarget >= 250`, `<= 300` の条件がある。

### 推測

- 通常時は `Passive` で待機/巡回する。
- 視覚・聴覚・被ダメージで `Attacking` へ遷移する。
- 見失った直後は `Investigating` または `Seeking` へ移り、最後の感知地点やEQS候補を調べる。
- 一定時間見つからなければ `Passive` に戻る。
- `Frozen` は状態異常による行動停止分岐。

## 3. プレイヤー目線の敵挙動

プレイヤーが敵の視界に入る、音で気づかれる、または敵へ攻撃すると、敵がプレイヤーをターゲットとして認識する。敵はプレイヤーの位置と距離をBlackboardに保持し、Behavior Treeで攻撃・移動・探索を切り替える。

近接敵は距離を詰め、攻撃範囲に入ると武器を構えて攻撃する。距離が離れすぎると再接近し、近すぎる/攻撃後の位置取りでは `EQS_Strafe` による横移動候補を使う構成がある。

遠距離敵はプレイヤーに射線が通る位置を探しつつ、必要に応じてカバー地点や理想距離の地点へ移動する。低HP時には回復を選ぶ条件がある。

魔法敵は遠距離攻撃に加えて、回復とテレポートを持つ。近づかれた場合や条件を満たした場合、`EQS_Teleport` で候補位置を選び、距離を取り直す。

Bossは通常敵より行動パターンが多い。近接攻撃、ジャンプ攻撃、AOE、テレポート、浮遊、無敵/割り込み制御を持ち、Boss専用AI ControllerとHP UIに連携している。

## 4. 実装上の工夫として書ける点

- 共通の判断情報を `BB_Enemy_Base` に集約し、敵ごとの差分は敵タイプ別BTに分けている。
- `AIC_Enemy_Base` で視覚・聴覚・被ダメージをまとめて扱い、戦闘開始の入口を統一している。
- `Passive`, `Investigating`, `Seeking`, `Frozen` をSubTree化し、複数の敵タイプで再利用している。
- 距離は `BTS_UpdateDistanceToTarget` で更新し、Decorator条件で接近・攻撃・位置取りを切り替えている。
- `BPI_EnemyAI` を通じて、BT Taskから敵BP側の `AttackStart`, `SetMovementSpeed`, `GetIdealRange`, `GetPatrolRoute` を呼ぶ構成になっている。
- `BPI_Damagable` を通じて、HP、被ダメージ、回復、死亡判定、攻撃トークンを扱える。
- `ReserveAttackToken` / `ReturnAttackToken` により、複数敵が同時に攻撃しすぎないようにする意図が見える。
- `EQS` を用途別に分け、敵の位置取りを手書き座標ではなく環境クエリで選ぶ構成にしている。
- 攻撃やテレポートの完了をイベント/Delegateで待つ構成があり、アニメーション・攻撃処理・AI判断を分離している。
- `Spawner_BP` でウェーブと敵種別を管理し、ステージ進行とBoss戦へつなげている。

## 5. README / ES 用説明文

### 200字程度

> 敵AIを主に担当し、Unreal Engine の Behavior Tree / Blackboard / AI Perception / EQS を用いて、近接・遠距離・魔法・Bossの行動を実装しました。視覚・聴覚・被ダメージでプレイヤーを検知し、距離や視線、HP状態に応じて追跡、探索、攻撃、回復、テレポートを切り替える構成にしました。

### 400字程度

> 本作では敵AIを主に担当しました。共通AI ControllerでAI Perceptionの視覚・聴覚・被ダメージを処理し、Blackboardに攻撃対象、状態、探索地点、ターゲット距離を保存します。Behavior Treeはその値を参照し、待機/巡回、調査、探索、攻撃、凍結などの状態を切り替えます。敵タイプは近接・遠距離・魔法・BossでBTを分け、近接は接近とストレイフ、遠距離は射線確保とカバー、魔法は回復とテレポート、Bossは複数攻撃や無敵/割り込み制御を持たせました。EQSを使って位置取り候補を選ぶことで、敵ごとに異なる戦い方を作りました。

### README向け短文

> 敵AIは Blueprint ベースで実装しました。共通AI Controllerがプレイヤーを視覚・聴覚・被ダメージで検知し、Blackboardへ状態・ターゲット・探索地点・距離を保存します。Behavior Treeはその情報をもとに、近接、遠距離、魔法、Bossごとの追跡、探索、攻撃、回復、テレポートを切り替えます。

## 6. 面接で聞かれそうな質問と回答例

### Q. AIは何を担当しましたか？

> 主に敵AIです。UE5の解説記事や動画も参考にしながら、`AIC_Enemy_Base`, `BB_Enemy_Base`, `BT_Enemy_Melee/Ranged/Mage/Boss`, `BP_Enemy_Base` 周辺を本作向けに組み込み・調整しました。AI Perceptionで検知し、Blackboardで状態を共有し、Behavior Treeで敵タイプごとの行動を切り替える構成です。

### Q. プレイヤーをどう検知していますか？

> `AIC_Enemy_Base` の `AIPerceptionComponent` で、視覚、聴覚、被ダメージを扱っています。検知すると `AttackTarget` と `State` をBlackboardへ設定し、見失った場合は `PointOfInterest` を使って調査・探索へ移る構成です。見失い後の探索時間として `TimeToSeekAfterLosingSight` が3秒に設定されています。

### Q. Behavior Treeをどう分けましたか？

> 共通状態はSubTree化し、敵タイプごとの戦闘行動は個別BTに分けました。`BT_SubTree_Passive` は待機/巡回、`Investigating` は感知地点の調査、`Seeking` は見失った対象の探索、`Frozen` は行動不能状態です。その上で、近接・遠距離・魔法・BossのBTがそれぞれ攻撃や位置取りを選びます。

### Q. EQSは何に使っていますか？

> 位置取りの候補選択に使っています。遠距離敵はカバーや射線が通る位置、近接敵はストレイフ位置、魔法敵はテレポート位置、探索時は前方範囲の探索候補、BossはPOI候補を選ぶためにEQSを分けています。

### Q. 一番工夫した点は？

> 共通化と差分管理です。検知と状態管理は `AIC_Enemy_Base` と `BB_Enemy_Base` に寄せ、敵ごとの個性は個別BTと敵BP側の攻撃処理に分けました。これにより、同じ基盤で近接、遠距離、魔法、Bossの違う挙動を作れます。

### Q. 参考にした教材や動画はありますか？

> はい、UE5のAI Perception、Behavior Tree、EQSの使い方は解説記事や動画も参考にしました。そのうえで、本作では敵タイプが近接、遠距離、魔法、Bossに分かれていたため、共通Blackboard、敵タイプ別BT、EQSの用途分け、攻撃Taskとの接続を自分たちのゲーム仕様に合わせて組み込み・調整しました。

### Q. 攻撃トークンは何ですか？

> `BP_Enemy_Base` に `ReserveAttackToken` / `ReturnAttackToken` があり、名前と参照から、複数の敵が一斉に攻撃しすぎないよう攻撃権を予約・返却する仕組みだと説明できます。ただし、面接前に実際のBlueprint画面で、トークン数と返却タイミングを確認しておくべきです。

### Q. 改善するなら？

> Blueprint中心でTaskやDecoratorが増えているため、現在State、AttackTarget、距離、EQS結果を画面上にデバッグ表示できるようにしたいです。また、敵ごとの距離や攻撃頻度はDataAsset化して、BTを開かずに調整できるようにすると保守しやすくなります。

## 7. 「自分が担当した」と書きやすい範囲

### 書いてよい

- 敵AIの実装・調整を担当。
- Behavior Tree / Blackboard / AI Controller を用いた敵AIを担当。
- AI Perception による視覚・聴覚・被ダメージ検知を組み込んだ。
- 近接・遠距離・魔法・Bossの敵タイプ別Behavior Treeを構成した。
- EQSを用いたカバー、理想距離、ストレイフ、テレポート、探索地点の選択を組み込んだ。
- ウェーブスポナーと敵AIの連携を担当。
- UE5の解説記事や動画を参考にしながら、本作の仕様に合わせて敵AIを組み込み・調整した。

### 面接前に画面確認してから書くとよい

- `AIC_Enemy_Base` の各Event Graphを自分が組んだ。
- `BTD_*`, `BTT_*`, `BTS_*` を自作した。
- `ReserveAttackToken` の仕様を設計した。
- EQSの重みやFilterを自分で設計した。
- SpawnerのWave配列を自分で組んだ。

### 危険

- UEのAI PerceptionやEQSそのものを自作した。
- すべてのAIアセットを完全にゼロから作った。
- 参考資料なしで全機能を独力で設計したように見せる。
- 高度な戦術AIや学習AIを実装した。
- Root順やObserver Abortなどの詳細設定を、確認なしで数値込みで断言する。

## 8. 追加で確認すべきBlueprint画面

- `AIC_Enemy_Base` Event Graph
  - `OnPerceptionUpdated`
  - `HandleSensedSight`
  - `HandleSensedSound`
  - `HandleSensedDamage`
  - `HandleLostSight`
  - `SeekAttackTarget`

- `BB_Enemy_Base`
  - key型
  - `State` のEnum型
  - `E_AIState` の表示名と順番

- `BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss`
  - Root直下の分岐順
  - Decorator条件
  - Observer Abort
  - Service Tick間隔
  - Taskパラメータ

- `BP_Enemy_Base`
  - `BPI_EnemyAI`
  - `BPI_Damagable`
  - `ReserveAttackToken`
  - `ReturnAttackToken`
  - `StoreAttackTokens`
  - `GetIdealRange`

- `Spawner_BP`
  - `Wave1`, `wave2`, `Wave3`
  - `EnemyType`
  - 生成数
  - 生成先
  - Boss map遷移

- `EQS_*`
  - `EQS_FindCover`
  - `EQS_FindIdealRangedLocation`
  - `EQS_Strafe`
  - `EQS_Teleport`
  - `EQS_Seeking`
  - `EQS_FurthestPOIActor`
