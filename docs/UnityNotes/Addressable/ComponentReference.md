# ComponentReference

指定加载 生成携带 某个组件 的物体

```C#
using System;
using UnityEngine;

public class ColorChanger : MonoBehaviour
{
    public void SetColor(Color col)
    {
        var rend = GetComponent<MeshRenderer>();
        var material = rend.material;
        material.color = col;
    }
}

[Serializable]
public class ComponentReferenceColorChanger : ComponentReference<ColorChanger>
{
    public ComponentReferenceColorChanger(string guid)
        : base(guid) { }
}
```



```C#
public ComponentReferenceColorChanger ColorShifterReference;

ColorShifterReference.LoadComponentAsync().Completed += LoadDone;
ColorShifterReference.InstantiateAsync().Completed += InstantiateDone;

void LoadDone(AsyncOperationHandle<ColorChanger> obj)
{
    if (obj.Result != null)
    {
        ColorChanger changer = Instantiate(obj.Result);
        changer.SetColor(RandomColor());
    }
}

//if using the InstantiateComponentAsync version above...
void InstantiateDone(AsyncOperationHandle<ColorChanger> obj)
{
    if(obj.Result != null)
        obj.Result.SetColor(RandomColor());
}

Color RandomColor()
{
    return new Color(UnityEngine.Random.Range(0.0f, 1.0f), UnityEngine.Random.Range(0.0f, 1.0f), UnityEngine.Random.Range(0.0f, 1.0f));
}
```



- ComponentReference
  我们选择关心的组件类型。
  释放ComponentReference应该通过ComponentReference类中的ReleaseInstance()来完成。

```C#
public class ComponentReference<TComponent> : AssetReference
{
    public void ReleaseInstance(AsyncOperationHandle<TComponent> op)
    {
        // Release the instance
        var component = op.Result as Component;
        if (component != null)
        {
            Addressables.ReleaseInstance(component.gameObject);
        }

        // Release the handle
        Addressables.Release(op);
    }
}
```



