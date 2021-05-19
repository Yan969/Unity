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
## Time.deltaTime 增量时间  
10米=(1/60 * 10米/秒) *60  
这样更好理解，帧速率是60，Time.deltaTime是一个1/60切割器，把10米/秒切成60份，然后在一秒钟内执行60次。

[转载：增量时间详解](https://blog.csdn.net/ChinarCSDN/article/details/82914420)  
## Time.timeScale 时间缩放因子 
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
# Particle System  
## ParticleSystem.Simulate  
Simulate(float t, bool withChildren, bool restart, bool fixedTimeStep); 模拟粒子发射，既可以模拟某个时间点的粒子效果，也可以模拟粒子整个发射过程。  
当参数restart设置为true时，模拟某个时间点的粒子效果，此时时间t应该为某一时间点，例Simulate(1, false, true，restart, false);表示模拟粒子1s时的效果；当参数为false时，t应该为deltaTime，意为某一时间增量，此时simulate函数可以在Update或者FixUpdate里每帧调用，模拟整个发射过程。  
**【倍速播放粒子效果】**让粒子以某倍速factor前进，实现快进或慢进，可以在Update里调用Simulete（deltaTime * factor，false，false，false）  

[转载自Burning：https://zhuanlan.zhihu.com/p/96086877](https://zhuanlan.zhihu.com/p/96086877)

