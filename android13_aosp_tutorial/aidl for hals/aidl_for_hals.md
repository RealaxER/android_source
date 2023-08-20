# ADIL FOR HALs
## 1. IMPLEMENT AIDL 
### Create file aidl
```
cd aosp/source
mkdir -p hardware/interfaces/hello/aidl/android/hardware/hello/
nano hardware/interfaces/hello/aidl/android/hardware/hello/IHello.aidl
```

#### IHello.aidl
```
package android.hardware.hello ;

interface IHello {
    void send_string(in String string_init);
}
```

***
## IMPLEMENT AIDL STABLE 

### Create aidl_interface.
```
nano hardware/interfaces/hello/aidl/Android.bp
```

```
aidl_interface {
    name: "android.hardware.hello",
    vendor: true,
    srcs: ["android/hardware/hello/*.aidl"],
    owner: "BHien",
    backend: {
        cpp: {
            enabled: false,
        },
        java: {
            sdk_version: "module_current",
        },
    },
}
```

### Update api aidl 
```
m android.hardware.hello-update-api
```
### Build aidl stable
```
mmm hardware/interfaces/hello/
```
### Add aidl stable for vendor
```
nano build/make/target/product/base_vendor.mk
```
```
PRODUCT_PACKAGES += \
    android.hardware.hello \
```

***
## 2. Implement HALs
### Write HALs 
```
cd hardware/interfaces/hello/aidl
mkdir -p default
cd default 
nano Change.cpp
```
```
#define LOG_TAG "Change"

#include <utils/Log.h>
#include <iostream>
#include <fstream>
#include "Change.h"

namespace aidl {
namespace android {
namespace hardware {
namespace hello {

//String getChars();
ndk::ScopedAStatus Change::getChars(std::string* _aidl_return) {
    std::ifstream change_file;
    change_file.open("/dev/hello");
    if(change_file.good()) {
        std::string line;
        change_file >> line;
        ALOGD("Change service: %s", line.c_str());
        *_aidl_return =  line;
    } else {
        ALOGE("Can not open /dev/hello");
        return ndk::ScopedAStatus::fromServiceSpecificError(-1);
    }
    return ndk::ScopedAStatus::ok();
}

//void send_string(in String msg);
ndk::ScopedAStatus Change::send_string(const std::string& in_msg) {
    std::ofstream change_file;
    change_file.open ("/dev/hello");
    if(change_file.good()) {
        change_file << in_msg;
        ALOGD("Change service: %s", in_msg.c_str());
    } else {
        ALOGE("Can not open /dev/hello");
        return ndk::ScopedAStatus::fromServiceSpecificError(-1);
    }
    return ndk::ScopedAStatus::ok();
}

}  // namespace hello
}  // namespace hardware
}  // namespace android
}  // namespace aidl
```

### Create header file aidl
nano Change.h
```
#pragma once

#include <aidl/android/hardware/hello/BnChange.h>

namespace aidl {
namespace android {
namespace hardware {
namespace hello {

class Change: public BnChange{
    public:
        //String getChars();
        ndk::ScopedAStatus getChars(std::string* _aidl_return);
        //void send_string(in String msg);
        ndk::ScopedAStatus send_string(const std::string& in_msg);
};

}  // namespace hello
}  // namespace hardware
}  // namespace android
}  // namespace aidl
```
***
## 3. Implement service HALs
### Write service file
```
nano service.cpp
```
```
#define LOG_TAG "Change"
/*Các thư viện của binder_manager*/
#include <android-base/logging.h>
#include <android/binder_manager.h>
#include <android/binder_process.h>
#include <binder/ProcessState.h>
#include <binder/IServiceManager.h>
// thư viện Change.h của HALs AIDL đã viết ở trên
#include "Change.h"

/*Do nó quá dài nên cần phải using lại cho ngắn*/
using aidl::android::hardware::hello::Change;
using std::string_literals::operator""s;

void log_line(std::string msg) {
    std::cout << msg << std::endl;
    ALOGD("%s", msg.c_str());
}

void log_err(std::string msg) {
    std::cout << msg << std::endl;
    ALOGE("%s", msg.c_str());
}

int main() {
    // Cho phép vendor gọi vendor bằng vndbinder call 
    android::ProcessState::initWithDriver("/dev/vndbinder");

    /*Cấu hình thread*/
    ABinderProcess_setThreadPoolMaxThreadCount(0);
    ABinderProcess_startThreadPool();

    /*Truy cập vào default của aidl hello*/
    std::shared_ptr<Change> hello = ndk::SharedRefBase::make<Change>();
    const std::string name = Change::descriptor + "/default"s;

    /*Check xem có trỏ đến được không*/
    if (hello != nullptr) {
        if(AServiceManager_addService(hello->asBinder().get(), name.c_str()) != STATUS_OK) {
            log_err("Failed to register IHello service");
            return -1;
        }
    } else {
        log_err("Failed to get IHello instance");
        return -1;
    }

    log_line("IHello service starts to join service pool");
    ABinderProcess_joinThreadPool();

    return EXIT_FAILURE; 
}
```

## 4. Implement configuration services HALs
```
nano Android.bp
```
```
cc_binary {
    name: "android.hardware.hello-service",
    vendor: true,
    relative_install_path: "hw",
    init_rc: ["android.hardware.hello-service.rc"],
    vintf_fragments: ["android.hardware.hello-service.xml"],

    srcs: [
        "Change.cpp",
        "service.cpp",
    ],

    cflags: [
        "-Wall",
        "-Werror",
    ],

    shared_libs: [
        "libbase",
        "liblog",
        "libhardware",
        "libbinder_ndk",
        "libbinder",
        "libutils",
        "android.hardware.hello-V1-ndk",
    ],
}
```



### Implement rc init

```
service android.hardware.hello-service /vendor/bin/hw/android.hardware.hello-service
        interface aidl android.hardware.hello.IHello/default
        class hal
        user system
        group system
```


### Implement device manifest
nano android.hardware.hello-service.xml
```
<manifest version="1.0" type="device">
    <hal format="aidl">
        <name>android.hardware.hello</name>
        <version>1</version>
        <fqname>IHello/default</fqname>
    </hal>
</manifest>
```

### Add device manifest in the framework manifest
```
cd && cd aosp/source
```
```
nano hardware/interfaces/compatibility_matrices/compatibility_matrix.7.xml
```
```
nano hardware/interfaces/compatibility_matrices/compatibility_matrix.current.xml
```
compatibility_matrix.current.xml and compatibility_matrix.7.xml
```
<hal format="aidl" optional="true">
    <name>android.hardware.hello</name>
    <version>1</version>
    <interface>
        <name>IHello</name>
        <instance>default</instance>
    </interface>
</hal>
```
## Add service hals for vendor to build
```
nano build/make/target/product/base_vendor.mk
```
```
PRODUCT_PACKAGES += \
    android.hardware.hello \
    android.hardware.hello-service \
```

## Add service hals for vendor
```
nano build/target/product/base_vendor.mk
```
```
PRODUCT_PACKAGES += \
    android.hardware.hello-service
```
