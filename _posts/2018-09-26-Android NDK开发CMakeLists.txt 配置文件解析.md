### Android NDK开发CMakeLists.txt 配置文件解析

在`Android Studio`中建立支持`NDK`项目是默认是`CMake`的方式来进行构建的，过去的`ndk-build`的方式从目前官方的文档上看，应该是逐渐淘汰的。现在大多数`APP`架构中都是有`Native Code`的，所以了解`CMake`的配置方式是进行`NDK`开发的基础。

#### 0x00 建立支持NDK项目

`Android Studio` 创建`NDK`项目是相当简单的，在项目创建向导页面中勾选支持 `include C++ support`，然后工程目录中就自动生成一个`CMakeLists.txt`文件。`gralde`文件中也会生成相应的`ndk`配置。

#### 0x01 Gradle 配置解析

```java
android {
  ...
  defaultConfig {...}

  // Encapsulates your external native build configurations.
  externalNativeBuild {
    // Encapsulates your CMake build configurations.
    cmake {
      // Provides a relative path to your CMake build script.
      path "CMakeLists.txt"
    }
  }
}
```

在模块中的`build.gradle`文件的`android{}`块中的 `externalNativeBuild`指定了 `CMakeLists` 文件的相对路径，这里的`CMakeLists`文件是在工程目录所以直接写文件名称。

此外还可以在` defaultConfig`块再指定一个`externalNativeBuild`块，指定一些通用的配置。

```java
android {
    ...
    defaultConfig {
        ...
        externalNativeBuild {
            // 这里可以配置一些通用的配置
            cmake {
                // 传递可选的参数到cmake中
                arguments "-DANDROID_ARM_NEON=TRUE", "-DANDROID_TOOLCHAIN=clang"
                cppFlags "-frtti -fexceptions"
                // Gradle 会构建这些ABI配置,但打包时仅把 ndk 块中的 ABI 打包进apk
                abiFilters 'x86', 'x86_64', 'armeabi', 'armeabi-v7a','arm64-v8a'
            }
        }
        ndk {
      		// 打包时仅仅会把此块中的 ABI 打包进apk
      		abiFilters 'armeabi-v7a','arm64-v8a'
    	}
    }
    externalNativeBuild {...}
}
```

#### 0x02 CMakeLists文件配置说明

`CMakeLists` 文件需要将`cmake_minimum_required`和`add_library` 添加进来。

```java
# 指定cmake最低的版本号
cmake_minimum_required(VERSION 3.4.1)

# 指定库的名称，还要指定编译的库是静态库（STATIC）还是共享库（SHARED）
# 然后提供源码文件的相对路径。如果有多个库需要编译，则使用多个add_library方法进行添加。
add_library( # Specifies the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )
```

通常还有源码头文件需要添加，所以还要指定头文件目录

```java
add_library(...)

# Specifies a path to native header files.
include_directories(src/main/cpp/include/)
```

CMake 使用以下规范来为库文件命名：

```java
lib库名称.so
```

所以在上面配置了库的名称为`native-lib`，那么`gradle` 将生成`libnative-lib.so`文件，在加载时使用

```java
static {
    System.loadLibrary(“native-lib”);
}
```

##### 添加NDK内置的库

使用`find_library`可以引用`NDK`内部的库，相当于可以使用其他第三方的API的能力

```java
find_library( # 指定库的名称
              log-lib

              # 指定 NDK 库中的名称，cmake 可以搜索到
              log )
# 把引用的库和我们自己要编译的链接起来，我们的库才可以使用
target_link_libraries( # 指定哪个库要使用
                       native-lib

                       # 把log库和我们的库链接起来.
                       ${log-lib} )
```

##### 添加 NDK 中的源文件

例如添加 `NDK中`的 `android_native_app_glue.c` 文件，然后关联到我们的 `nativ-lib`

```java
add_library( app-glue
             STATIC
             ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c )

# You need to link static libraries against your shared native library.
target_link_libraries( native-lib app-glue ${log-lib} )
```

##### 添加预编译库

```
add_library( imported-lib
             SHARED
             IMPORTED )

set_target_properties( # Specifies the target library.
                       imported-lib

                       # Specifies the parameter you want to define.
                       PROPERTIES IMPORTED_LOCATION

                       # Provides the path to the library you want to import.
                       imported-lib/src/${ANDROID_ABI}/libimported-lib.so )
```

同样地，可能还需要指定头文件

```java
include_directories( imported-lib/include/ )
```

#### 0x03 引用

https://developer.android.com/studio/projects/add-native-code