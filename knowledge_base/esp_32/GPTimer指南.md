# GPTimer（通用定时器）指南

GPTimer 是 ESP-IDF 提供的通用硬件定时器驱动。

它常用于周期性任务、精确定时、超时检测，
以及需要在定时到达后触发回调的场景。

---

## 初始化流程

![GPTimer 初始化流程](GPTimer-1-Init.png)

典型初始化顺序如下：

1. 定义定时器句柄。
2. 创建并配置定时器。
3. 配置报警比较器。
4. 注册报警回调函数。
5. 使能定时器。
6. 启动定时器。

---

## 常用概念

### Handle

`gptimer_handle_t` 表示一个 GPTimer 实例。

创建、配置报警、注册回调、启动和停止，
都需要通过这个句柄指定目标定时器。

### Resolution

`resolution_hz` 表示定时器每秒计数多少次。

例如 `resolution_hz = 1000000` 时，
每 `1 us` 计数一次。

### Alarm

Alarm 是 GPTimer 的比较触发点。

当计数值到达 `alarm_count` 时，
定时器会产生报警事件。

### Auto Reload

`auto_reload_on_alarm` 用于设置报警后是否自动重装载。

周期性定时通常设置为 `1`，
一次性比较触发通常设置为 `0`。

### Callback

GPTimer 的报警回调运行在中断相关上下文中。

回调中应只做轻量操作，
例如设置标志位或通知任务。

---

## 1. 定义定时器句柄

```c
gptimer_handle_t gptimer_name;
```

`gptimer_handle_t` 是 GPTimer 的句柄类型。

后续配置、使能、启动、停止等操作，
都需要通过这个句柄指定目标定时器。

---

## 2. 创建定时器

```c
esp_err_t gptimer_new_timer(
    const gptimer_config_t *config,
    gptimer_handle_t *ret_timer
);
```

该函数用于根据 `config` 创建定时器，
并通过 `ret_timer` 返回定时器句柄。

示例配置：

```c
gptimer_config_t gptimer_cfg = {
    .clk_src =
        GPTIMER_CLK_SRC_DEFAULT,
    .direction = GPTIMER_COUNT_UP,
    .flags.intr_shared = 0,
    .intr_priority = 0,
    .resolution_hz = 1000000,
};
```

配置项说明：

- `clk_src`
  选择定时器时钟源。

- `direction`
  设置向上计数或向下计数。

- `flags.intr_shared`
  设置是否共享中断资源。

- `intr_priority`
  设置中断优先级。
  `0` 表示自动分配。

- `resolution_hz`
  设置定时器分辨率，
  单位为 Hz。

当 `resolution_hz = 1000000` 时，
定时器每秒计数 `1,000,000` 次。

也就是每 `1 us` 计数一次。

---

## 3. 配置报警比较器

```c
esp_err_t gptimer_set_alarm_action(
    gptimer_handle_t timer,
    const gptimer_alarm_config_t *config
);
```

该函数用于设置定时器的报警动作。

当计数值到达 `alarm_count` 时，
GPTimer 会触发报警事件。

示例配置：

```c
gptimer_alarm_config_t
gptimer_alarm_cfg = {
    .alarm_count = 100000,
    .flags.auto_reload_on_alarm = 1,
    .reload_count = 0,
};
```

配置项说明：

- `alarm_count`
  报警阈值。

- `auto_reload_on_alarm`
  设置报警后是否自动重装载。

- `reload_count`
  设置自动重装载后的计数值。

以上配置表示：

- 定时器计数到 `100000` 时触发报警。
- 报警后自动回到 `0` 重新计数。
- 若分辨率为 `1 MHz`，周期为 `100 ms`。

---

## 4. 注册回调函数

```c
esp_err_t gptimer_register_event_callbacks(
    gptimer_handle_t timer,
    const gptimer_event_callbacks_t *cbs,
    void *user_data
);
```

该函数用于绑定 GPTimer 事件回调。

最常用的是 `on_alarm`，
即报警触发时调用的函数。

示例配置：

```c
gptimer_event_callbacks_t
gptimer_callback_cfg = {
    .on_alarm = LED_timercallback,
};
```

回调函数格式：

```c
bool IRAM_ATTR LED_timercallback(
    gptimer_handle_t timer,
    const gptimer_alarm_event_data_t *edata,
    void *user_ctx
)
{
    return false;
}
```

`IRAM_ATTR` 表示将函数放入 IRAM。

这样可以减少从 Flash 取指带来的延迟，
更适合中断回调这类高实时性代码。

注意：

- IRAM 空间有限，不应放入过大的函数。
- 若使用 `IRAM_ATTR`，需要确认工程配置支持。
- 回调中应避免耗时操作。

---

## 5. 使能定时器

```c
esp_err_t gptimer_enable(
    gptimer_handle_t timer
);
```

使能后，GPTimer 进入可工作状态。

通常在完成定时器配置和回调注册后调用。

---

## 6. 启动定时器

```c
esp_err_t gptimer_start(
    gptimer_handle_t timer
);
```

调用后，定时器开始计数。

若已配置报警比较器，
计数到阈值时会触发报警事件。

---

## 7. 停止和删除定时器

停止定时器：

```c
gptimer_stop(gptimer_name);
```

禁用定时器：

```c
gptimer_disable(gptimer_name);
```

删除定时器：

```c
gptimer_del_timer(gptimer_name);
```

常见释放顺序如下：

1. `gptimer_stop`
   停止计数。

2. `gptimer_disable`
   禁用硬件资源。

3. `gptimer_del_timer`
   删除定时器句柄。

如果后续还要重新启动，
通常只需要停止，
不要删除句柄。

---

## 完整示例

```c
static bool IRAM_ATTR LED_timercallback(
    gptimer_handle_t timer,
    const gptimer_alarm_event_data_t *edata,
    void *user_ctx
)
{
    return false;
}

void gptimer_init(void)
{
    gptimer_handle_t gptimer_name = NULL;

    gptimer_config_t gptimer_cfg = {
        .clk_src =
            GPTIMER_CLK_SRC_DEFAULT,
        .direction = GPTIMER_COUNT_UP,
        .flags.intr_shared = 0,
        .intr_priority = 0,
        .resolution_hz = 1000000,
    };

    gptimer_new_timer(
        &gptimer_cfg,
        &gptimer_name
    );

    gptimer_alarm_config_t
    gptimer_alarm_cfg = {
        .alarm_count = 100000,
        .flags.auto_reload_on_alarm = 1,
        .reload_count = 0,
    };

    gptimer_set_alarm_action(
        gptimer_name,
        &gptimer_alarm_cfg
    );

    gptimer_event_callbacks_t
    callback_cfg = {
        .on_alarm = LED_timercallback,
    };

    gptimer_register_event_callbacks(
        gptimer_name,
        &callback_cfg,
        NULL
    );

    gptimer_enable(gptimer_name);
    gptimer_start(gptimer_name);
}
```

---

## 常见注意点

### 分辨率决定计数单位

`resolution_hz` 越高，
计数单位越小。

例如 `1 MHz` 表示 `1 us` 一个计数。
计算 `alarm_count` 时，
应先确认当前分辨率。

### 回调中不要做耗时操作

GPTimer 报警回调适合做轻量通知。

耗时逻辑应放到普通 task 中处理，
例如通过队列、信号量或事件组转交。

### 周期性定时优先使用 auto reload

周期性定时场景中，
设置 `auto_reload_on_alarm = 1`
可以让硬件自动回到 `reload_count` 继续计数。

### IRAM 不是越多越好

`IRAM_ATTR` 适合高实时性中断回调。

但 IRAM 空间有限，
不要把复杂业务逻辑放进 IRAM 回调。

---

## 速查

1. `gptimer_new_timer`
   创建定时器。

2. `gptimer_set_alarm_action`
   设置报警阈值。

3. `gptimer_register_event_callbacks`
   注册事件回调。

4. `gptimer_enable`
   使能定时器。

5. `gptimer_start`
   启动计数。

6. `gptimer_stop`
   停止计数。

7. `gptimer_disable`
   禁用定时器。

8. `gptimer_del_timer`
   删除定时器句柄。

---

## 后续扩展复习点

1. GPTimer 和 `esp_timer` 的区别
   对比硬件定时器与软件高精度定时器的适用场景。

2. `resolution_hz` 和 `alarm_count` 计算
   整理常见周期对应的计数值，
   例如 `1 ms`、`10 ms`、`100 ms`。

3. 中断回调限制
   复习 ISR 上下文中哪些 API 能调用，
   哪些操作必须交给 task。

4. `intr_priority`
   理解中断优先级选择对实时性和系统稳定性的影响。

5. `reload_count`
   复习自动重装载时从哪个计数值重新开始。

6. 多个 GPTimer 协作
   学习多个硬件定时器同时存在时的资源管理和调试方法。
