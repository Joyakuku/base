# I2C 搭载设备指南

本指南记录 ESP-IDF 中
I2C master 总线初始化、
以及向总线挂载设备的基本流程。

典型顺序如下：

1. 创建 I2C master bus。
2. 配置从设备地址和通信速率。
3. 将设备挂载到 I2C 总线上。
4. 保存返回的设备句柄。

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

## 速查

1. `i2c_new_master_bus`
   创建 I2C master bus。

2. `i2c_master_bus_add_device`
   向总线挂载 I2C 设备。

3. `i2c_master_bus_handle_t`
   表示 I2C 总线句柄。

4. `i2c_master_dev_handle_t`
   表示 I2C 设备句柄。
