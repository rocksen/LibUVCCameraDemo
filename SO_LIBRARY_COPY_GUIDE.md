# SO库自动拷贝功能使用指南

## 功能概述
自动将编译生成的so库文件拷贝到 `src/main/libs` 目录下对应的CPU架构子目录中，方便后续的手动分发和管理。

## 自动触发场景

### 1. 发布版本编译后自动拷贝
```bash
# 编译发布版本时会自动拷贝
.\gradlew :libuvccamera:assembleRelease

# 或者仅编译原生代码
.\gradlew :libuvccamera:externalNativeBuildRelease
```

### 2. 发布版本AAR打包时拷贝
```bash
# 打包AAR时会确保先拷贝so库
.\gradlew :libuvccamera:bundleReleaseAar
```

## 手动执行拷贝

### 单独执行拷贝任务
```bash
# 手动执行拷贝任务（需要先编译原生代码）
.\gradlew :libuvccamera:copyNativeLibsToJniLibs
```

### 编译+拷贝组合
```bash
# 先编译再拷贝
.\gradlew :libuvccamera:externalNativeBuildRelease :libuvccamera:copyNativeLibsToJniLibs
```

## 目录结构

### 源目录（编译生成）
```
libuvccamera/build/intermediates/ndkBuild/release/obj/local/
├── armeabi-v7a/
│   ├── libUVCCamera.so
│   ├── libjpeg-turbo1500.so
│   ├── libusb100.so
│   └── libuvc.so
├── arm64-v8a/
├── x86/
└── x86_64/
```

### 目标目录（拷贝到此）
```
libuvccamera/src/main/libs/
├── armeabi-v7a/
│   ├── libUVCCamera.so
│   ├── libjpeg-turbo1500.so
│   ├── libusb100.so
│   └── libuvc.so
├── arm64-v8a/
├── x86/
└── x86_64/
```

## 功能特性

### ✅ 自动化特性
- **自动清理**: 拷贝前自动清理目标目录中的旧.so文件
- **目录创建**: 自动创建必要的架构目录
- **架构过滤**: 只拷贝配置的CPU架构 (armeabi-v7a, arm64-v8a, x86, x86_64)
- **文件验证**: 只拷贝.so扩展名的文件

### 📊 详细日志
拷贝过程会显示详细信息：
```
=== 开始拷贝so库文件 ===
源目录: D:\workspace\github\LibUVCCameraDemo\libuvccamera\build\intermediates\ndkBuild\release\obj\local
目标目录: D:\workspace\github\LibUVCCameraDemo\libuvccamera\src\main\libs

处理架构: armeabi-v7a
  已删除旧文件: libUVCCamera.so
  ✓ 拷贝: libUVCCamera.so (大小: 154KB)
  ✓ 拷贝: libjpeg-turbo1500.so (大小: 313KB)
  ✓ 拷贝: libusb100.so (大小: 108KB)
  ✓ 拷贝: libuvc.so (大小: 100KB)
  armeabi-v7a: 拷贝了 4 个文件

=== so库拷贝任务完成！总共拷贝 16 个文件 ===

最终libs目录结构:
  armeabi-v7a: 4个文件 - libUVCCamera.so, libjpeg-turbo1500.so, libusb100.so, libuvc.so
  arm64-v8a: 4个文件 - libUVCCamera.so, libjpeg-turbo1500.so, libusb100.so, libuvc.so
  x86: 4个文件 - libUVCCamera.so, libjpeg-turbo1500.so, libusb100.so, libuvc.so
  x86_64: 4个文件 - libUVCCamera.so, libjpeg-turbo1500.so, libusb100.so, libuvc.so
```

### ⚠️ 错误处理
- 源目录不存在时会显示警告但不会中断构建
- 目标目录权限问题会显示具体错误信息
- 单个文件拷贝失败不会影响其他文件

## 配置选项

### 启用Debug版本自动拷贝
如需要Debug版本也自动拷贝，可以在 `build.gradle` 中取消注释：

```gradle
// 在 afterEvaluate 块中取消注释这些行：
if (task.name == 'externalNativeBuildDebug') {
    task.finalizedBy copyNativeLibsToJniLibs
}
```

### 修改支持的架构
在 `copyNativeLibsToJniLibs` 任务中修改 `abis` 列表：

```gradle
// 当前支持的架构
def abis = ['armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64']

// 如果只需要ARM架构：
// def abis = ['armeabi-v7a', 'arm64-v8a']
```

### 自定义目标目录
修改 `targetDir` 变量：

```gradle
// 默认目标目录
def targetDir = "$projectDir/src/main/libs"

// 自定义目标目录，例如：
// def targetDir = "$projectDir/native-libs"
```

## 常见问题

### Q: 拷贝任务没有执行？
**A**: 确保先编译了Release版本的原生代码：
```bash
.\gradlew :libuvccamera:externalNativeBuildRelease
```

### Q: 提示源目录不存在？
**A**: 确保原生代码编译成功，检查构建日志是否有编译错误。

### Q: 如何只拷贝特定架构？
**A**: 修改 `build.gradle` 中的 `abis` 列表，只保留需要的架构。

### Q: 可以拷贝到项目外的目录吗？
**A**: 可以，修改 `targetDir` 为绝对路径，但需要确保有写入权限。

## 扩展用法

### 集成到CI/CD
```bash
# 在CI脚本中使用
./gradlew clean :libuvccamera:assembleRelease
# so库会自动拷贝到libs目录，可以直接打包分发
```

### 与其他任务结合
```gradle
// 自定义任务示例
task packageNativeLibs(type: Zip) {
    dependsOn copyNativeLibsToJniLibs
    from "$projectDir/src/main/libs"
    archiveFileName = "native-libs-${project.version}.zip"
    destinationDirectory = file("$buildDir/outputs")
}
```

---
📅 **更新时间**: 2025年9月22日  
🔧 **兼容版本**: Gradle 6.7.1 + Android Gradle Plugin 4.2.0