# 使用アセット・ライセンス整理

## 概要

| 項目 | 内容 |
|------|------|
| 目的 | ES・インターン提出用の外部素材出典・ライセンス整理。GitHub README への掲載も想定。 |
| 調査日時 | 2026-05-06 |
| 調査対象 | Content/ 全フォルダ、Config/、.uproject、Plugins/（存在しない）、README.md |
| 調査方法 | フォルダ名・ファイル名の確認、.uasset バイナリ内 `RelativeFilename`（インポート元パス）の文字列抽出、Config内参照検索 |

> **重要**: ライセンスは提出前に各配布元の最新規約を必ず確認してください。  
> 根拠がない項目は「要確認」としています。推測による断定はしていません。

---

## ライセンス確認サマリー

| 区分 | 件数 | 状態 |
|------|-----:|------|
| 確認済み | 8 グループ | バイナリ・フォルダ名・公式情報から出典を特定できたもの |
| 要確認 | 8 グループ | 出典またはライセンスが不明・推測にとどまるもの |
| リポジトリ公開注意 | 5 グループ | 再配布条件・AI生成開示・GitHub 掲載の要否を確認すべきもの |
| 削除/差し替え検討 | 0 | 現時点で即削除が必要なものは特定できていない |

---

## 使用アセット一覧

### 1. Unreal Engine 付属・Epic Games 提供アセット

| 種別 | パス | 推定出典 | ライセンス | 根拠 | 利用上の注意 | 状態 |
|------|------|----------|-----------|------|------------|------|
| UE テンプレート | `Content/StarterContent/` | Unreal Engine 5 Starter Content | UE EULA（Unreal Engine End User License Agreement） | UE5 新規プロジェクト作成時に追加される標準コンテンツ。フォルダ名から判断 | UE プロジェクト内での使用は認められている。アセット単体の再配布には制限がある可能性あり | 確認済み |
| UE テンプレート | `Content/Characters/Mannequins/` | UE5 ThirdPerson Template（Manny・Quinn） | UE EULA | ThirdPerson テンプレートのデフォルトキャラクター。`MM_Idle`, `MM_Run_Fwd` 等の命名規則と ABP_Manny/Quinn の存在から判断 | UE プロジェクト内での使用は認められている | 確認済み |
| UE テンプレート | `Content/Characters/Mannequin_UE4/` | UE4 Mannequin（UE5 移行コンテンツ） | UE EULA | フォルダ名 `Mannequin_UE4` と `SK_Mannequin` の存在から判断 | 同上 | 確認済み |
| UE テンプレート | `Content/LevelPrototyping/` | UE5 Level Prototyping content | UE EULA | フォルダ名と `M_PrototypeGrid`, `SM_ChamferCube` 等の標準命名から判断 | 同上 | 確認済み |
| Marketplace 無料 | `Content/ParagonFengMao/` | Paragon: Feng Mao（Epic Games 無料公開） | UE EULA（Epic Marketplace Standard License） | フォルダ名 `ParagonFengMao` から判断。Epic Games がParagonアセットを Marketplace で無料公開した事実は公知 | アセット単体の再配布禁止の可能性あり。詳細は Marketplace ページを確認 | 確認済み |
| Marketplace 無料 | `Content/ParagonGideon/` | Paragon: Gideon（Epic Games 無料公開） | UE EULA（Epic Marketplace Standard License） | フォルダ名 `ParagonGideon` から判断 | 同上 | 確認済み |
| Marketplace 無料 | `Content/ParagonSparrow/` | Paragon: Sparrow（Epic Games 無料公開） | UE EULA（Epic Marketplace Standard License） | フォルダ名 `ParagonSparrow` から判断 | 同上 | 確認済み |
| Marketplace 無料 | `Content/InfinityBladeEffects/` | Infinity Blade: Effects（Epic Games 無料公開） | UE EULA（Epic Marketplace Standard License） | フォルダ名 `InfinityBladeEffects` と `FX_Combat_Base`, `FX_Materials` 等のサブ構造から判断 | 同上。パッケージ化して配布する場合はゲームプロジェクトの一部としての配布に限定されると思われる | 確認済み |
| Lyra Starter Game | `Content/Weapons/Meshes/Rifle/SM_Rifle.uasset` | Lyra Starter Game（Epic Games サンプル） | UE EULA | バイナリ内インポートパスに `ArtSource_afarmer/depot/ArtSource/UE5/Lyra/Weapons/Rifle/SM_rifle.fbx` を確認。"afarmer" は Epic Games アーティストのローカルパス | Lyra のアセットは UE EULA 下で使用可能だが、配布条件を確認すること | 確認済み |

---

### 2. Mixamo アニメーション・キャラクター

| 種別 | パス | 推定出典 | ライセンス | 根拠 | 利用上の注意 | 状態 |
|------|------|----------|-----------|------|------------|------|
| アニメーション | `Content/Animations/Melee/sword-idle`, `sword-shield-*`, `shield-block*`, `spinning-sword-anim`, `stagger` 等 | Mixamo（Adobe） | Mixamo 利用規約（Adobe Mixamo Terms） | バイナリ内インポートパスに `mixamo_converter/OutgoingFbx/sword-idle.FBX` を確認 | ゲームプロジェクトへの組み込み・パッケージ化は許可。アセット単体の再配布は不可。Adobe アカウントが必要 | 確認済み |
| アニメーション | `Content/Animations/Axe/axe-idle_UE`, `axe-battlecry_UE`, `axe-combo-*`, `axe-run-*` 等 | Mixamo（Adobe） | Mixamo 利用規約 | バイナリ内に `mixamo_converter/OutgoingFbx/axe-idle.UE.fbx` と文字列 `mixamo.com` を確認 | 同上 | 確認済み |
| アニメーション | `Content/Animations/Rifle/rifle-idle-aim`, `rifle-run-*`, `rifle-walk-*`, `rifle-fire`, `rifle-hit` | Mixamo（Adobe） | Mixamo 利用規約 | ファイル命名規則がMixamo特有のkebab-case。Melee/Axe で Mixamo を確認しているため同一出典と判断 | 同上。ただし推定部分あり：直接バイナリを確認していない | 要確認 |
| アニメーション | `Content/Animations/Magic/mage-idle`, `magic-walk-*`, `hadouken` | Mixamo（Adobe） | Mixamo 利用規約 | 同上（kebab-case 命名規則） | 同上 | 要確認 |
| キャラクターモデル | `Content/Characters/Mage/Mesh/SKM_Mage`, `SK_Mage` | Mixamo（Adobe）「Kachujin」キャラクター | Mixamo 利用規約 | バイナリ内に `Mixamo_assets/mage-character-with-idle-anim.fbx` および `FBX.Kachujin.currentUVSet` を確認。「Kachujin」は Mixamo の無料配布キャラクター | 同上 | 確認済み |
| アニメーション | `Content/Characters/Mage/Animations/mage-cast-barrage`, `mage-ground-smash` 等 | Mixamo（Adobe） | Mixamo 利用規約 | バイナリ内に `mixamorig:Bow` 等のMixamoリグ名を確認 | 同上 | 確認済み |
| アニメーション | `Content/Enemies/hurank/Animation/axe-*_UE`, `sword-idle`, 等 | Mixamo（Adobe） | Mixamo 利用規約 | ファイル名が Animations/Axe と同一（コピー）。Axe フォルダでMixamo確認済み | 同上（同一ファイルの別フォルダコピー） | 確認済み |
| アニメーション | `Content/Enemies/pork/Animation/rifle-*`, `Content/Enemies/Sarami/Animation/mage-*` 等 | Mixamo（Adobe） | Mixamo 利用規約 | ファイル名が Animations/Rifle・Characters/Mage と同一（コピー） | 同上 | 確認済み |

> **Mixamo についての補足**: 2024年現在、Mixamo アセットは Adobe アカウント（無料プランでも可）があれば無料でダウンロードでき、ゲームへの組み込みは許可されています。ただし、アセット単体のソースファイル（.fbx 等）をリポジトリに公開して再配布することが許可されているかは要確認です。.uasset としてコミットする場合も含め、最新の Adobe/Mixamo 利用規約を参照してください。

---

### 3. 自作・オリジナルアセット

| 種別 | パス | 出典 | 根拠 | 注意点 |
|------|------|------|------|--------|
| 3Dキャラクターモデル | `Content/Pan/Mesh/pan_teisei.*` | 自作（shimadahajime 作成） | バイナリ内に `shimadahajime/Pictures/UESozai/Mesh/pan_teisei.fbx` を確認。プロジェクト作成者のローカルパス | 著作権は制作者に帰属。再利用・公開は問題なし |
| 3Dキャラクターモデル | `Content/Enemies/hurank/Mesh/furank_teisei.*` | 自作（shimadahajime 作成） | バイナリ内に `shimadahajime/Pictures/UESozai/Mesh/furank_teisei.fbx` を確認 | 同上 |
| 3Dキャラクターモデル | `Content/Enemies/Sarami/Mesh/sarami_teisei.*` | 自作（shimadahajime 作成） | バイナリ内に `shimadahajime/Pictures/UESozai/Mesh/sarami_teisei.fbx` を確認 | 同上 |
| 3Dキャラクターモデル | `Content/Enemies/pork/mesh/pork_teisei.*` | 自作（shimadahajime 作成） | バイナリ内に `shimadahajime/Pictures/UESozai/Mesh/pork_teisei.fbx` を確認 | 同上 |
| 3Dプロップ | `Content/ThirdPerson/MapAssets/` (Teppan, refrigerator, FlyngPan/huraipan) | 自作 | フォルダ名・ファイル名が日本語/ゲーム固有のもの（teppan=鉄板, reizouko=冷蔵庫）で、上記と同一インポートパターンと推定 | 要確認：直接バイナリ確認はしていない |
| アイテムモデル | `Content/Inventory/Mesh/` (tomato, masutado, onion, picrusu 等) | 自作 | 同上（日本語/食材系の固有名称） | 同上 |
| 武器モデル | `Content/Weapons/Meshes/fork/foku_model.uasset` | 自作 | 固有名称 `foku_model`（フォーク）から自作と推定 | 要確認 |
| 武器モデル | `Content/Weapons/Meshes/Knife/naihu_teisei.uasset` | 自作 | `naihu` はナイフの音読み、`teisei` は修正版の意。pan_teisei 等と同一命名パターン | 要確認 |
| Blueprint / AI | `Content/Enemies/AI/` 全体 | 自作（プロジェクト独自実装） | AIC_Enemy_Base, BB_Enemy_Base, BT_Enemy_Melee 等はプロジェクト固有名称。README に担当範囲として記載 | 著作権はプロジェクト制作者に帰属 |
| Blueprint / ゲームロジック | `Content/ThirdPerson/Blueprints/`, `Content/Player/`, `Content/Title/`, `Content/Widgets/` 等 | 自作（プロジェクト独自実装） | プロジェクト固有名称・構造から判断 | 同上 |
| マップ | `Content/ThirdPerson/Maps/Stage1〜3`, `Boss1〜3`, `Normal` | 自作 | ゲーム用マップはプロジェクト固有 | 同上 |

---

### 4. 出典が不明・要確認のアセット

#### 4-1. VFX / エフェクトパック

| パス | 推定出典 | ライセンス | 根拠 | 状態 |
|------|----------|-----------|------|------|
| `Content/FXVarietyPack/` | Marketplace / Fab のエフェクトパックと推定 | 要確認 | フォルダ名「FXVarietyPack」はMarketplaceパック名に多い命名。ただし実際のパック名・販売者・ライセンス条件は未確認 | リポジトリ公開注意 |
| `Content/StylizedKitchen/` | Marketplace / Fab の「Stylized Kitchen」系パックと推定 | 要確認 | フォルダ名・Meshes/Materials/Textures 構造がMarketplaceパックの典型。ライセンス未確認 | リポジトリ公開注意 |
| `Content/sA_SplatterofbloodFx/` | Marketplace / Fab の血しぶき VFX パックと推定 | 要確認 | フォルダ名から推定。`sA_` プレフィックスは特定クリエイターの命名規則と推定されるが確認できず | リポジトリ公開注意 |
| `Content/Effects/tool/Particles/` (Explosions, Fires, Slashes 等) | Marketplace / Fab または Tutorial アセットと推定 | 要確認 | フォルダ構造は Marketplace パックの典型。具体的なパック名・販売者・ライセンスは未確認 | リポジトリ公開注意 |
| `Content/Effects/tool/SwordTrailVFX/` | 「Sword Trail VFX」名称の Marketplace / Fab パックと推定 | 要確認 | フォルダ名から推定。Demoフォルダ・Trailsフォルダ構成はMarketplaceパックの典型 | リポジトリ公開注意 |
| `Content/Effects/use/CartoonSmoke/` | 要確認 | 要確認 | `NS_ToonSmoke` の命名からマーケットプレイスの Niagara エフェクトパックと推定されるが、具体的なパック名・ライセンスは不明 | 要確認 |
| `Content/Effects/use/NiagaraSnowfall/` | 要確認 | 要確認 | Niagara snowfall 系チュートリアル素材またはMarketplaceパックと推定。具体的なライセンスは不明 | 要確認 |

#### 4-2. 武器・その他 3D メッシュ

| パス | 推定出典 | ライセンス | 根拠 | 状態 |
|------|----------|-----------|------|------|
| `Content/Weapons/Meshes/Axe/SM_Axe.uasset` | 不明 | 要確認 | バイナリから明確な出典パスを取得できず。自作またはMarketplaceと推定 | 要確認 |
| `Content/Weapons/Meshes/Shield/SM_Shield.uasset` | 不明 | 要確認 | 同上 | 要確認 |
| `Content/StaticMeshes/SM_Sword.uasset` | 不明（`_Assets/SM_Sword.FBX` からインポート） | 要確認 | バイナリ内インポートパスは `_Assets/SM_Sword.FBX`（汎用フォルダ名）。自作・配布素材・Marketplaceのいずれかだが特定できず | 要確認 |
| `Content/Weapons/Meshes/Rifle/SM_Rifle.uasset` | Lyra Starter Game（Epic Games） | UE EULA | バイナリ内に `ArtSource_afarmer/depot/ArtSource/UE5/Lyra/Weapons/Rifle/SM_rifle.fbx` を確認済み | 確認済み |

#### 4-3. UINavigation プラグイン由来コンテンツ

| パス | 推定出典 | ライセンス | 根拠 | 状態 |
|------|----------|-----------|------|------|
| `Content/Examples/` (W_UINavComponent, UINavExampleMenu 等) | UINavigation プラグイン（Marketplace / Fab） | 要確認 | `Config/DefaultEngine.ini` に `GlobalDefaultGameMode=/UINavigation/UINavGameMode.UINavGameMode_C` の参照あり。`Content/Examples/` 内に UINav 系ファイルが多数存在。ただし `Plugins/` フォルダが存在せず、プラグインが標準インストールされていない状態 | リポジトリ公開注意 |

> **UINavigation についての補足**: UINavigation は Gonçalo Marques 氏による Marketplace プラグインです。プラグイン本体（Plugins/フォルダ）がリポジトリに含まれていないため、プラグインのライセンス条件は現時点では確認できません。`Content/Examples/` のアセットはプラグインに同梱のサンプルコンテンツと思われます。Marketplace プラグインのコンテンツを GitHub 上で公開する際は、販売者の利用規約を確認してください。

---

### 5. プラグイン

| プラグイン名 | パス | 有効/無効 | 出典 | ライセンス | 根拠 | 注意点 |
|------------|------|-----------|------|-----------|------|--------|
| ModelingToolsEditorMode | エンジン組み込み | 有効（Editor Only） | UE5 標準 | UE EULA | `.uproject` に記載あり。エディタ専用 | パッケージング非対象。問題なし |
| MeleeTrace | Plugins/（存在しない） | **無効** | Marketplace | 要確認 | `.uproject` に MarketplaceURL: `com.epicgames.launcher://ue/marketplace/product/42b0bb860bcd4995b70d40f152a6e9b9` あり | **無効のためゲームには影響しない。** ただし `Plugins/` フォルダが存在しないため実際には組み込まれていない |
| UINavigation | Plugins/（存在しない） | **参照あり・Plugins/なし** | Marketplace（Gonçalo Marques） | 要確認 | `DefaultEngine.ini` にクラス参照あり。`Content/Examples/` に関連コンテンツあり | プラグイン本体がリポジトリに含まれていないが、関連コンテンツが Content に残っている。プラグインなしではコンパイルエラーになる可能性あり（現在は Warning のみで Cook 成功） |

---

### 6. 音源

| フォルダ / ファイル | 推定出典 | ライセンス | クレジット要否 | 根拠 | 状態 |
|-------------------|----------|-----------|--------------|------|------|
| `Content/sound/BGM/Boss.uasset` | 不明 | 要確認 | 不明 | BGM ファイル。インポートメタデータからはオリジナルファイルパスを確認できず。Suno AI 生成・自作・フリー素材のいずれかと推定 | 要確認 |
| `Content/sound/BGM/FlyingPan.uasset` | 不明 | 要確認 | 不明 | 同上 | 要確認 |
| `Content/sound/BGM/Teppan.uasset` | 不明 | 要確認 | 不明 | 同上 | 要確認 |
| `Content/sound/BGM/Title.uasset` | 不明 | 要確認 | 不明 | 同上 | 要確認 |
| `Content/sound/BGM/reizouko.uasset` | 不明 | 要確認 | 不明 | 同上 | 要確認 |
| `Content/sound/SE/` (Button, Clear, Gass, KETCHAP, Lightning 等) | 不明 | 要確認 | 不明 | SE ファイル群。出典を特定できる情報が見つからず | 要確認 |
| `Content/sound/boss/Voice_boss_*` | 不明（ボイス素材） | 要確認 | 不明 | ボス敵の Voice ファイル。録音素材・ボイス生成AI・フリー素材のいずれかと推定 | 要確認 |
| `Content/sound/pan/Voice_pan_*` | 不明（ボイス素材） | 要確認 | 不明 | プレイヤーキャラクター（パン）の Voice ファイル。同上 | 要確認 |
| `Content/sound/sosage/Voice_sosage_*` | 不明（ボイス素材） | 要確認 | 不明 | ソーセージ敵の Voice ファイル。同上 | 要確認 |
| `Content/Audio/sword-swing`, `axe-swing`, `axe-block` 等 | 不明 | 要確認 | 不明 | 英語名の汎用 SE。Freesound・効果音ラボ・その他フリーSE サイト等と推定されるが確認できず | 要確認 |
| `Content/Audio/Weapons_Rifle2_Punch_01`, `Weapons_Rifle2_Sweetener_01` | Lyra Starter Game またはその他パックと推定 | 要確認 | 不明 | 命名パターン `Weapons_Rifle2_*` が特定のサウンドパックと推定されるが確認できず | 要確認 |
| `Content/Projectiles/Audio/` | 不明 | 要確認 | 不明 | サウンド出典を確認できず | 要確認 |

---

### 7. 画像素材・UI素材

| パス / ファイル | 推定出典 | ライセンス | クレジット要否 | 根拠 | 状態 |
|---------------|----------|-----------|--------------|------|------|
| `Content/Title/Picture/ChatGPT_Image_2025年5月10日_*` 等（7ファイル） | ChatGPT / DALL-E（OpenAI） | OpenAI 利用規約 | 要確認 | **ファイル名に `ChatGPT_Image_` プレフィックスあり（確実に AI 生成）。** 生成日: 2025年5月8日〜7月1日の複数ファイル | **リポジトリ公開注意** |
| `Content/Title/Picture/Gemini_Generated_Image_z32366z32366z323.uasset` | Google Gemini | Google 利用規約 | 要確認 | **ファイル名に `Gemini_Generated_Image_` プレフィックスあり（確実に AI 生成）** | **リポジトリ公開注意** |
| `Content/ThirdPerson/image/ChatGPT_Image_2025年5月10日_23_55_02.uasset` | ChatGPT / DALL-E（OpenAI） | OpenAI 利用規約 | 要確認 | ファイル名から判断 | **リポジトリ公開注意** |
| `Content/Title/Picture/5C6C25E9-155C-4390-9E8F-17855FACA1A8.uasset` | 不明（UUID形式のファイル名） | 要確認 | 不明 | UUID形式のファイル名から出典を特定できず。スクリーンショット・写真・生成AI等のいずれかと推定 | 要確認 |
| `Content/Title/Picture/Photos_Library.uasset` | 不明 | 要確認 | 不明 | `Photos_Library` という名称から、macOS の写真ライブラリから取り込んだ画像と推定。実際の写真の権利は撮影者・被写体に帰属する可能性あり | **要確認（公開注意）** |
| `Content/Title/Picture/` その他（EASY, HARD, NORMAL, GameOver, HOME 等） | 自作または AI 生成と推定 | 要確認 | 不明 | ゲーム UI 用素材。出典を特定できる情報がなく、自作または AI 生成素材と推定 | 要確認 |
| `Content/Widgets/Textures/T_Crosshair_Dot.uasset` | 不明 | 要確認 | 不明 | クロスヘア素材。自作または汎用素材と推定 | 要確認 |
| `Content/Widgets/HealthBar/BarImage/HPbar`, `HeatGageBackGround` 等 | 自作と推定 | — | — | ゲーム固有のUI素材と判断 | 確認済み（自作推定） |

> **AI 生成画像についての補足（2026年5月時点）**:  
> - OpenAI (ChatGPT/DALL-E) の利用規約では、ユーザーが生成した画像のコンテンツ権利はユーザーに帰属するとされています。ただし、商用利用・GitHub 公開については最新規約を必ず確認してください。  
> - Google Gemini の生成画像についても同様に、最新の利用規約を確認してください。  
> - 就活・インターン提出物での AI 生成画像の使用は、提出先の規定を確認した上で使用していることを明示することを推奨します。

---

### 8. 生成 AI 素材・コード生成 AI の利用

#### 8-1. AI 生成素材の一覧

| 素材 | 使用 AI | 確認方法 | 使用範囲 | 注意点 |
|------|---------|---------|---------|--------|
| タイトル・UI 画像（7点以上） | ChatGPT / DALL-E（OpenAI） | ファイル名に `ChatGPT_Image_` プレフィックスあり | `Content/Title/Picture/`, `Content/ThirdPerson/image/` | GitHub 公開時は AI 生成であることの明示を検討 |
| キャラクター画像（1点） | Google Gemini | ファイル名に `Gemini_Generated_Image_` プレフィックスあり | `Content/Title/Picture/` | 同上 |
| BGM・ボイス | 不明（Suno AI 等の可能性） | 確認できず | `Content/sound/BGM/`, `Content/sound/boss/pan/sosage/` | 出典が AI 生成の場合は最新の各サービス利用規約を確認すること |

#### 8-2. コード生成 AI（Claude Code）の利用

| 項目 | 内容 |
|------|------|
| 使用ツール | Claude Code（Anthropic） |
| 主な使用範囲 | パッケージ化エラー（`/Game/Stylized_Room`）の原因調査・修正案立案、`Config/DefaultGame.ini` への MapsToCook 追記、`.gitignore` 修正、`Docs/` ドキュメント作成支援 |
| 人間が判断した範囲 | 修正内容の採用判断、実機でのゲーム挙動確認、AI/Blueprint 実装全般の設計・実装 |
| AI が行っていない範囲 | Blueprint ノードの実際の設計・実装、Behavior Tree 構造の設計、EQS 設定、マップ配置、ゲームプレイの仕様決定 |
| 面接で説明できるようにしておくべき箇所 | 敵 AI 全般（AIC_Enemy_Base, BB_Enemy_Base, BT_Enemy_Melee 等）、EQS 設定（FindCover, Strafe, Teleport）、ウェーブ進行（Spawner_BP）、ダメージシステム（BPI_Damagable）、攻撃トークン制御（ReserveAttackToken）|

---

## リポジトリ公開時の注意

| 対象 | リスク | 推奨対応 |
|------|--------|---------|
| AI 生成画像（ChatGPT, Gemini） | 著作権の帰属・商用利用条件が変化している可能性。GitHub への掲載が「再配布」に該当するかどうかも要確認 | 最新の OpenAI / Google 利用規約を確認。プロフォリオ公開用には AI 生成であることを README 等に明記することを推奨 |
| Marketplace アセット（FXVarietyPack, StylizedKitchen, sA_SplatterofbloodFx, Effects/tool 等） | Fab / Marketplace の Standard License では、アセット単体のソースデータ（.uasset）の GitHub 公開（＝再配布）が禁止されている可能性が高い | パブリックリポジトリへの掲載前に各アセットの販売ページのライセンス条件を確認。.gitignore や LFS の活用を検討 |
| Mixamo アニメーション（.uasset として commit 済み） | Mixamo 利用規約上、アセット単体の再配布が制限されている可能性がある | プライベートリポジトリでの運用を検討。ES 提出先への公開範囲を限定するか、利用規約を再確認する |
| UINavigation プラグインコンテンツ（Content/Examples/） | Plugins/ フォルダが存在しないが、プラグイン由来のサンプルコンテンツが Content/ に含まれている | Marketplace プラグインのコンテンツ公開条件を確認。不要であれば削除を検討 |
| Photos_Library.uasset | macOS の写真ライブラリ由来の可能性。写真の権利（著作権・肖像権）に注意 | 画像の出典と権利を確認。問題がある場合は削除・差し替えを検討 |
| AI 生成ボイス・BGM（可能性あり） | Suno AI 等のサービスによる生成の場合、商用利用・再配布条件を確認する必要がある | 出典を確認し、利用規約上問題がある場合はゲーム内使用・GitHub 掲載を分けて対応 |

---

## 提出前 TODO

| 優先度 | TODO | 対象 | 理由 |
|--------|------|------|------|
| 高 | BGM・SE・ボイス素材の出典を全件確認する | `Content/sound/`, `Content/Audio/` | 出典・ライセンス不明のまま提出するとリスクあり |
| 高 | AI 生成画像の使用について提出先の規定を確認する | `Content/Title/Picture/ChatGPT_Image_*` 等 | 就活・インターン提出での AI 生成素材の扱い方は提出先によって異なる |
| 高 | Marketplace アセットのライセンスを一つずつ確認する（FXVarietyPack, StylizedKitchen, sA_SplatterofbloodFx, Effects/tool） | 上記フォルダ | GitHub パブリック公開前に必要 |
| 高 | Mixamo 利用規約の最新版を確認し、.uasset 公開が許容されるか確認する | `Content/Animations/`, `Content/Characters/Mage/` | GitHub パブリック公開時のリスク低減 |
| 中 | Photos_Library.uasset の内容と権利を確認する | `Content/Title/Picture/Photos_Library.uasset` | 写真・画像の著作権・肖像権リスク |
| 中 | UINavigation プラグインの取り扱いを整理する | `Content/Examples/`, `Config/DefaultEngine.ini` | プラグイン本体なしに関連コンテンツが残っている状態の整理 |
| 中 | SM_Axe, SM_Shield, SM_Sword の出典を確認する | `Content/Weapons/Meshes/`, `Content/StaticMeshes/` | 武器モデルの出典が不明 |
| 低 | 未使用の Showcase マップ・アセットの整理を検討する | `Content/StylizedKitchen/`, `Content/Effects/`, `Content/sA_SplatterofbloodFx/` | パッケージ化からは除外済みだが、リポジトリに含まれたまま |
| 低 | Shipping 設定でのパッケージ化を確認する | `Docs/PACKAGING_REPORT.md` 参照 | 最終提出前に Development → Shipping に変更 |

---

## 参照した情報

| 情報源 | 内容 |
|--------|------|
| `Content/` 全フォルダの `find` 出力 | フォルダ・ファイル構造の把握 |
| `*.uasset` バイナリ内 `RelativeFilename` 文字列 | pan_teisei, furank_teisei, sarami_teisei, pork_teisei（shimadahajime ローカルパス確認）; sword-idle（mixamo_converter パス確認）; axe-idle（mixamo_converter + mixamo.com 文字列確認）; SKM_Mage（Mixamo_assets/mage-character-with-idle-anim.fbx + Kachujin 確認）; SM_Rifle（Lyra パス確認） |
| `Config/DefaultEngine.ini` | `GlobalDefaultGameMode=/UINavigation/UINavGameMode.UINavGameMode_C` 参照確認 |
| `SausageHunter.uproject` | MeleeTrace プラグインの MarketplaceURL 確認 |
| `README.md` | 担当範囲・実装内容の記載を参照 |
| ファイル名 | ChatGPT_Image_* / Gemini_Generated_Image_* による AI 生成確認 |
