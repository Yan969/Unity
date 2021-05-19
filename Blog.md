# Transform
## gameObject.GetComponent<Transform>() 与 transform.GetComponent<T>()  
gameObject.GetComponent<Transform>()：获取gameObject的Transform组件  
transform.GetComponent<T>()：获取组件transform所在gameObject的T组件  
```c#
  static void GetControllableParticleSystems(Transform t, ICollection<ParticleSystem> roots, HashSet<ParticleSystem> subEmitters)
        {
            var ps = t.GetComponent<ParticleSystem>();    //获取组件transform所在gameObject的ParticleSystem组件
            if (ps != null)
            {
                if (!subEmitters.Contains(ps))
                {
                    roots.Add(ps);
                    CacheSubEmitters(ps, subEmitters);
                }
            }

            for (int i = 0; i < t.childCount; ++i)
            {
                GetControllableParticleSystems(t.GetChild(i), roots, subEmitters);
            }
        }
```  

# Time  
## Time.deltaTime增量时间  
10米=(1/60 * 10米/秒) *60  
这样更好理解，帧速率是60，Time.deltaTime是一个1/60切割器，把10米/秒切成60份，然后在一秒钟内执行60次。

[转载：增量时间详解](https://blog.csdn.net/ChinarCSDN/article/details/82914420)  
## Time.timeScale  
缩放正在流逝的时间。可以用于慢动作效果。  
timeScale = 1.0 : 时间流逝的和真实时间（realtime）一样快；  
timeScale = 0.5 : 时间流逝比真实时间慢2倍；  
timeScale = 0   : 与帧速率(frame rate)相关的功能都会暂停。  
**【建议】** 如果降低timeScale，建议也将Time.fixedDeltaTime降低相同的量。  
```c#
using UnityEngine;

public class Example : MonoBehaviour
{
    // Toggles the time scale between 1 and 0.7
    // whenever the user hits the Fire1 button.

    void Update()
    {
        if (Input.GetButtonDown("Fire1"))
        {
            if (Time.timeScale == 1.0f)
                Time.timeScale = 0.7f;
            else
                Time.timeScale = 1.0f;
            // Adjust fixed delta time according to timescale
            // The fixed delta time will now be 0.02 frames per real-time second
            Time.fixedDeltaTime = 0.02f * Time.timeScale;
        }
    }
}
```

