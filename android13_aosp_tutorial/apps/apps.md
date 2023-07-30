# APP TEST


### Add app in the vendor
```
cd ..
cd source
nano build/target/product/base_vendor.mk
```
copy it
```
PRODUCT_PACKAGES += \
    Change
```

### Build system and reload Flash Image android and kernel
```
m all -j$(nproc)
```
## Reload flash image and kernel 

### Add package app Change
```
cd packages/apps
git clone https://github.com/RealaxER/Change
```

### Build app
```
cd - 
m all -j$(nproc)
```

### Adb install app
```
adb install out/target/product/rpi4/system/app/Change/Change.apk
```
