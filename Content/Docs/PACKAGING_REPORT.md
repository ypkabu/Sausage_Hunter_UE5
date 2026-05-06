# Packaging Report

## 実行日時

2026-05-06

---

## 実行したコマンド

```powershell
"C:\Program Files\Epic Games\UE_5.4\Engine\Build\BatchFiles\RunUAT.bat" BuildCookRun `
  -project="C:\WipeAndSnap\Sausage_Hunter_UE5\SausageHunter.uproject" `
  -noP4 `
  -platform=Win64 `
  -clientconfig=Development `
  -cook `
  -stage `
  -pak `
  -archive `
  -archivedirectory="C:\WipeAndSnap\Sausage_Hunter_UE5\Builds\Windows" `
  -noBuild
```

---

## 使用したUEバージョン

- **UE 5.4**（`C:\Program Files\Epic Games\UE_5.4`）
- uprojectの `EngineAssociation: "5.4"` と一致

---

## 発生したエラー

**エラー: 0件（BUILD SUCCESSFUL）**

---

## 修正した内容

### 1. `/Game/Stylized_Room` 参照問題の解決

**原因**:
- `Content/StylizedKitchen/Levels/Stylized_Interior.umap` がバイナリ内で  
  `/Game/Stylized_Room` および `/Game/Stylized_Room.Stylized_Room` を参照
- このパッケージは存在しない（削除済みまたは未コミット）
- `MapsToCook` が未指定だったため、Cook時にすべての `.umap` が対象となり、  
  この壊れた参照を持つ `Stylized_Interior.umap` もCookしようとしていた

**修正**:
`Config/DefaultGame.ini` に `[/Script/UnrealEd.ProjectPackagingSettings]` セクションを追加し、  
ゲーム本編マップ10本だけを `MapsToCook` で明示指定（デモ・Showcaseマップを除外）:

```ini
[/Script/UnrealEd.ProjectPackagingSettings]
+MapsToCook=(FilePath="/Game/Title/Title")
+MapsToCook=(FilePath="/Game/Title/Death")
+MapsToCook=(FilePath="/Game/Title/Death_Crear")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Stage1")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Stage2")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Stage3")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Boss1")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Boss2")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Boss3")
+MapsToCook=(FilePath="/Game/ThirdPerson/Maps/Normal")
```

Cook対象から除外されたマップ（壊れた参照を持つ）:
- `/Game/StylizedKitchen/Levels/Stylized_Interior`（`/Game/Stylized_Room` を参照）
- `/Game/StylizedKitchen/Levels/AssetsOverview`
- `/Game/Effects/tool/SceneAssets/DemoMap`
- `/Game/Effects/tool/SwordTrailVFX/Maps/Demo`
- `/Game/Effects/use/CartoonSmoke/Map/Overview`
- `/Game/Effects/use/NiagaraSnowfall/Map/Overview`
- `/Game/sA_SplatterofbloodFx/Levels/Lv_Showroom`
- `/Game/ThirdPerson/Maps/ThirdPersonMap`（テンプレートマップ）
- `/Game/StarterContent/Maps/*`

### 2. `.gitignore` に `images/` を追加

`images/` フォルダはすでに未追跡（`??`）だったため `git rm` は不要。  
`.gitignore` に1行追加してgit管理対象外を明示。

---

## 残っているWarning

### 重要度：中（提出前に確認推奨）

| Warning | 内容 |
|---------|------|
| `LogCook: Warning: Unable to generate long package name, /UINavigation/UINavGameMode` | UINavigationプラグインがない。`GlobalDefaultGameMode` が `/UINavigation/UINavGameMode.UINavGameMode_C` を指しているが、Pluginsフォルダが存在しない。ゲーム自体はマップ単位でGameModeを上書きしていると思われるため実害は少ないが、タイトル等でUINavigationを使っている場合は動作確認が必要 |
| `SM_Platform.uasset: '/Engine/TemplateResources/MI_Template_Red_Metal'` | エンジンテンプレートのマテリアル参照が解決できない。エンジン内の実際のパスが変わった可能性。実害は「そのメッシュにデフォルトマテリアルが使われる」程度 |
| AnimMontage out-of-sync (Sword Hit / Rifle Hit) | AnimMontageのセグメント長さとAnimSequenceの長さがわずかにずれている。エディタで「ResaveAll」をかけると直る |

### 重要度：低（ParagonFX系マテリアル）

多数の Paragon FX マテリアルが `PCD3D_SM6` / `PCD3D_SM5` でコンパイル失敗し、デフォルトマテリアルで代替。  
これらのエフェクトが実際にゲームで使われている場合、見た目が白/グレーになる可能性あり。

---

## 生成されたファイルの場所

```
C:\WipeAndSnap\Sausage_Hunter_UE5\Builds\Windows\
├── SausageHunter.exe              ← メインの実行ファイル
├── Engine\                        ← エンジンバイナリ
├── SausageHunter\
│   └── Content\
│       └── Paks\
│           └── SausageHunter-Windows.pak  ← ゲームコンテンツ（~706MB）
├── Manifest_UFSFiles_Win64.txt
├── Manifest_NonUFSFiles_Win64.txt
└── Manifest_DebugFiles_Win64.txt
```

---

## 次に人間が確認すべきこと

1. **実行確認**: `Builds\Windows\SausageHunter.exe` を実行してタイトル画面が出るか
2. **UINavigation**: タイトル/UIがUINavigation依存なら、プラグインをPluginsフォルダに配置して再Cook
3. **ParagonFX**: ゲーム内でParagon系エフェクトが正常に表示されるか目視確認
4. **アニメーション**: 剣の被ダメ・ライフル被ダメモーションが正常か
5. **uproject整理**: `Soseji_Hunter.uproject` 削除 / `SausageHunter.uproject` 追加のgit commitが未実施
6. **Shipping最適化**: 提出最終版はclientconfigを `Shipping` に変更（デバッグ情報なし）

---

## 合計時間

- Cook: 約 19秒（Commandlet）
- Pak: 約 5秒
- 合計: 約 28秒（キャッシュ活用）
