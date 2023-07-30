# SELINUX
Note that the 2 copied files must be in the same position and the same, the 2 files must be similar in all points
### Create new type
```
nano system/sepolicy/prebuilts/api/33.0/public/hwservice.te
```
```
nano system/sepolicy/public/hwservice.te
```
hwservice.te and hwservice.te
```
type hal_change_hwservice, hwservice_manager_type;
```


### Service context
```
nano system/sepolicy/prebuilts/api/33.0/private/hwservice_contexts
```
```
nano system/sepolicy/private/hwservice_contexts
```
copy it 
```
android.hardware.change::IChange                         u:object_r:hal_change_hwservice:s0
```

### Create attributes
```
nano system/sepolicy/prebuilts/api/32.0/public/attributes
```
```
nano system/sepolicy/public/attributes
```
copy it 
```
hal_attribute(change);
```

### Create domain
```
nano system/sepolicy/vendor/hal_change_service.te
```
copy it
```
type hal_change_service, domain;
hal_server_domain(hal_change_service, hal_change)
type hal_change_service_exec, exec_type, vendor_file_type, vendor_file_type, file_type;
init_daemon_domain(hal_change_service)
```


### Link binder
```
nano system/sepolicy/prebuilts/api/33.0/public/hal_change.te
```
```
nano system/sepolicy/public/hal_change.te
```
copy it 
```
binder_call(hal_change_client, hal_change_server)
binder_call(hal_change_server, hal_change_client)
hal_attribute_hwservice(hal_change, hal_change_hwservice)
```

### Set cilent domain
```
nano system/sepolicy/prebuilts/api/33.0/private/system_server.te
```
```
nano system/sepolicy/private/system_server.te
```
copy it 
```
hal_client_domain(system_server, hal_change)
```

### Ignore lower api
```
nano system/sepolicy/prebuilts/api/33.0/private/compat/32.0/32.0.ignore.cil
```
```
nano system/sepolicy/private/compat/32.0/32.0.ignore.cil
```
copy it 
```
(type new_objects)
(typeattribute new_objects)
(typeattributeset new_objects
    ( new_objects
        hal_change_hwservice
    )
)
```

### Add path
```
nano system/sepolicy/vendor/file_contexts
```
copy it
```
/(vendor|system/vendor)/bin/hw/android\.hardware\.change-service u:object_r:hal_change_service_exec:s0
```

### Initialization on boot
```
nano device/brcm/rpi4/ramdisk/init.rpi4.rc
```
copy it
```
on boot
    chown system system /dev/change
    chmod 0600 /dev/change
```
