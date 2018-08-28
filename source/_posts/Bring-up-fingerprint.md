---
title: TrustZone及指纹
tags: FingerPrint TEE
---




#### 背景
- 客户反馈需要指纹，通过 IFAA 认证
- 产品有数据收集需求的功能 - 多用户


#### OMTP 

![OMTP](https://note.youdao.com/yws/res/18071/WEBRESOURCE1d1268b09a532fde9f160624d9db6f49)

- REE: (Rich Execution Environment) 非信执行环境
- TEE: (Trust Execution Environment) 可信执行环境
- TA:(Trust application) 运行在 TEE 内的 app.

#### TEE

TEE通常用于运行关键的操作：
- (1)、移动支付：指纹验证、PIN码输入等；
- (2)、机密数据：私钥、证书等的安全存储；
- (3)、内容保护：DRM(数字版权保护)等。


TEE三层架构：
 - TA(Trusted Application)层：可信钱包、TUI；
 - TEE层：安全操作系统，对上层TA提供的库；
 - 硬件层：CPU状态隔离、内存隔离、外设隔离，如下图：
![TEE Framework](https://note.youdao.com/yws/res/17915/WEBRESOURCE1d4994126f8a3f9c3bf3f96d98692ce0)


- AMD:
- - Platform Security Processor (PSP)[20][21][22]
- - AMD Secure Execution Environment[23]
- ARM:
- - TrustZone[24]
- - - QSEE, a commercial implementation from Qualcomm
- Intel:
- - Trusted Execution Technology[23]
- - SGX Software Guard Extensions[25]
- - "Silent Lake" (available on Atom processors)[26][27][28]

- ARM在芯片IP设计中已全面支持了TEE，包括高通、联发科、三星、海思、展讯等企业已采用，成为基于硬件安全的方案。




#### 硬件原理图

![硬件原理图](https://note.youdao.com/yws/res/17836/WEBRESOURCEa124b1930116d968cc0fd037a054858c)



#### 软件框图

![软件框图](https://note.youdao.com/yws/res/17862/WEBRESOURCE7d38e06579d915c1c534bf2a9ede3adc)

#### 驱动移植
```
178         //goodix GF3206 fp
179         goodix_fp{
180                 compatible = "goodix,fingerprint";
181
182                 interrupt-parent = <&tlmm>;
183
184                 fp-gpio-irq = <&tlmm 121 0x00>;
185
186                 fp-gpio-reset = <&tlmm 91 0x00>;
187
188                 vdd-fp-supply = <&pm8998_l19>;
189
190                 status = "ok";
191         };

```


#### 原生指纹框架编译

- /android/device/lenovo/g2/g2.mk
```
272 #Fingerprint Sensor
273 PRODUCT_PACKAGES += android.hardware.biometrics.fingerprint@2.1-service \
274                     android.hardware.biometrics.fingerprint@2.1
275
276 PRODUCT_COPY_FILES += frameworks/native/data/etc/android.hardware.fingerprint.xml:vendor/etc/permissions/android.hardware.fingerprint.xml \
277                       device/lenovo/g2/goodixfp/libfingerprint.default.so:vendor/lib64/hw/fingerprint.default.so \
278                       device/lenovo/g2/goodixfp/libgf_ca.so:vendor/lib64/libgf_ca.so \
279                       device/lenovo/g2/goodixfp/libgf_hal.so:vendor/lib64/libgf_hal.so \
280                       device/lenovo/g2/goodixfp/libvendor.goodix.hardware.biometrics.fingerprint@2.1.so:vendor/lib64/libvendor.goodix.hardware.biometrics.fingerprint@2.1.so \
281                       device/lenovo/g2/goodixfp/libgoodixhwfingerprint.so:vendor/lib64/libgoodixhwfingerprint.so

```

![image](http://note.youdao.com/yws/res/18104/46E7F116537946BDB40A9CF41B526516)

#### 指纹 SO 库部署
```

```

#### SE 权限文件移植
- device/qcom/sepolicy
> ![common](https://note.youdao.com/yws/res/18248/WEBRESOURCE334dc1c9b9e56450c1c5d111364fe65f)

- platform/system/sepolicy
> ![systemse](http://note.youdao.com/yws/res/18245/WEBRESOURCE82f2264b594275facb8f16db2121dbf6)

#### TrustZone TA 程序编写
- SampleApp
- Function:  **tz_app_init** /  *Description: the entry of TA*

- CMD 分发
```
    switch (cmd_unpacked->msg.operation_id)
        {
            // normal operation
            case GF_OPERATION_ID:
            {
                ret = gf_ta_invoke_cmd_entry_point(cmd_unpacked->cmd_id, ptr,
                                                   req_size);
                break;
            }

            // especial operation
            case GF_USER_OPERATION_ID:
            ...
```


```
 37 //All SEs have to be listed below. Any SE not present cannot be accesssed by any subsystem. 
 38 //It's designed to be flexible enough to list only available SEs on a particular platform.
 39 
 40 const QUPv3_se_security_permissions_type qupv3_perms_default[] =
 41 {
 42   /*   PeriphID,         ProtocolID,               Mode, NsOwner,bAllowFifo,bLoad,bModExcl  */
 43   { QUPV3_0_SE0, QUPV3_PROTOCOL_SPI,     QUPV3_MODE_GSI, AC_TZ,       FALSE, TRUE, TRUE }, // NFC eSE
 44   /*QUPV3_0_SE1*/
 45   { QUPV3_0_SE2, QUPV3_PROTOCOL_SPI, QUPV3_MODE_FIFO,AC_HLOS,      TRUE, TRUE,FALSE }, // Napier MAWC UART
 46   { QUPV3_0_SE3, QUPV3_PROTOCOL_I2C,     QUPV3_MODE_GSI, AC_HLOS,      TRUE, TRUE,FALSE }, // NFC 
 47   /*QUPV3_0_SE4*/
 48   { QUPV3_0_SE5, QUPV3_PROTOCOL_I2C,     QUPV3_MODE_FIFO,AC_HLOS,      TRUE, TRUE,FALSE }, // legacy touch for WP
 49   { QUPV3_0_SE6, QUPV3_PROTOCOL_UART_4W, QUPV3_MODE_FIFO,AC_HLOS,      TRUE, TRUE,FALSE }, // Cherokee BT HCI
 50   /*QUPV3_0_SE7*/
 51   { QUPV3_1_SE0, QUPV3_PROTOCOL_SPI,     QUPV3_MODE_GSI, AC_HLOS,      TRUE, TRUE,FALSE }, // WCD9340
 52   { QUPV3_1_SE1, QUPV3_PROTOCOL_UART_2W, QUPV3_MODE_FIFO,AC_HLOS,      TRUE,FALSE,FALSE }, // Debug
 53   { QUPV3_1_SE2, QUPV3_PROTOCOL_I2C,     QUPV3_MODE_FIFO,AC_HLOS,      TRUE, TRUE,FALSE }, // Haptics (sensors hub)
 54   { QUPV3_1_SE3, QUPV3_PROTOCOL_SPI,     QUPV3_MODE_GSI, AC_HLOS,      TRUE, TRUE,FALSE }, // CDP spare
 55   { QUPV3_1_SE4, QUPV3_PROTOCOL_SPI,     QUPV3_MODE_FIFO,AC_HLOS,      TRUE, TRUE,FALSE }, // ANX (TypeC)
 56   /*QUPV3_1_SE5*/
 57   { QUPV3_1_SE6, QUPV3_PROTOCOL_I2C,     QUPV3_MODE_FIFO, AC_HLOS,     TRUE, TRUE,FALSE }, // Forcetouch
 58   { QUPV3_1_SE7, QUPV3_PROTOCOL_SPI,     QUPV3_MODE_GSI, AC_TZ,       FALSE, TRUE, TRUE }, // Fingerprint
 59 };

```

```
 AC_NONE – Peripheral cannot be accessed. This setting is uncommon. It cannot be paired with any other EE identifier.
 AC_TZ – Peripheral may be accessed from the TZ software. This cannot be paired with another EE identifier.
 AC_HLOS – Peripheral may be accessed from the HLOS
 AC_HYP – Peripheral may be accessed from the hypervisor
 AC_SSC_Q6_ELF – Peripheral may be accessed from the sensors core
 AC_ADSP_Q6_ELF – Peripheral may be accessed from the ADSP
 AC_DEFAULT – QUP access driver does not manage ownership for the peripheral. The peripheral’s access control is determined by the static configuration previously set by TZ/HYP.
 AC_MSS_MSA – Peripheral may be accessed from the modem subsystem

```

#### 指纹 TA 编译

#### DEBUG
- TA load failed
> 找不到 TA 程序，TA 是打包在 NON-HLOS.bin 中，可通过如下方式 push 到文件系统中验证
```
adb shell mount -o remount,RW /firmware
#adb push ./build/ms/bin/PIL_IMAGES/SPLITBINS_WAXAANAA/goodixfp.mdt /system/etc/firmware
adb push ./build/ms/bin/PIL_IMAGES/SPLITBINS_WAXAANAA/goodixfp.b00 /firmware/image
```

- SessionID/CmdID not match
> libgf_ca.so <---> gf_ta Update SO.

- RESET GPIO 占用
> 去除 Qcom 自带的驱动，Qcom 原生的指纹 QTI 在竞争该 GPIO 口。

- 休眠状态下有概率无法解锁
>驱动使用 wakeup_source 接口保持 2000 ms 处理时间。


#### 几个指标
- 据真率 FRR < 3%
- 认假率 FAR < 1%
- 匹配时间 250ms
- 抗静电指标 15KV




- [TEE Wiki](https://en.wikipedia.org/wiki/Trusted_execution_environment)
- [TEE(Trusted Execution Environment)简介](https://blog.csdn.net/fengbingchun/article/details/78657188)
