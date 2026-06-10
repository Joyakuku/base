# I2C 搭载设备指南

I2C 是 Inter-Integrated Circuit，
常用于 ESP32 和传感器、RTC、EEPROM、OLED 等低速外设通信。

ESP-IDF 新版 I2C master 驱动通常分为两层：

- I2C master bus
  表示一条 I2C 总线，
  负责 SCL、SDA、上拉、中断和传输队列等公共配置。

- I2C device
  表示挂载到总线上的一个从设备，
  保存设备地址和通信速率等参数。

---

## 初始化流程

典型初始化顺序如下：

1. 创建 I2C master bus。
2. 配置从设备地址和通信速率。
3. 将设备挂载到 I2C 总线上。
4. 保存返回的设备句柄。

---

## 常用概念

### Bus

Bus 表示一条 I2C 总线。

同一条总线上的设备共享：

- `SCL`
  时钟线。

- `SDA`
  数据线。

同一条 I2C 总线可以挂多个设备，
但设备地址不能冲突。

### Device

Device 表示 I2C 总线上的一个从设备。

ESP-IDF 会用 `i2c_master_dev_handle_t`
保存设备句柄。

后续读写该设备时，
应使用这个设备句柄。

### Address

I2C 设备地址通常是 `7 bit` 地址。

ESP-IDF 中配置 `device_address` 时，
一般填写手册中的 `7-bit address`，
不要填写左移后的 `8-bit read/write address`。

### Speed

`scl_speed_hz` 用于设置 SCL 时钟频率。

常见值：

- `100000`
  标准模式，100 kHz。

- `400000`
  快速模式，400 kHz。

调试新设备时，
建议先用 `100 kHz` 验证通信链路。

---

## 1. 初始化 I2C 总线

```c
esp_err_t i2c_new_master_bus(
    const i2c_master_bus_config_t
        *bus_config,
    i2c_master_bus_handle_t
        *ret_bus_handle
);
```

该函数用于创建 I2C master bus。

创建成功后，
`ret_bus_handle` 会返回总线句柄。

后续向总线添加设备时，
需要使用这个句柄。

示例配置：

```c
i2c_master_bus_config_t bus_cfg = {
    .clk_source =
        I2C_CLK_SRC_DEFAULT,
    .flags.enable_internal_pullup =
        true,
    .glitch_ignore_cnt = 7,
    .i2c_port = I2C_NUM_0,
    .intr_priority = 0,
    .scl_io_num = SCL,
    .sda_io_num = SDA,
    .trans_queue_depth = 0,
};
```

配置项说明：

- `clk_source`
  设置 I2C 时钟源。

- `enable_internal_pullup`
  是否启用内部上拉。

- `glitch_ignore_cnt`
  设置毛刺过滤阈值。

- `i2c_port`
  选择使用的 I2C 控制器。

- `intr_priority`
  设置中断优先级。
  `0` 表示使用默认优先级。

- `scl_io_num`
  绑定 SCL 引脚。

- `sda_io_num`
  绑定 SDA 引脚。

- `trans_queue_depth`
  设置异步传输队列深度。

当 `trans_queue_depth = 0` 时，
驱动采用同步传输。

注意：

- I2C 总线必须上拉才能正常工作。
- 内部上拉通常较弱。
- 实际项目中更推荐使用硬件上拉。

---

## 2. 挂载 I2C 设备

```c
esp_err_t i2c_master_bus_add_device(
    i2c_master_bus_handle_t
        bus_handle,
    const i2c_device_config_t
        *dev_config,
    i2c_master_dev_handle_t
        *ret_handle
);
```

该函数用于向 I2C 总线添加设备。

创建成功后，
`ret_handle` 会返回设备句柄。

后续读写该设备时，
需要使用这个设备句柄。

示例配置：

```c
i2c_device_config_t dev_cfg = {
    .dev_addr_length =
        I2C_ADDR_BIT_LEN_7,
    .device_address = I2C_ADDR,
    .scl_speed_hz = 400000,
};
```

配置项说明：

- `dev_addr_length`
  设置设备地址位宽。

- `device_address`
  设置设备地址。
  具体地址由芯片手册决定。

- `scl_speed_hz`
  设置 SCL 时钟频率。

常见 I2C 速率：

- `100000`
  标准模式，100 kHz。

- `400000`
  快速模式，400 kHz。

---

## 完整示例

```c
void i2c_device_init(void)
{
    i2c_master_bus_handle_t
    bus_handle = NULL;

    i2c_master_dev_handle_t
    dev_handle = NULL;

    i2c_master_bus_config_t bus_cfg = {
        .clk_source =
            I2C_CLK_SRC_DEFAULT,
        .flags.enable_internal_pullup =
            true,
        .glitch_ignore_cnt = 7,
        .i2c_port = I2C_NUM_0,
        .intr_priority = 0,
        .scl_io_num = SCL,
        .sda_io_num = SDA,
        .trans_queue_depth = 0,
    };

    i2c_new_master_bus(
        &bus_cfg,
        &bus_handle
    );

    i2c_device_config_t dev_cfg = {
        .dev_addr_length =
            I2C_ADDR_BIT_LEN_7,
        .device_address = I2C_ADDR,
        .scl_speed_hz = 400000,
    };

    i2c_master_bus_add_device(
        bus_handle,
        &dev_cfg,
        &dev_handle
    );
}
```

---

## 常见注意点

### I2C 总线必须上拉

I2C 是开漏 / 开集电极结构，
SCL 和 SDA 需要上拉电阻才能稳定工作。

ESP32 内部上拉通常偏弱。
实际项目中更推荐使用外部上拉电阻。

### 注意 7-bit 和 8-bit 地址

很多芯片手册会同时给出 `7-bit address`
和 `8-bit read/write address`。

ESP-IDF 的 `device_address`
通常填写 `7-bit address`。

### 同一总线设备地址不能冲突

多个设备可以共享同一条 I2C 总线，
但地址必须不同。

如果两个模块地址固定且冲突，
需要换地址脚、换总线，
或使用 I2C multiplexer。

### 先低速验证再提速

新设备调试阶段建议先用 `100 kHz`。

确认供电、上拉、地址和读写时序正确后，
再提高到 `400 kHz` 或更高。

---

## 速查

1. `i2c_new_master_bus`
   创建 I2C master bus。

2. `i2c_master_bus_add_device`
   向总线挂载 I2C 设备。

3. `i2c_master_bus_handle_t`
   表示 I2C 总线句柄。

4. `i2c_master_dev_handle_t`
   表示 I2C 设备句柄。

---

## 后续扩展复习点

1. I2C 上拉电阻计算
   复习总线电容、上升时间、
   `100 kHz` / `400 kHz` 对上拉电阻的影响。

2. 7-bit 和 8-bit 地址
   继续整理常见芯片手册中的地址写法，
   避免把 `0x78` 当成 `0x3C` 设备地址。

3. `glitch_ignore_cnt`
   理解毛刺过滤阈值如何影响噪声环境下的通信稳定性。

4. `trans_queue_depth`
   复习同步传输和异步传输队列的使用边界。

5. Clock stretching
   学习从设备拉低 SCL 延长传输的场景和排错方法。

6. Repeated start
   复习寄存器读取中写寄存器地址后继续读数据的典型时序。
