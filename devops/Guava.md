Guava 专辑

## cache

### 单独用
```
  private final Cache<String, String> cache1 = CacheBuilder.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000)
            .build();

    public String get(String key) throws ExecutionException {
        return cache1.get(key, ()->{
            //query from db;
            return "empty-"+key+" val";
        });
    }

```
### spring-cache集成使用