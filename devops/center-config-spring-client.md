配置中心有非常多选择，下面这篇文章不是介绍配置中心，而是介绍如何自己造轮子，写一个简易的client，无侵入的从配置中心拉去配置。


```
public class CenterConfigClient extends PropertyPlaceholderConfigurer implements EnvironmentAware {
    private Environment environment;

    @Override
    protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess, Properties props)
            throws BeansException {

        props.put("config.value", ">||>> " +props.get("config.value") );

        super.processProperties(beanFactoryToProcess, props);

    }

    @Override
    protected void loadProperties(Properties props) throws IOException {

        super.loadProperties(props);

        if ("dev".equalsIgnoreCase(this.environment.getActiveProfiles()[0])) {
            Properties p = new Properties();
            p.setProperty("config.value1", "dev从 配置中心读取-" + StringUtils.trimArrayElements(this.environment.getActiveProfiles()));
            props.putAll(p);
            props.put("config.value", props.get("config.value") + "> dev配置中心修改后");
        }else if("default".equalsIgnoreCase(this.environment.getActiveProfiles()[0])){
            Properties p = new Properties();
            p.setProperty("config.value1", "default从 配置中心读取-" + StringUtils.trimArrayElements(this.environment.getActiveProfiles()));
            props.putAll(p);
            props.put("config.value", props.get("config.value") + "> default配置中心修改后");
        }

    }

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

}
```
核心代码如上所示：原理如下 ，[源码][1]
* 继承 PropertyPlaceholderConfigurer，实现loadProperties方法，来加载配置。
* 你也可以实现 processProperties，在注入属性的时候，做一些特殊化处理。

[1]: https://github.com/moheqionglin/spring-demo/tree/master/devops/devops-service/src/main/java/com/moheqionglin/centerconfig/client