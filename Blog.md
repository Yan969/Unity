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
# RectTransform
[详解：https://blog.csdn.net/jmu201521121014/article/details/105725175](https://blog.csdn.net/jmu201521121014/article/details/105725175)  


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

[timeline学习干货：https://www.jianshu.com/p/527e74eb59ca](https://www.jianshu.com/p/527e74eb59ca)

# Playable
[Playable API：定制你的动画系统 简单使用](https://blog.csdn.net/wangjiangrong/article/details/105630666)
[Playable API：定制你的动画系统 简单使用](https://www.it610.com/article/1296811372927066112.htm)
[官方使用例子](https://docs.unity3d.com/2017.2/Documentation/Manual/Playables-Examples.html)
[官方教程](https://mp.weixin.qq.com/s?__biz=MzU5MjQ1NTEwOA==&mid=2247493316&idx=1&sn=7e4fef834a8066faca3d2f1f1a090bb4&chksm=fe1dd26fc96a5b79856840f556cf65026facb83520ac1891605e42d5e777d30a0d5219060e21&mpshare=1&scene=1&srcid=0606YJLYnfprk9UjpPQCnre1#rd)
[可视化工具]()  
# Audio
## AudioClip
**编辑模式下利用反射播放AudioClip**
```c#
using UnityEngine;
using UnityEditor;
using System;
using System.Reflection;
 
public static class EditorSFX
{
 
    public static void PlayClip(AudioClip clip, int startSample = 0, bool loop = false)
    {
        Assembly unityEditorAssembly = typeof(AudioImporter).Assembly;
     
        Type audioUtilClass = unityEditorAssembly.GetType("UnityEditor.AudioUtil");
        MethodInfo method = audioUtilClass.GetMethod(
            "PlayPreviewClip",
            BindingFlags.Static | BindingFlags.Public,
            null,
            new Type[] { typeof(AudioClip), typeof(int), typeof(bool) },
            null
        );
 
        Debug.Log(method);
        method.Invoke(
            null,
            new object[] { clip, startSample, loop }
        );
    }
 
    public static void StopAllClips()
    {
        Assembly unityEditorAssembly = typeof(AudioImporter).Assembly;
 
        Type audioUtilClass = unityEditorAssembly.GetType("UnityEditor.AudioUtil");
        MethodInfo method = audioUtilClass.GetMethod(
            "StopAllPreviewClips",
            BindingFlags.Static | BindingFlags.Public,
            null,
            new Type[] { },
            null
        );
 
        Debug.Log(method);
        method.Invoke(
            null,
            new object[] { }
        );
    }
}
```  
**【扩展】**  
从某个采样/时间点开始播放音频  
```c#
sampleStart = (int)Math.Ceiling (audioClip.samples * ((__sequence.timeCurrent - node.startTime) / audioClip.length));//do your calculation
                     
AudioUtilW.PlayClip (audioClip, 0, node.loop);//startSample doesn't work in this function????
AudioUtilW.SetClipSamplePosition (audioClip, sampleStart);  
``` 
或者  
```c#
 AudioUtility.PlayClip(selectedAudioClip);
 //Get desired position as percentage
 var desiredClipPercentagePos = desiredTime / selectedAudioClip.length;
 //Get the number of samples:
 var samples = AudioUtility.GetSampleCount(selectedAudioClip);
 //Then, simply multiply your percentage against that:
 int playFrom = Mathf.FloorToInt(samples * desiredClipPercentagePos);
 AudioUtility.SetClipSamplePosition(selectedAudioClip, playFrom);
```  
**【Example】**  
I want to play at a position that's 50% of the way through the clip:  
First, get the number of samples:  
```c#
var samples = AudioUtility.GetSampleCount(source.clip);
```  
Then, simply multiply your percentage against that:  
```c#
int playFrom = Mathf.FloorToInt(samples * 0.5f);
```  
Then, use it together:
```c#
AudioUtility.PlayClip(source.clip, playFrom, false);
AudioUtility.SetClipSamplePosition(source.clip, playFrom);
```  

[Way to play audio in editor using an editor script?](https://forum.unity.com/threads/way-to-play-audio-in-editor-using-an-editor-script.132042/)  
[How to Play AudioClip from Editor from a Start Sample/Time](https://answers.unity.com/questions/844896/how-to-play-audioclip-from-editor-from-a-start-sam.html?_gl=1*czio5y*_ga*MzQ1MTQ5MjE0LjE2MTg0NTY3ODM.*_ga_1S78EFL1W5*MTYyMjYzMDUwMi4zLjEuMTYyMjYzMTc1NC4w&_ga=2.116201422.97339729.1622515344-345149214.1618456783#)  
[Reflected AudioUtil class for making audio based Editor Extensions](https://forum.unity.com/threads/reflected-audioutil-class-for-making-audio-based-editor-extensions.308133/)  
[自定义参考：https://www.programmersought.com/article/65214557718/](https://www.programmersought.com/article/65214557718/)  
[自定义：https://blog.unity.com/technology/extending-timeline-a-practical-guide](https://blog.unity.com/technology/extending-timeline-a-practical-guide)  
[MarkerEditor](https://docs.unity3d.com/Packages/com.unity.timeline@1.2/api/UnityEditor.Timeline.TrackEditor.html)

# Camera
水平翻转Camera（屏幕镜像效果）可以使用```camera.projectionMatrix ```自定义投影矩阵缩放x轴  
``` C#
 using UnityEngine;
 [RequireComponent(typeof(Camera))]
 [ExecuteInEditMode]
 public class MirrorFlipCamera : MonoBehaviour {
     new Camera camera;
     public bool flipHorizontal;
     void Awake () {
         camera = GetComponent<Camera>();
     }
     void OnPreCull() {
         camera.ResetWorldToCameraMatrix();
         camera.ResetProjectionMatrix();
         Vector3 scale = new Vector3(flipHorizontal ? -1 : 1, 1, 1);
         camera.projectionMatrix = camera.projectionMatrix * Matrix4x4.Scale(scale);
     }
     void OnPreRender () {
         GL.invertCulling = flipHorizontal;   //没有这行，会出现Culling（剔除）的问题
     }
     
     void OnPostRender () {
         GL.invertCulling = false;
     }
 }
```  
