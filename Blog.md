# Transform
## gameObject.GetComponent<Transform>() 与 transform.GetComponent<T>()  
```gameObject.GetComponent<Transform>()```：获取gameObject的Transform组件  
```transform.GetComponent<T>()```：获取组件transform所在gameObject的T组件  
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
10米 = (1/60 * 10米/秒) *60  
这样更好理解，帧速率是60，```Time.deltaTime```是一个1/60切割器，把10米/秒切成60份，然后在一秒钟内执行60次。

[增量时间详解：https://blog.csdn.net/ChinarCSDN/article/details/82914420](https://blog.csdn.net/ChinarCSDN/article/details/82914420)  
## Time.timeScale 时间缩放因子 
缩放正在流逝的时间。可以用于慢动作效果。  
timeScale = 1.0 : 时间流逝的和真实时间（realtime）一样快；  
timeScale = 0.5 : 时间流逝比真实时间慢2倍；  
timeScale = 0   : 与帧速率(frame rate)相关的功能都会暂停。  
【**建议**】 如果降低timeScale，建议也将```Time.fixedDeltaTime```降低相同的量。  
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
```Simulate(float t, bool withChildren, bool restart, bool fixedTimeStep);``` 模拟粒子发射，既可以模拟某个时间点的粒子效果，也可以模拟粒子整个发射过程。  
当参数restart设置为true时，模拟某个时间点的粒子效果，此时时间t应该为某一时间点，例```Simulate(1, false, true，restart, false);```表示模拟粒子1s时的效果；当参数为false时，t应该为deltaTime，意为某一时间增量，此时simulate函数可以在Update或者FixUpdate里每帧调用，模拟整个发射过程。  
【**倍速播放粒子效果**】让粒子以某倍速factor前进，实现快进或慢进，可以在Update里调用```Simulete（deltaTime * factor，false，false，false）```。  
[转载自Burning：https://zhuanlan.zhihu.com/p/96086877](https://zhuanlan.zhihu.com/p/96086877)  
【**不受缩放因子（Time.timeScale）的影响**】Unity中默认的动画Animation、Animator、粒子特效ParticleSystem等和时间有关的都会受到时间缩放因子（Time.timeScale）的影响。但有时候我们希望它们不受Time.timeScale影响：比如在实现一个类型“子弹时间”的效果时，UI特效是正常播放的, 实现如下：  
1. Particle System: ParticleSystem类似Animation需要在Update中用真实的游戏时间做动画采样  
```c#
public class RealParticleSystem : MonoBehaviour
{
    // Field
    private ParticleSystem mParticle;
 
    // Method
    private void Awake()
    {
        if (mParticle == null)
            mParticle = gameObject.GetComponent<ParticleSystem>();
    }
 
    private void Update()
    {
        if (mParticle == null)
            return;
        mParticle.Simulate(Time.unscaledDeltaTime, true, false);
    }
}
```  
2. Animation: Animation需要在Update中用真实的游戏时间做动画采样  
```c#
[RequireComponent(typeof(Animation))]
public class RealAnimation : MonoBehaviour
{
    // Field
    private Animation mAnim;
    private float time = 0;
 
    // Method
    private void Awake()
    {
        if (mAnim == null)
            mAnim = gameObject.GetComponent<Animation>();
    }
 
    private void Update()
    {
        if (mAnim == null)
            return;
        time += Time.unscaledDeltaTime;
        foreach (AnimationState anim in mAnim)
        {
            if (mAnim.IsPlaying(anim.name))
                anim.normalizedTime = time / anim.length;
        }
        mAnim.Sample();
    }
}
```
3. Animator: Animator比较简单，Unity原生支持，只需更改Animator.updateMode即可  
```c#
mAnimator.updateMode = AnimatorUpdateMode.UnscaledTime;
```  
[转自cube454517408：https://blog.csdn.net/cube454517408/article/details/107563746](https://blog.csdn.net/cube454517408/article/details/107563746)

# Timeline 
【**倒放**】可以通过PlayableDirector的time属性，从大到小，即可实现倒放  
```c#
using UnityEngine;
using UnityEngine.Playables;

public class Runner: MonoBehaviour
{
    public PlayableDirector timelinePlayer;
    private double m_timer;

    void Start()
    {
    	// 获取总时长
        m_timer = timelinePlayer.duration;
        timelinePlayer.Play();
    }

    void Update()
    {
        // 倒放
        if (m_timer > 0)
            m_timer -= Time.deltaTime;
        timelinePlayer.time = m_timer;
    }
}
``` 
[转载自林新发：https://blog.csdn.net/linxinfa/article/details/108374878](https://blog.csdn.net/linxinfa/article/details/108374878)

