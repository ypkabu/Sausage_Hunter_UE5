# AI Understanding Guide

このファイルは、自分で面接で説明できるようにするための噛み砕きメモ。

## まず一言で説明するなら

> 敵AIは、AI Controllerがプレイヤーを感知してBlackboardに情報を書き、Behavior Treeがその情報を見て、敵タイプごとに移動・探索・攻撃を選ぶ仕組みです。

これが全体像。

## 役割分担

| アセット | たとえるなら | 役割 |
| --- | --- | --- |
| `AIC_Enemy_Base` | 敵の頭脳の入口 | プレイヤーを見た/聞いた/攻撃された、を検知する |
| `BB_Enemy_Base` | 敵のメモ帳 | ターゲット、状態、距離、探索地点を保存する |
| `BT_Enemy_*` | 行動表 | 今の状態なら何をするか決める |
| `BTT_*` | 具体的な命令 | 攻撃する、移動する、武器を構える、回復する |
| `BTD_*` | 条件チェック | ターゲットが見えるか、距離内か、HPが低いか |
| `BTS_*` | 定期更新 | ターゲット距離更新、死亡チェックなど |
| `EQS_*` | 候補地点探し | カバー、テレポート先、ストレイフ先、探索地点を探す |
| `BP_Enemy_Base` | 敵キャラ本体 | HP、攻撃、被ダメージ、トークン、アニメ完了イベントを持つ |
| `Spawner_BP` | 敵の出現管理 | Waveごとに敵を生成してステージ進行を作る |

## 敵AIの流れ

1. 敵が通常状態で待機または巡回する。
2. `AIC_Enemy_Base` がプレイヤーを視覚・聴覚・被ダメージで検知する。
3. `AttackTarget` にプレイヤー、`State` に攻撃状態、`PointOfInterest` に感知位置などを書き込む。
4. Behavior Tree が `State` を見て、攻撃系分岐へ入る。
5. Service が `DistanceToAttackTarget` を更新する。
6. Decorator が距離、視線、HPなどをチェックする。
7. Task が攻撃、移動、回復、テレポートなどを実行する。
8. 見失った場合は最後の感知地点やEQS候補を調べる。
9. 見つからなければ待機/巡回に戻る。

## 敵タイプごとの説明

### 近接敵

近接敵はプレイヤーに近づいて攻撃する敵。`BT_Enemy_Melee` では、距離が遠ければ理想距離まで移動し、攻撃範囲に入ったら `BTT_MeleeAttack` を実行する。`EQS_Strafe` があるので、ただ直線で詰めるだけでなく横移動候補も持っている。

面接での言い方:

> 近接敵は距離条件を見て、接近、武器装備、攻撃、ストレイフを切り替えます。距離はBlackboardの `DistanceToAttackTarget` に保持して、Serviceで更新しています。

### 遠距離敵

遠距離敵はプレイヤーと距離を取り、射線が通る位置から攻撃する敵。`BT_Enemy_Ranged` は `EQS_FindCover` と `EQS_FindIdealRangedLocation` を参照している。HPが低いと回復する条件もある。

面接での言い方:

> 遠距離敵はプレイヤーへ近づくだけでなく、射線やカバーを考慮して位置を選びます。EQSで候補地点を出し、BT側で射撃や回復を切り替えています。

### 魔法敵

魔法敵は遠距離攻撃に加えて回復とテレポートを持つ敵。`BT_Enemy_Mage` は `EQS_Teleport` を使い、`ShouldTeleport` と `TeleportLocation` の文字列がある。近づかれた時や条件を満たした時に距離を取り直す構成。

面接での言い方:

> 魔法敵は攻撃だけでなく、回復とテレポートで戦闘距離を作り直します。テレポート先はEQSで選ぶため、固定座標ではなく状況に応じた候補を使えます。

### Boss

Bossは通常敵と別のController/BTを持ち、攻撃パターンが多い。`Quick Attack`, `Jump Attack`, `AOE Trail`, `Teleport`, `StartFloat`, `StopFloat`, `SetIsInvincible`, `SetIsInterruptible` などがある。HP UIとも連携する。

面接での言い方:

> Bossは通常敵よりも状態制御を多く持たせています。攻撃、テレポート、浮遊、無敵、割り込み可否をBT Taskで切り替え、Boss専用UIとも連携しています。

## 攻撃トークンをどう説明するか

`BP_Enemy_Base` には `ReserveAttackToken`, `ReturnAttackToken`, `StoreAttackTokens`, `ReservedAttackTokens`, `TokensUsedInCurrentAttack`, `TokensNeeded` がある。

説明としてはこう。

> 複数の敵が同時に攻撃しすぎると理不尽になるため、攻撃権をトークンとして予約・返却する仕組みを用意しています。攻撃開始時にトークンを確保し、攻撃終了時に返却することで、同時攻撃数を制御する意図があります。

ただし、実際のトークン数や返却タイミングは `BP_Enemy_Base` の画面を見て確認してから話す。

## そのまま使える面接回答

### 1分版

> 主に敵AIを担当しました。共通のAI Controllerでプレイヤーを視覚・聴覚・被ダメージから検知し、Blackboardへターゲット、状態、探索地点、距離を書き込みます。Behavior Treeはその値を見て、待機、巡回、探索、攻撃、回復、テレポートを切り替えます。敵タイプは近接、遠距離、魔法、BossでBTを分け、共通の状態管理を使いながら敵ごとの戦い方を変えました。

### 3分版

> 敵AIは `AIC_Enemy_Base`, `BB_Enemy_Base`, 敵タイプ別Behavior Tree, `BP_Enemy_Base` を中心に作っています。AI ControllerはAI Perceptionで視覚、聴覚、被ダメージを検知し、Blackboardに `AttackTarget`, `State`, `PointOfInterest`, `DistanceToAttackTarget` などを保存します。BTは `State` を見て、Passive, Investigating, Seeking, Frozen, Attacking のような状態を切り替えます。共通行動はSubTree化し、敵タイプごとの違いは `BT_Enemy_Melee`, `BT_Enemy_Ranged`, `BT_Enemy_Mage`, `BT_Enemy_Boss` に分けました。近接は距離を詰めて攻撃、遠距離はカバーや射線、魔法は回復とテレポート、Bossは複数攻撃や無敵/割り込み制御を持ちます。EQSはカバー、理想距離、ストレイフ、テレポート、探索地点の候補選択に使っています。

## 自作と書く時の安全な言い方

おすすめ:

> 敵AIをBlueprintで実装し、Behavior Tree / Blackboard / AI Perception / EQS を用いて敵タイプごとの行動を制御しました。

参考資料を使ったことも含めて正直に言う場合:

> UE5のAI機能は解説記事や動画も参考にしながら学び、本作の敵仕様に合わせてBehavior Tree / Blackboard / AI Perception / EQS を組み込み・調整しました。

さらに具体的:

> `AIC_Enemy_Base`, `BB_Enemy_Base`, `BT_Enemy_Melee/Ranged/Mage/Boss`, `BP_Enemy_Base` 周辺を担当し、検知、状態遷移、距離判定、攻撃、探索、ウェーブ生成との連携を実装・調整しました。

避ける:

> AIを全部完全自作しました。

理由:

現場エンジニアは「AI Perception/EQS自体を作ったのか」「BTのTaskも全部書いたのか」「教材やテンプレートとの差分は何か」を聞いてくる可能性が高い。参考にした記事や動画があるなら、そこで学んだ基本形と、自分がプロジェクトに合わせて変えた部分を分けて説明する方が信頼される。
