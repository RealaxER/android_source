# SELINUX
### Create new type
```
code system/sepolicy/prebuilts/api/33.0/public/hwservice.te
```
```
code system/sepolicy/public/hwservice.te
```
Thêm type mới sẽ có những thuộc tính sau để gán cho type mới 

+ hwservice_manager_type : mặc định
+ coredomain_hwservice : sẽ có độ tin cậy cao hơn hwservice thông thường từ vendor
+ protected_hwservice : bảo vệ HAL từ chối các ứng dụng không đáng tin cậy truy cập 

```
type hal_change_hwservice, hwservice_manager_type;
```

### Set service context interface:
```
code system/sepolicy/prebuilts/api/33.0/private/hwservice_contexts
```
```
code system/sepolicy/private/hwservice_contexts
```
copy
```
android.hardware.change::IChange                         u:object_r:hal_change_hwservice:s0
```

### Create attributes
```
code system/sepolicy/prebuilts/api/32.0/public/attributes
```
```
code system/sepolicy/public/attributes
```
copy it 
```
hal_attribute(change);
```
Dùng để define 
+ hal_change
+ hal_change_client
+ hal_change_server


### Create domain
```
code system/sepolicy/vendor/hal_change_service.te
```
copy it
```
type hal_change_service, domain;

#ha_change_service là sever của HAL_Change
hal_server_domain(hal_change_service, hal_change)
# tạo 
type hal_change_service_exec, exec_type, vendor_file_type, vendor_file_type, file_type;
init_daemon_domain(hal_change_service)
```


### Link binder
```
code system/sepolicy/prebuilts/api/33.0/public/hal_change.te
```
```
code system/sepolicy/public/hal_change.te
```
copy it 
```
binder_call(hal_change_client, hal_change_server)
binder_call(hal_change_server, hal_change_client)
# Liên kết hal với aidl hal
hal_attribute_hwservice(hal_change, hal_change_hwservice) 
```

### Set cilent domain
```
code system/sepolicy/prebuilts/api/33.0/private/system_server.te
```
```
code system/sepolicy/private/system_server.te
```
copy it 
```
#system sever là client của HAL_Change
hal_client_domain(system_server, hal_change)
```

### Add path
```
code system/sepolicy/vendor/file_contexts
```
copy it
```
/(vendor|system/vendor)/bin/hw/android\.hardware\.change-service u:object_r:hal_change_service_exec:s0
```

### Initialization on boot
```
code device/brcm/rpi4/ramdisk/init.rpi4.rc
```
copy it
```
on boot
    chown system system /dev/change
    chmod 0600 /dev/change
```
