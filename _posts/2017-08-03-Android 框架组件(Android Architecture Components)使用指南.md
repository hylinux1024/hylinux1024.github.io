---
layout: post
title: Android 框架组件(Android Architecture Components)使用指南
categories: 模块化
description: 面对越来越复杂的 App 需求，Google 官方发布了Android 框架组件库（Android Architecture Components ）。为开发者更好的开发 App 提供了非常好的样本。
keywords: 模块化,架构
---

面对越来越复杂的 App 需求，Google 官方发布了Android 框架组件库（Android Architecture Components ）。为开发者更好的开发 App 提供了非常好的样本。这个框架里的组件是配合 Android 组件生命周期的，所以它能够很好的规避组件生命周期管理的问题。今天我们就来看看这个库的使用。

#### 0x00 通用的框架准则

官方建议在架构 App 的时候遵循以下两个准则：

1. **关注分离**

   其中早期开发 App 最常见的做法是在 Activity 或者 Fragment 中写了大量的逻辑代码，导致 Activity 或 Fragment 中的代码很臃肿，十分不易维护。现在很多 App 开发者都注意到了这个问题，所以前两年 MVP 结构就非常有市场，目前普及率也很高。

2. **模型驱动UI**

   模型持久化的好处就是：即使系统回收了 App 的资源用户也不会丢失数据，而且在网络不稳定的情况下 App 依然可以正常地运行。从而保证了 App 的用户体验。


#### 0x01 App 框架组件

框架提供了以下几个核心组件，我们将通过一个实例来说明这几个组件的使用。

- ViewModel
- LiveData
- Room

假设要实现一个用户信息展示页面。这个用户信息是通过REST API 从后台获取的。

#### 0x02 建立UI

我们使用 fragment (UserProfileFragment.java) 来实现用户信息的展示页面。为了驱动 UI，我们的数据模型需要持有以下两个数据元素

- **用户ID**: 用户的唯一标识。可以通过 fragment 的 arguments 参数进行传递这个信息。这样做的好处就是如果系统销毁了应用，这个参数会被保存并且下次重新启动时可以恢复之前的数据。
- **用户对象数据**：POJO 持有用户数据。

我们要创建 **ViewModel** 对象用于保存以上数据。

那什么是 ViewModel 呢？

>A **ViewModel** provides the data for a specific UI component, such as a fragment or activity, and handles the communication with the business part of data handling, such as calling other components to load the data or forwarding user modifications. The ViewModel does not know about the View and is not affected by configuration changes such as recreating an activity due to rotation.

ViewModel 是一个框架组件。它为 UI 组件 (fragment或activity) 提供数据，并且可以调用其它组件加载数据或者转发用户指令。ViewModel 不会关心 UI 长什么样，也不会受到 UI 组件配置改变的影响，例如不会受旋转屏幕后 activity 重新启动的影响。因此它是一个与 UI 组件无关的。

```java
public class UserProfileViewModel extends ViewModel {
    private String userId;
    private User user;

    public void init(String userId) {
        this.userId = userId;
    }
    public User getUser() {
        return user;
    }
}
```

```java
public class UserProfileFragment extends LifecycleFragment {
    private static final String UID_KEY = "uid";
    private UserProfileViewModel viewModel;

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        String userId = getArguments().getString(UID_KEY);
        viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
        viewModel.init(userId);
    }

    @Override
    public View onCreateView(LayoutInflater inflater,
                @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.user_profile, container, false);
    }
}
```

需要的是：由于框架组件目前还处于预览版本，这里`UserProfileFragment` 是继承于 `LifecycleFragment` 而不是 `Fragment`。待正式发布版本之后 Android Support 包中的 `Fragment` 就会默认实现 `LifecycleOwner` 接口。而 `LifecycleFragment` 也是实现了 `LifecycleOwner` 接口的。即正式版本发布时 Support 包中的 UI 组件类就是支持框架组件的。

现在已经有了 UI 组件和 ViewModel，那么我们如何将它们进行连接呢？这时候就需要用到 LiveData 组件了。

>**LiveData** is an observable data holder. It lets the components in your app observe [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html) objects for changes without creating explicit and rigid dependency paths between them. LiveData also respects the lifecycle state of your app components (activities, fragments, services) and does the right thing to prevent object leaking so that your app does not consume more memory.

LiveData 的使用有点像 RxJava。因此完全可以使用 RxJava 来替代 LiveData 组件。

现在我们修改一下 `UserProfileViewModel` 类

```java
public class UserProfileViewModel extends ViewModel {
    ...
    private LiveData<User> user;
    public LiveData<User> getUser() {
        return user;
    }
}
```

将 `User user` 替换成  `LiveData<User> user`

然后再修改 `UserProfileFragment` 类中

```java
@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    viewModel.getUser().observe(this, user -> {
      // update UI
    });
}
```

当用户数据发生改变时，就会通知 UI 进行更新。ViewModel 与 UI 组件的交互就是这么简单。

但细心的朋友可能发现了：fragment 在 `onActivityCreated` 方法中添加了相应的监听，但是没有在其它对应的生命周期中移除监听。有经验的朋友就会觉得这是不是有可能会发生引用泄露问题呢？其实不然，LiveData 组件内部已经为开发者做了这些事情。即 LiveData 会再正确的生命周期进行回调。

#### 0x03 获取数据

现在已经成功的把 ViewModel 与 UI 组件（fragment）进行了通信。那么 ViewModel 又是如何获取数据的呢？

假设我们的数据是通过REST API 从后天获取的。我们使用 [Retrofit](http://square.github.io/retrofit/) 库实现网络请求。

以下是请求网络接口 `Webservice` 

```java
public interface Webservice {
    /**
     * @GET declares an HTTP GET request
     * @Path("user") annotation on the userId parameter marks it as a
     * replacement for the {user} placeholder in the @GET path
     */
    @GET("/users/{user}")
    Call<User> getUser(@Path("user") String userId);
}
```

ViewModel 可以引用 `Webservice` 接口，但是这样做违背了我们在上文提到的**关注分离**准则。因为我们推荐使用 `Repository` 模型对 `Webservice` 进行封装。

>**Repository** modules are responsible for handling data operations. They provide a clean API to the rest of the app. They know where to get the data from and what API calls to make when data is updated. You can consider them as mediators between different data sources (persistent model, web service, cache, etc.).

关于 Repository 模式可以参考我的上一篇《App 组件化/模块化之路——Repository模式》

以下是使用 Repository 封装 `WebService` 

```java
public class UserRepository {
    private Webservice webservice;
    // ...
    public LiveData<User> getUser(int userId) {
        // This is not an optimal implementation, we'll fix it below
        final MutableLiveData<User> data = new MutableLiveData<>();
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                // error case is left out for brevity
                data.setValue(response.body());
            }
        });
        return data;
    }
}
```

使用 Respository 模式抽象数据源接口，也可以很方便地替换其它数据。这样 ViewModel 也不用知道数据源到底是来自哪里。

#### 0x04 组件间的依赖管理

从上文我们知道 `UserRepository` 类需要有一个 `WebService` 实例才能工作。我们可以直接创建它，但这么做我们就必须知道它的依赖，而且会由很多重复的创建对象的代码。这时候我们可以使用依赖注入。本例中我们将使用 Dagger 2 来管理依赖。

#### 0x05 连接 ViewModel 和 Repository

修改 `UserProfileViewModel` 类，引用 Repository 并且通过 Dagger 2 对 Repository 的依赖进行管理。

```java
public class UserProfileViewModel extends ViewModel {
    private LiveData<User> user;
    private UserRepository userRepo;

    @Inject // UserRepository parameter is provided by Dagger 2
    public UserProfileViewModel(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    public void init(String userId) {
        if (this.user != null) {
            // ViewModel is created per Fragment so
            // we know the userId won't change
            return;
        }
        user = userRepo.getUser(userId);
    }
    public LiveData<User> getUser() {
        return this.user;
    }
}
```

#### 0x06 缓存数据

前面我们实现的 Repository 是只有一个网络数据源的。这样做每次进入用户信息页面都需要去查询网络，用户需要等待，体验不好。因此在 Repository 中加一个缓存数据。

```java
@Singleton  // informs Dagger that this class should be constructed once
public class UserRepository {
    private Webservice webservice;
    // simple in memory cache, details omitted for brevity
    private UserCache userCache;
    public LiveData<User> getUser(String userId) {
        LiveData<User> cached = userCache.get(userId);
        if (cached != null) {
            return cached;
        }

        final MutableLiveData<User> data = new MutableLiveData<>();
        userCache.put(userId, data);
        // this is still suboptimal but better than before.
        // a complete implementation must also handle the error cases.
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                data.setValue(response.body());
            }
        });
        return data;
    }
}
```

#### 0x07 持久化数据 （Room 组件）

Android 框架提供了 Room 组件，为 App 数据持久化提供了解决方案。

>**Room** is an object mapping library that provides local data persistence with minimal boilerplate code. At compile time, it validates each query against the schema, so that broken SQL queries result in compile time errors instead of runtime failures. Room abstracts away some of the underlying implementation details of working with raw SQL tables and queries. It also allows observing changes to the database data (including collections and join queries), exposing such changes via *LiveData* objects. In addition, it explicitly defines thread constraints that address common issues such as accessing storage on the main thread.

Room 组件提供了数据库操作，配合 LiveData 使用可以监听数据库的变化，进而更新 UI 组件。

要使用 Room 组件，需要以下步骤：

- 使用注解 `@Entity` 定义实体
- 创建 `RoomDatabase` 子类
- 创建数据访问接口（DAO）
- 在 `RoomDatabase` 中引用 DAO

1. **用注解 `@Entity` 定义实体类**

```java
@Entity
class User {
  @PrimaryKey
  private int id;
  private String name;
  private String lastName;
  // getters and setters for fields
}
```

2. **创建 `RoomDatabase`子类**

```java
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
}
```

需要注意的是 `MyDatabase` 是抽象类，Room 组件为我们提供具体的实现。

3. **创建 DAO**
```java
@Dao
public interface UserDao {
    @Insert(onConflict = REPLACE)
    void save(User user);
    @Query("SELECT * FROM user WHERE id = :userId")
    LiveData<User> load(String userId);
}
```
4. **在 `RoomDatabase` 中引用 DAO**
```java
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```
现在有了 Room 组件，那么我们可以修改 `UserRepository` 类

```java
@Singleton
public class UserRepository {
    private final Webservice webservice;
    private final UserDao userDao;
    private final Executor executor;

    @Inject
    public UserRepository(Webservice webservice, UserDao userDao, Executor executor) {
        this.webservice = webservice;
        this.userDao = userDao;
        this.executor = executor;
    }

    public LiveData<User> getUser(String userId) {
        refreshUser(userId);
        // return a LiveData directly from the database.
        return userDao.load(userId);
    }

    private void refreshUser(final String userId) {
        executor.execute(() -> {
            // running in a background thread
            // check if user was fetched recently
            boolean userExists = userDao.hasUser(FRESH_TIMEOUT);
            if (!userExists) {
                // refresh the data
                Response response = webservice.getUser(userId).execute();
                // TODO check for error etc.
                // Update the database.The LiveData will automatically refresh so
                // we don't need to do anything else here besides updating the database
                userDao.save(response.body());
            }
        });
    }
}
```

目前为止我们的代码就基本完成了。UI 组件通过 ViewModel 访问数据，而 ViewModel 通过 LiveData 监听数据的变化，并且使用 Repository 模式封装数据源。这些数据源可以是网络数据，缓存以及持久化数据。

#### 0x08 框架结构图

![final architecture](../images/final-architecture.png)



#### 0x09 参考文档

https://developer.android.com/topic/libraries/architecture/guide.html#recommended_app_architecture

https://github.com/googlesamples/android-architecture-components