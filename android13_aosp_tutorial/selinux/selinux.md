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
+ domain: Loại dành riêng cho các tiến trình (processes), đại diện cho phạm vi và quyền của các tiến trình trong hệ thống.

+ fs_type: Đại diện cho các loại file system (hệ thống tập tin), đặc biệt là loại của hệ thống tập tin được sử dụng trên máy tính.

+ file_type: Được sử dụng để phân loại các tập tin (files) dựa trên loại, thường được sử dụng trong việc quản lý quyền truy cập đối với các tập tin.

+ exec_type: Đại diện cho các loại tệp thực thi (executable files), thường được sử dụng để kiểm soát quyền truy cập vào các tệp thực thi.

+ data_file_type: Được sử dụng để định rõ các loại tệp dữ liệu (data files), thường được sử dụng trong việc quản lý quyền truy cập vào các tệp dữ liệu cụ thể.

+ core_data_file_type: Tương tự như "data_file_type", nhưng có thể áp dụng đặc biệt cho các tệp dữ liệu trong thư mục "/data" và không thuộc "/data/vendor".

+ vendor_file_type: Đại diện cho các tệp trong thư mục "/vendor", có thể áp dụng kiểm soát truy cập đặc biệt cho các tệp trong thư mục này.

+ proc_type: Đại diện cho các loại tệp trong hệ thống tệp procfs, thường được sử dụng để kiểm soát truy cập vào thông tin liên quan đến các tiến trình trong hệ thống.

+ sysfs_type: Đại diện cho các loại tệp trong hệ thống sysfs, thường được sử dụng để kiểm soát truy cập vào thông tin liên quan đến các thiết bị và tài nguyên hệ thống.


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
