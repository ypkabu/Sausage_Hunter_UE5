# AI Screenshot Review

## 調査範囲

- 既存Markdown: `README.md`, `Docs/AI_CONTEXT.md`, `Docs/AI_TASKS.md`, `Docs/AI_WORKLOG.md`, `Docs/AI_PORTFOLIO_ANALYSIS.md`, `Docs/AI_UNDERSTANDING_GUIDE.md`
- 実装アセット: `Content/Enemies/**`, `Content/DamageSystem/BPI_Damagable.uasset`, `Saved/*AI*.json`
- スクリーンショット: `images/*.png`
- 俯瞰用に `Saved/ImageContactSheets/contact_sheet_01.jpg` から `contact_sheet_08.jpg` を作成。対応表は `Saved/ImageContactSheets/manifest.tsv`。

## 画像の画面種別分類

| 種別 | 主な画像 | 読み取れる内容 |
| --- | --- | --- |
| Blackboard | `スクリーンショット 2026-05-06 083742.png`, `083749.png` | `BB_Enemy_Base` のキー一覧。`SelfActor`, `AttackTarget`, `State`, `PointOfInterest`, `AttackRadius`, `DefendRadius`, `DistanceToAttackTarget`。 |
| AI Controller Blueprint | `083415.png`, `083430.png`, `083445.png`, `083452.png`, `083511.png`, `083516.png`, `083523.png`, `083534.png` | `AIC_Enemy_Base` の Possess, Perception, Sight/Hearing/Damage, 忘却/探索、状態設定処理。 |
| Behavior Tree 全体図 | `081843.png`, `081849.png`, `081947.png`, `082008.png`, `082019.png`, `082041.png`, `082053.png` | 敵タイプ別BTの状態分岐、攻撃、回復、移動、EQS呼び出し。 |
| Behavior Tree SubTree | `082102.png`, `082107.png`, `082116.png`, `082131.png` | `Frozen`, `Investigating`, `Passive`, `Seeking` の共通行動。 |
| EQS | `スクリーンショット (23).png`, `(24).png`, `(25).png`, `(29).png` など | カバー、理想遠距離、テレポート、AttackTarget Context のGenerator/Test。 |
| Enemy Base / Damage / Token | `081354.png`, `081419.png`, `084112.png` | HP UI、死亡処理、被弾時の凍結/反撃遷移、攻撃トークン返却。 |
| Spawner / Wave | `081403.png` など | 敵死亡後のスポナー通知、ランダム生成、敵種別選択らしき処理。 |
| Details / Class Defaults | `スクリーンショット (30).png` から `(99).png` 付近 | BT Task / Decorator / EQS / BP変数の詳細設定。文字は一部読みにくいため、数値断定にはエディタ再確認が必要。 |
| 参考外または補助 | `スクリーンショット (1).png`, `(2).png`, `081724.png` | UE画面と外部モニタ、Content Browser等。実装根拠としては弱い。 |

## スクショから追加で確定できたこと

- `AIC_Enemy_Base` は `Detour Crowd AIController` 系のBlueprint。
- `On Possess` で `BP_Enemy_Base` にCastし、敵BPが持つ `BehaviorTree` を `Run Behavior Tree` に渡す。
- `On Possess` 後に `Set State as Passive` を呼び、`Get Ideal Range` で `AttackRadius` と `DefendRadius` を取得してBlackboardへ書く。
- `On Possess` で `CheckForgottenSeenActor` を0.5秒ループTimerとして開始している。
- `On Perception Updated` では、更新Actorごとに Sight / Hearing / Damage を `Can Sense Actor` で確認し、Sight成功なら `HandleSensedSight`、Sight失敗なら `HandleLostSight`、Hearingなら刺激Locationを `HandleSensedSound`、Damageなら `HandleSensedDamage` へ渡す。
- `HandleSensedSight` は `KnownSeenActors` に追加し、`On Same Team` 判定と `E_AIState` の現在状態を見てから攻撃状態へ遷移する構成。
- `On Same Team` は、自分のControlled Pawnと相手Actorの `Get Team Number` を比較している。
- 忘却/探索処理では、`KnownSeenActors` と `AI Perception` の現在視認Actorを比較し、見えなくなったActorに `HandleForgotActor` を呼ぶ構成が見える。
- `SeekAttackTarget` は `AttackTarget` の現在Locationを使って `Set State as Seeking` へつなぐ。
- `Set State as Passive` はBlackboardの `State` keyへ `Passive` enumを書いている。ほかの状態設定関数も同型と推測できる。
- `BT`上のBlackboard条件は複数箇所で `aborts both` が見える。状態変化時に現在実行中の枝を中断する意図がある。
- `BTS_StopAttackingIfTargetIsDead` はスクショ上でtick 1.00s、`BTS_UpdateDistanceToTarget` はtick 0.10s と読める。

## AI判断条件

### 確定寄り

- 状態分岐: `State == Frozen / Attacking / Seeking / Investigating / Passive`。
- 検知入口: `Sight`, `Hearing`, `Damage`。
- 味方判定: `Get Team Number` 比較。
- 見失い/忘却: `KnownSeenActors` と現在のSight感知リストの差分。
- 距離判断: `DistanceToAttackTarget` をBT Serviceで更新。
- 近接BT: `DistanceToAttackTarget <= 200`, `>= 350` の条件がある。
- 魔法BT: `DistanceToAttackTarget <= 200` で近すぎる場合のGroundSmash系分岐がある。
- 遠距離BT: `BTD_CanSeeTarget`, `BTD_IsHealthBelowThreshold`, `HealthThreshold 0.300000` が見える。
- Boss BT: `BTD_Chance`, `DistanceToAttackTarget >= 250`, `<= 300`, 各攻撃Cooldownがある。

### 推測として扱うべき

- `Hearing` は `PointOfInterest` を感知Locationに設定して `Investigating` に入る可能性が高いが、最終ノードはスクショだけでは完全には追い切れていない。
- `Damage` はDamage Causerを `AttackTarget` にして攻撃状態へ入る可能性が高い。
- `KnownSeenActors` は見失い後すぐPassiveに戻さず、一定時間Seekingへ移すための補助配列と考えられる。
- 攻撃トークンは同時攻撃数制御の意図が強いが、正確なトークン配布数と返却漏れ対策は追加確認が必要。

## 敵タイプ別の役割

| アセット | 役割 |
| --- | --- |
| `AIC_Enemy_Base` | 通常敵共通の検知/状態管理。BT起動、Blackboard初期化、Perception処理、見失い処理。 |
| `AIC_Enemy_Boss` | Boss専用Controller。通常敵と同じkey名を持ち、Boss BP/UI連携がある。 |
| `BB_Enemy_Base` | 敵AIの共有メモ。ターゲット、状態、探索地点、攻撃/防御距離、現在距離を保持。 |
| `BT_Enemy_Melee` | 近接敵。装備、接近、近接攻撃、距離条件、ストレイフ。 |
| `BT_Enemy_Ranged` | 遠距離敵。射線確保、カバー探索、回避移動、射撃、低HP時回復。 |
| `BT_Enemy_Mage` | 魔法敵。射線/距離管理、魔法攻撃、回復、テレポート、近距離時のGroundSmash。 |
| `BT_Enemy_Boss` | Boss。Quick/Jump/AOE/Throw系攻撃、テレポート、浮遊、無敵/割り込み制御。 |
| `BT_SubTree_Passive` | Patrol Routeがあれば巡回、なければIdleで待機。 |
| `BT_SubTree_Investigating` | `PointOfInterest` へJogging移動し、3秒待機後Passiveへ戻す。 |
| `BT_SubTree_Seeking` | 見失った対象をEQSで探す用途と推測。 |
| `BT_SubTree_Frozen` | Focus解除、移動速度Idle、5秒待機。凍結/行動不能用。 |
| `BP_Enemy_Base` | HP UI、死亡、被ダメージ、状態異常、攻撃開始/終了、攻撃トークンなどの共通本体。 |
| `Spawner_BP` | Wave/敵種別/死亡通知/生成を扱う。AI単体ではなく進行制御寄り。 |

## プレイヤー目線の挙動

プレイヤーが敵の視界に入る、音で気づかれる、または攻撃すると、敵はプレイヤーをターゲットとして認識する。見えている間は攻撃状態に入り、見失った場合はすぐに完全停止せず、最後に見た位置やEQSの候補地点を調べる。

近接敵は接近して武器攻撃する。攻撃距離外では理想距離へ移動し、状況によってストレイフ位置へ動く。遠距離敵はプレイヤーへの射線を確保し、カバーや理想距離へ移動しながら射撃する。魔法敵は遠距離攻撃に加え、近づかれたときのGroundSmash、回復、テレポートで距離を作り直す。Bossは通常敵より攻撃パターンが多く、テレポート、浮遊、AOE、無敵/割り込み制御を使う。

## 実装上の工夫として書ける点

- `AIC_Enemy_Base` に検知入口を集約し、敵BP側の `BehaviorTree` を起動することで、共通Controller + 敵別BTの構成にしている。
- Blackboard keyを共通化し、敵タイプごとの違いはBTと敵BP側の攻撃実装へ分離している。
- `Passive`, `Investigating`, `Seeking`, `Frozen` をSubTree化し、近接/遠距離/魔法で再利用している。
- `BPI_EnemyAI` で `GetIdealRange`, `GetPatrolRoute`, `SetMovementSpeed`, `AttackStart` を呼び、BT Taskと敵本体の責務を分けている。
- `BPI_Damagable` でHP、死亡、被ダメージ、チーム番号、攻撃トークンを統一的に扱っている。
- `On Same Team` により味方を攻撃対象から除外する設計が見える。
- `KnownSeenActors` とTimerで、見失い処理をPerceptionイベントだけに依存させない構成が見える。
- `EQS_FindCover`, `EQS_FindIdealRangedLocation`, `EQS_Strafe`, `EQS_Teleport`, `EQS_Seeking` により、固定座標ではなく環境に応じた位置候補を選ぶ構成にしている。
- 攻撃/テレポート/浮遊/無敵/割り込みをBT Taskに分け、攻撃演出や敵本体処理とAI判断を分離している。

## 面接で突っ込まれそうな点

| 質問 | 答え方 |
| --- | --- |
| AI Perception自体を作ったのか？ | UE標準のAI Perceptionを利用し、Sight/Hearing/Damageのイベント処理とBlackboard更新をBlueprintで組みました。 |
| なぜBTを敵ごとに分けた？ | 検知や状態管理は共通化しつつ、攻撃距離や攻撃種類、EQSが敵タイプごとに違うため、戦闘行動はBTを分けました。 |
| EQSは何を評価している？ | カバー/理想遠距離/テレポート/ストレイフ/探索で、PathExist, Trace, Distanceを使っています。重みの詳細は画面で確認して説明します。 |
| 攻撃トークンは本当に機能している？ | 名前と接続から同時攻撃数制御の意図があります。ただし、最終的なトークン数や返却漏れ対策は面接前に再確認します。 |
| `BTT_RangedAttack` のTarget KeyがSelfActorに見えるが？ | スクショ上はそう見える箇所があります。射撃先をTask内部で別取得しているのか、設定ミスなのか追加確認が必要です。 |
| `Frozen` ノードが `Investigating State` と表示されている箇所は？ | ノード名の付け忘れ/複製名の可能性があります。条件は `State == Frozen` で `BT_SubTree_Frozen` を呼んでいるため、説明時は条件とSubTree名を優先します。 |
| C++は使った？ | このプロジェクトのAIはBlueprint中心です。UEのAI機能をBlueprintで組み合わせ、調整しました。 |

## README/ESに使える表現

短め:

> 敵AIをBlueprintで実装・調整しました。UE5の解説記事や動画も参考にしながら、共通AI Controller、Blackboard、Behavior Tree、EQSを本作の敵仕様に合わせて組み込みました。視覚・聴覚・被ダメージでプレイヤーを検知し、近接・遠距離・魔法・Bossごとの追跡、探索、攻撃、回復、テレポートを切り替えます。

少し具体:

> 敵AIでは、共通の `AIC_Enemy_Base` と `BB_Enemy_Base` で検知・状態管理を行い、`BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss` で敵タイプ別の戦闘行動を分けました。EQSを用いてカバー、理想距離、ストレイフ、テレポート、探索地点を選ぶことで、単純な追跡だけでなく敵ごとの位置取りを持たせました。

参考資料に触れる場合:

> UE5のAI機能は解説記事やYouTube動画を参考に学びました。そのうえで、本作では近接、遠距離、魔法、Bossの敵タイプに合わせて、Blackboard key、Behavior Treeの分岐、EQSの用途、攻撃Taskとの接続を調整しました。

## 担当したと書きやすい範囲

- 敵AIの実装・調整。
- `AIC_Enemy_Base`, `BB_Enemy_Base`, 敵タイプ別Behavior Tree周辺。
- AI PerceptionによるSight/Hearing/Damage検知と状態遷移。
- Blackboard key設計と距離/状態/ターゲット管理。
- EQSを用いた位置取り候補の利用。
- `BP_Enemy_Base` のAI連携、被ダメージ、攻撃トークンまわり。ただし、自分の記憶と照合してから強く言う。
- SpawnerとのAI連携。Wave設計全体を担当したかは追加確認後にする。
- UE5の参考記事・動画をもとに、プロジェクト用に組み込み・調整した範囲。

## 書くと危険な範囲

- UEのAI Perception/EQS/Behavior Tree自体を自作した、という言い方。
- AIをすべて一人で完全自作した、という言い方。
- 参考資料を使っていないように見せる言い方。
- すべての `BTT_*`, `BTD_*`, `BTS_*` を自分が作った、と断言すること。
- EQSの重み、BTのObserver Abort、Cooldown、Token数を未確認の数値込みで説明すること。
- Boss AIやSpawner全体を担当したと断言すること。関与範囲が曖昧なら「AI連携を含む周辺実装」と言う。

## 追加確認したい画面

- `BTT_RangedAttack` のDetailsと内部Graph。Target Keyが `SelfActor` に見える理由。
- `ReserveAttackToken`, `ReturnAttackToken`, `StoreAttackTokens` の初期値、返却タイミング、失敗時処理。
- `E_AIState` の6番目の表示名。`Dead` 相当かどうか。
- `BT_Enemy_Boss` の全体図。Quick/Jump/AOE/Throwの優先順とCooldown。
- `EQS_*` のTest重み/Filter/Score設定。
- `Spawner_BP` のWave配列、敵生成数、難易度係数、Boss map遷移条件。
- どの `BTT_*`, `BTD_*`, `BTS_*` を自分が作ったかの記憶確認。
