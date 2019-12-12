![][1]
aop植入
https://www.cnblogs.com/xiaoxing/p/10270285.html

|  表头   | 表头  |
|  ----  | ----  |
| @PostConstuctor  | InitDestroyAnnotationBeanPostProcessor |
| InitializingBean  | org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeInitMethods |


```
o.sf.c.s.AbstractApplicationContext#refresh
	o.sf.c.s.AbstractApplicationContext#finishBeanFactoryInitialization
		o.sf.b.f.c.ConfigurableListableBeanFactory#preInstantiateSingletons
			o.sf.b.f.s.AbstractBeanFactory#getBean(java.lang.String)
				o.sf.b.f.s.AbstractBeanFactory#doGetBean
					o.sf.b.f.s.DefaultSingletonBeanRegistry#getSingleton(java.lang.String)
													
```

```
-- 9次后置处理器
【1】InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation(); getBeanPostProcessors().postProcessAfterInitialization
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInstantiation
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
【2】SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors()
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
				o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBeanInstance
					o.sf.b.f.s.AbstractAutowireCapableBeanFactory#determineConstructorsFromBeanPostProcessors
		 					SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors()
【3】 MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#applyMergedBeanDefinitionPostProcessors			 					
【4】SmartInstantiationAwareBeanPostProcessor.getEarlyBeanReference 解决循环引用
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#getEarlyBeanReference
				SmartInstantiationAwareBeanPostProcessor.getEarlyBeanReference
【5】InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#populateBean
				o.sf.b.f.c.InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
				
【6】InstantiationAwareBeanPostProcessor.postProcessPropertyValues
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#populateBean
				o.sf.b.f.c.InstantiationAwareBeanPostProcessor#postProcessPropertyValues
【7】getBeanPostProcessors().postProcessBeforeInitialization
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
				o.sf.b.f.s.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization
					getBeanPostProcessors().postProcessBeforeInitialization
					
【8】getBeanPostProcessors().postProcessAfterInitialization
	o.sf.b.f.s.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
		o.sf.b.f.s.AbstractAutowireCapableBeanFactory#doCreateBean
			o.sf.b.f.s.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
				o.sf.b.f.s.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
					getBeanPostProcessors().postProcessAfterInitialization
【9】	销毁 DestructionAwareBeanPostProcessor.postProcessBeforeDestruction
```

[1]: ../images/spring/lifesycle.jpg