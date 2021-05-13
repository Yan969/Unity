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
