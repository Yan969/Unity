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

# Time.deltaTime增量时间    
## 10米=(1/60 * 10米/秒) *60  
这样更好理解，帧速率是60，Time.deltaTime是一个1/60切割器，把10米/秒切成60份，然后在一秒钟内执行60次。

[转载：增量时间详解](https://blog.csdn.net/ChinarCSDN/article/details/82914420)
