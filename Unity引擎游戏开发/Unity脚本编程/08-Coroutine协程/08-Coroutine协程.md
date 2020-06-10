# Coroutine协程

Unity中的Coroutine协程机制，能够帮助我们实现延时等功能。

## Coroutine例子

下面代码实现了延时3秒，将游戏对象设为active的功能。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoroutineTest : MonoBehaviour
{
    public GameObject o;
    void Start()
    {
        // 延时3秒，将游戏对象设为active
        StartCoroutine(delayActive());
    }

    IEnumerator delayActive()
    {
        yield return new WaitForSeconds(3);
        o.SetActive(true);
    }
}
```
