# Spring IOC 源码解析

```plain
 　       1）、传入配置类，创建ioc容器
流程：
          2）、注册配置类，调用refresh（）刷新容器；
          3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
              1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
              2）、给容器中加别的BeanPostProcessor
              3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
              4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
              5）、注册没实现优先级接口的BeanPostProcessor；
              6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
                  创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
                  1）、创建Bean的实例
                  2）、populateBean；给bean的各种属性赋值
                  3）、initializeBean：初始化bean；
                          1）、invokeAwareMethods()：处理Aware接口的方法回调
                          2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
                          3）、invokeInitMethods()；执行自定义的初始化方法
                          4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
                  4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
              7）、把BeanPostProcessor注册到BeanFactory中；
                  beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
  
              AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
          4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
              1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
                  getBean->doGetBean()->getSingleton()->
              2）、创建bean
                  【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，　　　　　　　　　　　　会调用postProcessBeforeInstantiation()】
                  1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
                      只要创建好的Bean都会被缓存起来
                  2）、createBean（）;创建bean；
                      AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
                      【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
                      【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
                      1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
                          希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
                          1）、后置处理器先尝试返回对象；
                              bean = applyBeanPostProcessorsBeforeInstantiation（）：
                                  拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
                                  就执行postProcessBeforeInstantiation
                              if (bean != null) {
                                bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                            }
  
                      2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
                      3）、
```