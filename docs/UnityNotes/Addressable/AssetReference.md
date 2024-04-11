

# AssetReference

```C#
using UnityEngine;
using UnityEngine.AddressableAssets;

public class BasicReference : MonoBehaviour
{
	public AssetReference baseCube;

	public void SpawnThing()
	{
		baseCube.InstantiateAsync();
	}
}
```

```C#
// release == false, 说明这个物体不是通过 Address Instantiate
// 所以需要手动删除
// release == true, 物体会被删除, 并正确回收。
void Release()
{
    bool release = Addressables.ReleaseInstance(gameObject);
    if (!release) Destroy(gameObject);
}
```

- 场景关闭时仍然存在的任何实例都将通过场景关闭过程自动释放（递减引用计数）（即使它们的自毁脚本被禁用）。



```C#
public List<AssetReference> shapes;

void Start ()
{
    foreach (var shape in shapes)
    {
        shape.LoadAssetAsync<GameObject>().Completed += OnShapeLoaded;
    }
}

public void SpawnAThing()
{
     for(int count = 0; count <= currentIndex; count++)
            GameObject.Instantiate(shapes[currentIndex].Asset);
        currentIndex++;
        if (currentIndex >= shapes.Count)
            currentIndex = 0;
}

void OnDestroy()
{
    foreach (var shape in shapes)
    {
        shape.ReleaseAsset();
    }
}
```

- 这里的对象是通过传统方式实例化的`GameObject.Instantiate`，不会增加 Addressables 引用计数。这些对象仍然调用 Addressables 来释放它们自己，但是由于它们不是通过 Addressables 实例化的，所以释放只会销毁对象，而不会减少引用计数。

- 这些 AssetReferences 的管理者必须释放它们，`OnDestroy`否则引用计数将在场景关闭后继续存在。

# FilteredReferences

```C#
[Serializable]
public class AssetReferenceMaterial : AssetReferenceT<Material>
{
    public AssetReferenceMaterial(string guid) : base(guid) { }
}

public AssetReferenceGameObject rightObject;
public AssetReferenceMaterial spawnMaterial;
```





# SubobjectReference

```C#
public AssetReference sheetReference;
public AssetReference sheetSubReference;
public List<SpriteRenderer> spritesToChange;

public void LoadMainAsset()
{
    sheetReference.LoadAssetAsync<IList<Sprite>>().Completed += AssetDone;
}

public void LoadSubAsset()
{
    sheetSubReference.LoadAssetAsync<Sprite>().Completed += Subassetdone;
}

void AssetDone(AsyncOperationHandle<IList<Sprite>> op)
{
    if (op.Result == null)
    {
        Debug.LogError("no sheets here.");
        return;
    }

    spritesToChange[0].sprite = op.Result[1];
}

 void Subassetdone(AsyncOperationHandle<Sprite> op)
{
    if (op.Result == null)
    {
        Debug.LogError("no sprite in sheet here.");
        return;
    }

    spritesToChange[1].sprite = op.Result;
}
```

- 假设有子依赖，可以在inspector面板选择需要加载的单个子依赖。
- 也可以不选择子依赖，而是加载整个资源并用代码去选择List中的指定元素。





























