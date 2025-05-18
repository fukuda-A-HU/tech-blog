---
title: "Unity AssetBundleでアバターを簡単に読み込む方法"
emoji: "👾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "assetbundle", "avatar", "csharp", "humanoid"]
published: false
---

## はじめに：AssetBundleでアバターを読み込む利点

Unityでゲームやアプリを開発している際、アバターなどの大型アセットをどのように扱うかは重要な問題です。特にアバターはモデルデータ、テクスチャ、シェーダー、アニメーションなど多くのリソースを含むため、効率的な読み込み方法が必要です。

この記事では、AssetBundleを使ってHumanoidアバターを動的に読み込む方法を初心者向けに解説します。AssetBundleを活用することで以下のメリットがあります：

- アプリ本体のサイズを小さく保てる
- 必要なときだけアセットをダウンロードできる
- アプリのアップデートなしにコンテンツを追加できる
- シェーダーなどの依存関係も一括管理できる

## 必要なもの

- Unity 2020.3以降（この記事では2022.3 LTS使用）
- 基本的なC#とUnityの知識
- Humanoid形式のアバターモデル
- カスタムシェーダー（例としてURP/Litを使用）

## 1. AssetBundleの基本

AssetBundleとは、Unityアセットをパッケージ化して配布するための仕組みです。アプリ起動後に動的にロードできるため、初期インストールサイズを削減できます。

### AssetBundleのビルド準備

まず、アバターとシェーダーをAssetBundleとしてビルドするための設定をします。

1. アバターモデルを選択し、インスペクターの下部にある「AssetBundle」項目で新しいバンドル名（例：`avatar_bundle`）を入力します
2. シェーダーを含むマテリアルも同様に選択し、同じまたは別のバンドル名（例：`shader_bundle`）を設定します

```csharp
// AssetBundle情報を確認するヘルパーメソッド
public static void PrintAssetBundleInfo(string bundlePath)
{
    var manifest = AssetBundle.LoadFromFile(bundlePath + "/assetbundle");
    if (manifest != null)
    {
        Debug.Log($"AssetBundleマニフェスト読み込み成功: {manifest.name}");
        
        // バンドル内のアセット一覧表示
        string[] assetNames = manifest.GetAllAssetNames();
        foreach (var name in assetNames)
        {
            Debug.Log($"アセット名: {name}");
        }
    }
    else
    {
        Debug.LogError("AssetBundleマニフェスト読み込み失敗");
    }
}
```

### AssetBundleをビルドする

次に、エディタスクリプトを作成してAssetBundleをビルドします。

```csharp
using UnityEditor;
using System.IO;
using UnityEngine;

public class AssetBundleBuilder
{
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAllAssetBundles()
    {
        // 出力先ディレクトリを作成
        string assetBundleDirectory = "Assets/AssetBundles";
        if (!Directory.Exists(assetBundleDirectory))
        {
            Directory.CreateDirectory(assetBundleDirectory);
        }
        
        // AssetBundleのビルド
        BuildPipeline.BuildAssetBundles(
            assetBundleDirectory,
            BuildAssetBundleOptions.None,
            BuildTarget.StandaloneWindows64);
            
        Debug.Log("AssetBundleのビルドが完了しました！");
    }
}
```

このスクリプトを`Editor`フォルダ内に保存し、Unityメニューから「Assets > Build AssetBundles」を選択してビルドします。

## 2. アバターとシェーダーの読み込み

AssetBundleからHumanoidアバターとシェーダーを読み込むスクリプトを作成します。

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.Networking;

public class AvatarLoader : MonoBehaviour
{
    [SerializeField] private string avatarBundleUrl = "http://example.com/bundles/avatar_bundle";
    [SerializeField] private string shaderBundleUrl = "http://example.com/bundles/shader_bundle";
    [SerializeField] private string avatarPrefabName = "HumanoidAvatar";
    [SerializeField] private Transform avatarSpawnPoint;
    
    private AssetBundle avatarBundle;
    private AssetBundle shaderBundle;
    
    void Start()
    {
        StartCoroutine(LoadAvatarWithShaders());
    }
    
    IEnumerator LoadAvatarWithShaders()
    {
        // 1. シェーダーバンドルを先に読み込む
        using (UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle(shaderBundleUrl))
        {
            yield return www.SendWebRequest();
            
            if (www.result != UnityWebRequest.Result.Success)
            {
                Debug.LogError($"シェーダーバンドルの読み込み失敗: {www.error}");
                yield break;
            }
            
            shaderBundle = DownloadHandlerAssetBundle.GetContent(www);
            Debug.Log("シェーダーバンドル読み込み成功！");
        }
        
        // 2. アバターバンドルの読み込み
        using (UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle(avatarBundleUrl))
        {
            yield return www.SendWebRequest();
            
            if (www.result != UnityWebRequest.Result.Success)
            {
                Debug.LogError($"アバターバンドルの読み込み失敗: {www.error}");
                yield break;
            }
            
            avatarBundle = DownloadHandlerAssetBundle.GetContent(www);
            Debug.Log("アバターバンドル読み込み成功！");
        }
        
        // 3. アバターをインスタンス化
        GameObject avatarPrefab = avatarBundle.LoadAsset<GameObject>(avatarPrefabName);
        if (avatarPrefab != null)
        {
            GameObject avatarInstance = Instantiate(avatarPrefab, 
                                                   avatarSpawnPoint.position, 
                                                   avatarSpawnPoint.rotation);
            
            // アバターのセットアップ処理
            SetupAvatar(avatarInstance);
            
            Debug.Log("アバターのインスタンス化と設定完了！");
        }
        else
        {
            Debug.LogError($"アバタープレハブ '{avatarPrefabName}' が見つかりません");
        }
    }
    
    void SetupAvatar(GameObject avatar)
    {
        // アニメーターの設定
        Animator animator = avatar.GetComponent<Animator>();
        if (animator != null)
        {
            // ランタイムコントローラーがあれば設定
            // animator.runtimeAnimatorController = yourController;
            
            // アニメーターの有効化
            animator.applyRootMotion = true;
            Debug.Log("アニメーター設定完了");
        }
        
        // シェーダーの修正
        FixShaderReferences(avatar);
    }
    
    void FixShaderReferences(GameObject avatar)
    {
        // アバターの全てのレンダラーを取得
        Renderer[] renderers = avatar.GetComponentsInChildren<Renderer>(true);
        
        foreach (Renderer renderer in renderers)
        {
            // 各マテリアルを取得
            foreach (Material material in renderer.materials)
            {
                // シェーダー名を取得
                string shaderName = material.shader.name;
                
                // シェーダーバンドルから対応するシェーダーを読み込み
                Shader bundledShader = shaderBundle.LoadAsset<Shader>(shaderName);
                if (bundledShader != null)
                {
                    // シェーダーを置き換え
                    material.shader = bundledShader;
                    Debug.Log($"シェーダー置き換え成功: {shaderName}");
                }
                else
                {
                    Debug.LogWarning($"バンドル内にシェーダーが見つかりません: {shaderName}");
                }
            }
        }
    }
    
    void OnDestroy()
    {
        // バンドルのアンロード
        if (avatarBundle != null)
            avatarBundle.Unload(false);
            
        if (shaderBundle != null)
            shaderBundle.Unload(false);
    }
}
```

## 3. オフラインキャッシュの活用

アバターを毎回ダウンロードするのは非効率なので、キャッシュ機能を実装しましょう。

```csharp
IEnumerator LoadAssetBundleWithCaching(string url, string bundleName, System.Action<AssetBundle> onComplete)
{
    // キャッシュ用のパス
    string cachedPath = $"{Application.persistentDataPath}/BundleCache/{bundleName}";
    
    // キャッシュがあるか確認
    if (File.Exists(cachedPath))
    {
        AssetBundle cachedBundle = AssetBundle.LoadFromFile(cachedPath);
        if (cachedBundle != null)
        {
            Debug.Log($"キャッシュからロード: {bundleName}");
            onComplete(cachedBundle);
            yield break;
        }
    }
    
    // キャッシュがない場合はダウンロード
    using (UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle(url))
    {
        yield return www.SendWebRequest();
        
        if (www.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError($"ダウンロード失敗: {www.error}");
            onComplete(null);
            yield break;
        }
        
        AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(www);
        
        // キャッシュディレクトリを作成
        Directory.CreateDirectory(Path.GetDirectoryName(cachedPath));
        
        // ダウンロードしたデータをキャッシュに保存
        File.WriteAllBytes(cachedPath, www.downloadHandler.data);
        
        Debug.Log($"バンドルをキャッシュに保存: {bundleName}");
        onComplete(bundle);
    }
}
```

## 4. アバターのコントロールと操作

読み込んだHumanoidアバターを動かすための基本的なコントロールを実装します。

```csharp
public class AvatarController : MonoBehaviour
{
    private Animator animator;
    
    // アニメーションパラメータ名
    private readonly string walkParam = "IsWalking";
    private readonly string runParam = "IsRunning";
    private readonly string jumpParam = "Jump";
    
    private void Start()
    {
        animator = GetComponent<Animator>();
        
        if (animator == null)
        {
            Debug.LogError("Animatorコンポーネントが見つかりません");
            enabled = false;
        }
    }
    
    private void Update()
    {
        // 移動入力の取得
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");
        
        // 移動があるかチェック
        bool isMoving = Mathf.Abs(horizontal) > 0.1f || Mathf.Abs(vertical) > 0.1f;
        
        // 走るキー（Shift）が押されているか
        bool isRunning = Input.GetKey(KeyCode.LeftShift) && isMoving;
        
        // ジャンプキー
        bool isJumping = Input.GetKeyDown(KeyCode.Space);
        
        // アニメーターパラメータ設定
        animator.SetBool(walkParam, isMoving && !isRunning);
        animator.SetBool(runParam, isRunning);
        
        if (isJumping)
        {
            animator.SetTrigger(jumpParam);
        }
    }
}
```

## 5. アバター切り替え機能

複数のアバターを切り替えられる機能を追加します。

```csharp
public class AvatarSwitcher : MonoBehaviour
{
    [SerializeField] private string[] avatarBundleUrls;
    [SerializeField] private string[] avatarPrefabNames;
    [SerializeField] private Transform spawnPoint;
    
    private int currentAvatarIndex = -1;
    private GameObject currentAvatar;
    
    // 事前にシェーダーバンドルを読み込んでおく
    [SerializeField] private string shaderBundleUrl;
    private AssetBundle shaderBundle;
    
    private void Start()
    {
        StartCoroutine(LoadShaderBundle());
    }
    
    private IEnumerator LoadShaderBundle()
    {
        // シェーダーバンドル読み込み
        using (UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle(shaderBundleUrl))
        {
            yield return www.SendWebRequest();
            
            if (www.result == UnityWebRequest.Result.Success)
            {
                shaderBundle = DownloadHandlerAssetBundle.GetContent(www);
                Debug.Log("シェーダーバンドル読み込み完了");
                
                // 最初のアバターを読み込み
                SwitchAvatar(0);
            }
            else
            {
                Debug.LogError($"シェーダーバンドル読み込み失敗: {www.error}");
            }
        }
    }
    
    public void SwitchToNextAvatar()
    {
        int nextIndex = (currentAvatarIndex + 1) % avatarBundleUrls.Length;
        SwitchAvatar(nextIndex);
    }
    
    public void SwitchToPreviousAvatar()
    {
        int prevIndex = currentAvatarIndex - 1;
        if (prevIndex < 0) prevIndex = avatarBundleUrls.Length - 1;
        SwitchAvatar(prevIndex);
    }
    
    private void SwitchAvatar(int index)
    {
        if (index == currentAvatarIndex || index < 0 || index >= avatarBundleUrls.Length)
            return;
            
        currentAvatarIndex = index;
        StartCoroutine(LoadAndSwitchAvatar(avatarBundleUrls[index], avatarPrefabNames[index]));
    }
    
    private IEnumerator LoadAndSwitchAvatar(string bundleUrl, string prefabName)
    {
        // UIでロード中表示など
        Debug.Log("アバター読み込み中...");
        
        using (UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle(bundleUrl))
        {
            yield return www.SendWebRequest();
            
            if (www.result != UnityWebRequest.Result.Success)
            {
                Debug.LogError($"アバターバンドル読み込み失敗: {www.error}");
                yield break;
            }
            
            AssetBundle avatarBundle = DownloadHandlerAssetBundle.GetContent(www);
            
            // プレハブ読み込み
            GameObject avatarPrefab = avatarBundle.LoadAsset<GameObject>(prefabName);
            if (avatarPrefab == null)
            {
                Debug.LogError($"アバタープレハブが見つかりません: {prefabName}");
                avatarBundle.Unload(false);
                yield break;
            }
            
            // 既存のアバターを破棄
            if (currentAvatar != null)
            {
                Destroy(currentAvatar);
            }
            
            // 新しいアバターをインスタンス化
            currentAvatar = Instantiate(avatarPrefab, spawnPoint.position, spawnPoint.rotation);
            
            // シェーダーの設定
            ApplyShaders(currentAvatar);
            
            // コントローラーの設定
            SetupController(currentAvatar);
            
            // バンドルをアンロード（アセットはメモリに残す）
            avatarBundle.Unload(false);
            
            Debug.Log($"アバター切り替え完了: {prefabName}");
        }
    }
    
    private void ApplyShaders(GameObject avatar)
    {
        if (shaderBundle == null) return;
        
        Renderer[] renderers = avatar.GetComponentsInChildren<Renderer>(true);
        foreach (Renderer renderer in renderers)
        {
            foreach (Material material in renderer.materials)
            {
                string shaderName = material.shader.name;
                Shader bundledShader = shaderBundle.LoadAsset<Shader>(shaderName);
                
                if (bundledShader != null)
                {
                    material.shader = bundledShader;
                }
            }
        }
    }
    
    private void SetupController(GameObject avatar)
    {
        // アバターコントローラー設定
        AvatarController controller = avatar.AddComponent<AvatarController>();
        
        // その他のセットアップ
        Animator animator = avatar.GetComponent<Animator>();
        if (animator != null)
        {
            // アニメーター設定
        }
    }
    
    private void OnDestroy()
    {
        if (shaderBundle != null)
        {
            shaderBundle.Unload(true);
        }
    }
}
```

## よくある問題と解決方法

### 1. ピンクテクスチャ問題（シェーダーが見つからない）

AssetBundleでアバターを読み込むと、モデルは表示されるがマテリアルがピンク色になることがあります。これはシェーダーの参照が正しく解決されていない証拠です。

**解決方法:**
- シェーダーを含むバンドルを先に読み込む
- `material.shader = bundledShader` で明示的にシェーダーを設定する
- URPを使用している場合は、URPアセットもバンドルに含める

### 2. Humanoidアニメーションが動作しない

アバターがT-poseのままで動かない場合は、以下を確認しましょう。

**解決方法:**
- アニメーターコントローラーが正しく設定されているか確認
- Avatarの設定が「Humanoid」になっているか確認
- アバターのボーン構造がHumanoidマッピングに適合しているか確認

### 3. パフォーマンスの最適化

複数のアバターを扱う場合、メモリ使用量が問題になることがあります。

**最適化方法:**
- 未使用のAssetBundleはこまめに`Unload(true)`する
- LOD (Level Of Detail)を活用してポリゴン数を削減
- テクスチャサイズとシェーダー複雑性のバランスを取る

## まとめ

この記事では、UnityでAssetBundleを使ってHumanoidアバターを動的に読み込む方法を解説しました。ポイントをまとめると：

1. アバターとシェーダーを適切なバンドルに分ける
2. シェーダーを先に読み込んでから、アバターを読み込む
3. シェーダー参照を明示的に修正する
4. キャッシュ機能を実装して読み込み時間を短縮
5. 複数アバターの切り替え機能を実装

これらのテクニックを活用することで、効率的なアバターローディングシステムを構築できます。AssetBundleはアプリケーションの容量削減だけでなく、コンテンツ更新の柔軟性も向上させるため、積極的に活用していきましょう。

## 参考リンク

- [Unity公式ドキュメント - AssetBundles](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)
- [Unity AssetBundle Fundamentals](https://learn.unity.com/tutorial/assets-resources-and-assetbundles)
- [Unity Humanoid Animation](https://docs.unity3d.com/Manual/ConfiguringtheAvatar.html)
