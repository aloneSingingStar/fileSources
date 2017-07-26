# dagger2基本原理
---
## 1.实现步骤
### 1.使用@Inject标注需要注入的依赖,比如TasksActivity中依赖TasksPresenter
```
@Inject TasksPresenter mTasksPresenter;
```
build之后，会在build/generated/source/apt/ 目录下生成文件：目标类_MembersInjector.java类，比如这里生成的是TasksActivity_MembersInjector.java.

### 2.创建所依赖对象的实例，有Inject构造函数、@Module两种方式。
* 2.1使用@Inject标注构造函数

  会在bulid/generated/source/apt目录下生成文件：依赖对象_Factory.java类，比如这个生成的是TasksPresenter_Factory.java.

* 2.2使用@Module配合@Provides提供依赖实例。

  会在build/generated/source/apt目录下生成文件：依赖对象Module_Provide提供的实例Factory.java

  比如如下Module，会生成TasksPresenterModule_ProvideTasksContractViewFactory.java类，**如果提供多个依赖实例，即一个Module中有多个@Provide,就会生成对应个数的JAVA类，每个生成的类的格式和上面格式一样**
```
@Module
public class TasksPresenterModule {

    private final TasksContract.View mView;

    public TasksPresenterModule(TasksContract.View view) {
        mView = view;
    }

    @Provides
    TasksContract.View provideTasksContractView() {
        return mView;
    }

}
```

* 2.3 @Inject和@Module的区别
  
  有以下几个场景无法使用@Inject，需要使用@Module配合@Provide来提供实例  
  * 接口没有构造函数
  * 第三方库的类不能被标注
  * 构造函数中的参数必须配置
  
* 2.4 @Inject配合@Module使用

  同时使用Module和@Inject构造，优先从Module中寻找依赖实例，Module中没找到，就从@Inject构造生成的工厂类中找。

  应用场景，如下代码，TasksPresenter构造中需要传入TasksRepository和TasksContract.View,所以在获取TasksPresenter实例之前，要先获取TasksContract.View和TasksRepository的实例，所以可以先在Module中先提供需要的参数实例。然后在@Inject构造中获取TasksPresenter实例之前能拿到所需参数的实例。


```
@Inject
    TasksPresenter(TasksRepository tasksRepository, TasksContract.View tasksView) {
        mTasksRepository = tasksRepository;
        mTasksView = tasksView;
    }

```

### 3.使用@Component标注**接口或者抽象类**，用于完成依赖注入。

  会在build/generated/source/apt目录下生成文件：Dagger[对应的Componet].java

  比如如下Component,会生成DaggerTasksComponent.java类。

```
@FragmentScoped
@Component(dependencies = TasksRepositoryComponent.class, modules = TasksPresenterModule.class)
public interface TasksComponent {
	
    void inject(TasksActivity activity);
}
```

## 2.三者之间的联系
### 1.@Inject构造函数与@Module不关联使用的情况

![@Inject构造函数与@Module不关联使用的情况](https://github.com/aloneSingingStar/fileSources/blob/master/images/依赖注入过程_Inject与Module不关联使用.png)

### 2.@Inject构造函数与@Module关联使用的情况

![@Inject构造函数与@Module关联使用的情况](https://github.com/aloneSingingStar/fileSources/blob/master/images/依赖注入过程_Module与@Inject同时使用.png)

![@Inject构造函数与@Module关联使用的情况](https://github.com/aloneSingingStar/fileSources/blob/master/images/Inject与Module同时使用时依赖注入的过程.png)