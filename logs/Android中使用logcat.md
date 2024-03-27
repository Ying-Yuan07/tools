# Android中使用logcat

## 使用__android_log_print打印log

**头文件**

`./system/logging/liblog/include/android/log.h:int __android_log_print(int prio, const char* tag, const char* fmt, ...)`

```c++
// ./system/logging/liblog/include/android/log.h
/**
 * Writes a formatted string to the log, with priority `prio` and tag `tag`.
 * The details of formatting are the same as for
 * [printf(3)](http://man7.org/linux/man-pages/man3/printf.3.html).
 */
int __android_log_print(int prio, const char* tag, const char* fmt, ...)
    __attribute__((__format__(printf, 3, 4)));

```

**定义**

`./system/logging/liblog/logger_write.cpp:int __android_log_print(int prio, const char* tag, const char* fmt, ...) {`

```c++
// ./system/logging/liblog/logger_write.cpp
int __android_log_print(int prio, const char* tag, const char* fmt, ...) {
  ErrnoRestorer errno_restorer;

  if (!__android_log_is_loggable(prio, tag, ANDROID_LOG_VERBOSE)) {
    return -EPERM;
  }

  va_list ap;
  __attribute__((uninitialized)) char buf[LOG_BUF_SIZE];

  va_start(ap, fmt);
  vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);
  va_end(ap);

  __android_log_message log_message = {
      sizeof(__android_log_message), LOG_ID_MAIN, prio, tag, nullptr, 0, buf};
  __android_log_write_log_message(&log_message);
  return 1;
}
```





### 使用样例1



#### 定义LOGI

定义时包含`include <android/log.h>`

```c++
//./system/chre/apps/wifi_offload/include/chre/apps/wifi_offload/wifi_offload.h
#include <android/log.h>

#ifndef LOG_TAG
#define LOG_TAG "[Offload HAL]"
#endif

// Define these to logging functions that are available for offload HAL
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__)
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)
#define LOGW(...) __android_log_print(ANDROID_LOG_WARN, LOG_TAG, __VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)
```



#### 使用

```c++
//./system/chre/apps/wifi_offload/utility.cc

void LogBssid(const uint8_t *bssid) {
  const char *bssid_str = "<non-printable>";
  char bssidBuffer[kBssidStrLen];
  if (ParseBssidToStr(bssid, bssidBuffer, kBssidStrLen)) {
    bssid_str = bssidBuffer;
  }
  LOGI("  bssid: %s", bssid_str);
}
```



### 使用样例2

```c++
//art/libart_fake/fake.cc
#define LOG_TAG "libart_fake"

#include <android/log.h>
//#1 定义
#define LOGIT(...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__)
...
//#2 使用
void Dbg::SuspendVM() {
  LOGIT("Linking to and calling into libart.so internal functions is not supported. "
        "This call to '%s' is being ignored.", __func__);
}
```

