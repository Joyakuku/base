# Camera（摄像头）运用指南

Camera 是 ESP32 上通过 DVP 接口驱动的摄像头外设，基于 `esp32-camera` 组件。
它主要解决图像采集问题：拍照、视频流、低分辨率监控、二维码扫描、AI 视觉等场景，
通过统一的 frame buffer 接口获取图像数据。

---

## 初始化流程

典型初始化顺序如下：

1. 从乐鑫组件注册表下载 `esp32-camera` 组件。
2. 配置 `camera_config_t` 结构体（引脚、时钟、格式、分辨率）。
3. 调用 `esp_camera_init()` 初始化摄像头。
4. 通过 `esp_camera_fb_get()` 获取帧数据，处理完成后调用 `esp_camera_fb_return()` 释放。

---

## 常用概念

### DVP（Digital Video Port）

ESP32 与摄像头之间的并行数据接口。
通过 8 个数据引脚（D0-D7）和同步信号（PCLK、VSYNC、HREF）传输图像数据。

### SCCB（Serial Camera Control Bus）

两线串行总线，用于配置摄像头的内部寄存器（分辨率、白平衡、曝光等）。
在 ESP32 上通过两个 GPIO 模拟：
- `pin_sccb_sda`：数据线。
- `pin_sccb_scl`：时钟线。

与 I2C 协议兼容，可以理解为 I2C 的一个子集。

### XCLK（External Clock）

ESP32 提供给摄像头的时钟信号，是摄像头工作的主时钟基准。
通常设置为 10 MHz 或 20 MHz，由摄像头数据手册规定。

### PCLK（Pixel Clock）

摄像头输出的像素时钟。每个 PCLK 周期输出一个像素的数据。
接收端（ESP32）用 PCLK 的边沿采样 D0-D7 上的像素值。

XCLK 是输入给摄像头的时钟，PCLK 是摄像头输出的时钟，两者分别用于驱动和采样。

### VSYNC（Vertical Sync）

帧同步信号。每拉高一次表示一帧图像的开始，信号结束时一帧传输完成。

### HREF（Horizontal Reference）

行同步信号。为高时表示当前正在传输一行的有效像素数据。

### Frame Buffer

摄像头采集到的图像数据存放的内存区域。
通过 `camera_fb_t` 结构体访问：

- `fb->buf`：像素数据缓冲区指针。
- `fb->width` / `fb->height`：图像宽高。
- `fb->len`：数据长度（字节）。
- `fb->format`：像素格式。

每次调用 `esp_camera_fb_get()` 获取一个 frame buffer，
用完后必须调用 `esp_camera_fb_return()` 归还，否则会导致内存耗尽。

### fb_count

frame buffer 的缓存数量。通常设为 `1` 或 `2`。
设为 `2` 可以在处理一帧的同时接收下一帧，但占用内存翻倍。

### grab_mode

当所有 frame buffer 都被占用时的新帧处理策略：

- `CAMERA_GRAB_WHEN_EMPTY`：有空闲 buffer 时才接收新帧（不丢帧模式）。
- `CAMERA_GRAB_LATEST`：始终接收最新帧，覆盖旧数据（适合实时显示）。

---

## 1. 安装组件

通过 VS Code 命令面板安装 `esp32-camera`：

1. `Ctrl+Shift+P` 打开命令面板。
2. 搜索 `Show ESP Component Registry`（乐鑫组件注册表）。
3. 选择目标 **IDF 版本**、**芯片型号**（target）。
4. 在 Tags 中筛选 `camera`，找到 `esp32-camera`。
5. 点击 Install，等待下载完成。

组件安装后，在代码中引入头文件：

```c
#include "esp_camera.h"
```

---

## 2. 配置摄像头

定义 `camera_config_t` 结构体，设置引脚、时钟和图像参数。

```c
static camera_config_t camera_config = {
    .pin_pwdn  = -1,                   // 低功耗控制，不接时设为 -1
    .pin_reset = -1,                   // 硬件复位，不接时设为 -1
    .pin_xclk  = GPIO_NUM_10,          // 主时钟输出（ESP32 -> 摄像头）
    .pin_sccb_sda = GPIO_NUM_40,       // SCCB 数据线
    .pin_sccb_scl = GPIO_NUM_39,       // SCCB 时钟线
    .pin_d7 = GPIO_NUM_48,             // 数据位 7
    .pin_d6 = GPIO_NUM_11,             // 数据位 6
    .pin_d5 = GPIO_NUM_12,             // 数据位 5
    .pin_d4 = GPIO_NUM_14,             // 数据位 4
    .pin_d3 = GPIO_NUM_16,             // 数据位 3
    .pin_d2 = GPIO_NUM_18,             // 数据位 2
    .pin_d1 = GPIO_NUM_17,             // 数据位 1
    .pin_d0 = GPIO_NUM_15,             // 数据位 0
    .pin_vsync = GPIO_NUM_38,          // 帧同步信号（摄像头 -> ESP32）
    .pin_href  = GPIO_NUM_47,          // 行同步信号（摄像头 -> ESP32）
    .pin_pclk  = GPIO_NUM_13,          // 像素时钟（摄像头 -> ESP32）

    .xclk_freq_hz  = 20000000,         // XCLK 频率，单位 Hz
    .ledc_timer    = LEDC_TIMER_0,     // XCLK 使用的 LEDC 定时器
    .ledc_channel  = LEDC_CHANNEL_0,   // XCLK 使用的 LEDC 通道
    .pixel_format  = PIXFORMAT_GRAYSCALE, // 像素格式
    .frame_size    = FRAMESIZE_QQVGA,     // 输出分辨率
    .jpeg_quality  = 10,               // JPEG 压缩质量
    .fb_count      = 1,                // frame buffer 数量
    .grab_mode     = CAMERA_GRAB_WHEN_EMPTY, // 抓取策略
    .fb_location   = CAMERA_FB_IN_PSRAM,    // buffer 存放位置
};
```

配置项说明：

- `pin_pwdn`
  低功耗控制引脚。
  高电平时摄像头进入低功耗模式。
  不使用时设为 `-1`。

- `pin_reset`
  硬件复位引脚。
  低电平时摄像头复位。
  不使用时设为 `-1`。

- `pin_xclk`
  ESP32 输出给摄像头的主时钟。
  必须接任意可用 GPIO，
  最终通过 LEDC 生成指定频率的方波。

- `pin_sccb_sda` / `pin_sccb_scl`
  SCCB 总线的数据和时钟引脚。
  用于配置摄像头的内部寄存器（类似 I2C）。

- `pin_d0` ~ `pin_d7`
  8 位并行数据线。
  每个 PCLK 周期传输一个字节的像素数据。

- `pin_vsync`
  帧同步信号，高电平期间摄像头输出一帧数据。

- `pin_href`
  行同步信号，高电平期间摄像头输出一行数据。

- `pin_pclk`
  像素时钟。
  ESP32 在每个 PCLK 边沿采样数据线。
  与 XCLK 不同：XCLK 是 ESP32 给摄像头的，
  PCLK 是摄像头输出给 ESP32 的。

- `xclk_freq_hz`
  主时钟频率，单位为 Hz。
  常用值：`10 MHz`（10000000）或 `20 MHz`（20000000）。
  具体参考摄像头数据手册，
  OV2640 / OV3660 通常支持 10-20 MHz。

- `ledc_timer` / `ledc_channel`
  XCLK 时钟由 ESP32 的 LEDC（LED PWM Controller）外设生成。
  选择一对未被占用的定时器和通道即可。

- `pixel_format`
  输出像素格式。常用：
  - `PIXFORMAT_JPEG`：JPEG 压缩输出。
  - `PIXFORMAT_RGB565`：16 位彩色。
  - `PIXFORMAT_GRAYSCALE`：灰度（常用于 OCR 或简单监控）。

- `frame_size`
  输出分辨率。常用：
  `FRAMESIZE_QQVGA`（160 × 120）、
  `FRAMESIZE_QVGA`（320 × 240）、
  `FRAMESIZE_VGA`（640 × 480）。
  分辨率越高，帧率越低，内存占用越大。

- `jpeg_quality`
  JPEG 压缩质量，取值范围 `0`-`63`。
  值越小质量越高、数据量越大。
  仅在 `pixel_format = PIXFORMAT_JPEG` 时生效。

- `fb_count`
  frame buffer 缓存数量。
  `1`：节省 PSRAM，但帧率有限。
  `2`：可流水线处理，但 PSRAM 占用翻倍。

- `grab_mode`
  新帧到达时所有 buffer 都被占用的处理策略。
  `CAMERA_GRAB_WHEN_EMPTY` 等待有空闲 buffer。
  `CAMERA_GRAB_LATEST` 覆盖最新的旧帧。

- `fb_location`
  frame buffer 存放位置。
  `CAMERA_FB_IN_PSRAM`：存于 PSRAM（推荐，容量大）。
  `CAMERA_FB_IN_DRAM`：存于内部 DRAM（容量小但速度快）。

---

## 3. 初始化摄像头

调用 `esp_camera_init()`，传入配置结构体的指针。

```c
esp_err_t camera_init(void)
{
    // 将低功耗引脚置低，确保摄像头退出低功耗模式
    gpio_set_level(PWDN_GPIO_NUM, 0);

    esp_err_t err = esp_camera_init(&camera_config);
    if (err != ESP_OK) {
        ESP_LOGE("camera", "Camera init failed: %s", esp_err_to_name(err));
        return err;
    }
    return ESP_OK;
}
```

注意：

- `esp_camera_init()` 内部会配置所有 GPIO 和 LEDC，
  调用前不要重复初始化相关引脚。
- 函数会分配 frame buffer 内存（PSRAM 或 DRAM），
  确保对应内存区域已在 menuconfig 中启用。

---

## 4. 获取图像帧

通过 `esp_camera_fb_get()` 获取一帧图像，
处理完成后调用 `esp_camera_fb_return()` 归还。

```c
camera_fb_t *fb = NULL;

esp_err_t camera_capture(void)
{
    fb = esp_camera_fb_get();
    if (!fb) {
        ESP_LOGE("camera", "Frame buffer could not be acquired");
        return ESP_FAIL;
    }

    // 在此处理图像数据：fb->buf, fb->width, fb->height, fb->len
    oled_draw_picture(fb->buf, fb->width, fb->height);

    esp_camera_fb_return(fb);
    fb = NULL;
    return ESP_OK;
}
```

注意：

- `esp_camera_fb_get()` 可能返回 `NULL`，
  表示所有 buffer 都被占用（如 `grab_mode = CAMERA_GRAB_WHEN_EMPTY` 时无空闲 buffer）。
- 归还后 `fb` 指针失效，
  设置为 `NULL` 避免悬空指针。
- 不要在未归还 buffer 的情况下连续多次调用 `esp_camera_fb_get()`，
  当 `fb_count = 1` 时会直接失败。

---

## 完整示例

```c
#include "esp_camera.h"
#include "esp_log.h"

#define PWDN_GPIO_NUM  -1
#define RESET_GPIO_NUM -1
#define XCLK_GPIO_NUM  10
#define SCCB_SDA_GPIO  40
#define SCCB_SCL_GPIO  39

static camera_config_t camera_config = {
    .pin_pwdn  = PWDN_GPIO_NUM,
    .pin_reset = RESET_GPIO_NUM,
    .pin_xclk  = XCLK_GPIO_NUM,
    .pin_sccb_sda = SCCB_SDA_GPIO,
    .pin_sccb_scl = SCCB_SCL_GPIO,
    .pin_d7 = GPIO_NUM_48,
    .pin_d6 = GPIO_NUM_11,
    .pin_d5 = GPIO_NUM_12,
    .pin_d4 = GPIO_NUM_14,
    .pin_d3 = GPIO_NUM_16,
    .pin_d2 = GPIO_NUM_18,
    .pin_d1 = GPIO_NUM_17,
    .pin_d0 = GPIO_NUM_15,
    .pin_vsync = GPIO_NUM_38,
    .pin_href  = GPIO_NUM_47,
    .pin_pclk  = GPIO_NUM_13,

    .xclk_freq_hz  = 20000000,
    .ledc_timer    = LEDC_TIMER_0,
    .ledc_channel  = LEDC_CHANNEL_0,
    .pixel_format  = PIXFORMAT_GRAYSCALE,
    .frame_size    = FRAMESIZE_QQVGA,
    .jpeg_quality  = 10,
    .fb_count      = 1,
    .grab_mode     = CAMERA_GRAB_WHEN_EMPTY,
    .fb_location   = CAMERA_FB_IN_PSRAM,
};

esp_err_t camera_init(void)
{
    if (PWDN_GPIO_NUM != -1) {
        gpio_set_level(PWDN_GPIO_NUM, 0);
    }
    return esp_camera_init(&camera_config);
}

camera_fb_t *fb;

esp_err_t camera_capture(void)
{
    fb = esp_camera_fb_get();
    if (!fb) {
        ESP_LOGE("camera", "Frame buffer could not be acquired");
        return ESP_FAIL;
    }

    // 处理 fb->buf

    esp_camera_fb_return(fb);
    fb = NULL;
    return ESP_OK;
}
```

以上示例表示：

- 使用默认 OV2640/OV3660 引脚映射。
- XCLK = 20 MHz。
- 灰度格式，QQVGA 分辨率。
- 单 buffer，PSRAM 存储。
- 包含完整的 init / capture 调用链。

---

## 常见注意点

### PWDN 引脚未拉低导致初始化失败

摄像头默认可能处于低功耗模式。
在 `esp_camera_init()` 之前将 `PWDN` 引脚置低：

```c
gpio_set_level(PWDN_GPIO_NUM, 0);
```

如果硬件未连接 PWDN，
设置 `pin_pwdn = -1` 即可。

### fb_count = 1 时忘记归还 buffer

当 `fb_count = 1` 时，
获取一帧后不归还就再次调用 `esp_camera_fb_get()` 会返回 `NULL`。
确保每次获取后都配对调用 `esp_camera_fb_return()`。

### PCLK 和 XCLK 引脚混淆

`pin_xclk` 是 ESP32 输出给摄像头的时钟，
`pin_pclk` 是摄像头输出给 ESP32 的时钟。
两者功能不同，不能互换。

### JPEG 质量参数仅在 JPEG 格式下生效

`jpeg_quality` 仅在 `pixel_format = PIXFORMAT_JPEG` 时有效。
使用 `PIXFORMAT_RGB565` 或 `PIXFORMAT_GRAYSCALE` 时该参数被忽略。

### 高分辨率导致 PSRAM 不足

`VGA`（640 × 480）及以上分辨率在 RGB565 格式下单帧约 600 KB。
`fb_count = 2` 约需 1.2 MB。
确认 PSRAM 已在 menuconfig 中启用（`Component config → ESP PSRAM`）。

### GPIO 与片上功能冲突

ESP32-S3 等芯片的部分 GPIO 已被 PSRAM/Flash 占用。
分配摄像头引脚前先查阅芯片数据手册，
确认目标 GPIO 可用。

### 帧率受多重因素制约

实际帧率受限于：
分辨率、像素格式、XCLK 频率、fb_count、grab_mode、
以及你的帧处理函数的耗时。
不要在 `esp_camera_fb_get()` 和 `esp_camera_fb_return()` 之间执行耗时操作。

---

## 速查

1. `esp_camera_init(&config)`
   初始化摄像头驱动，配置 GPIO / LEDC / DMA。

2. `esp_camera_fb_get()`
   获取一个 frame buffer，返回 `camera_fb_t *`。

3. `esp_camera_fb_return(fb)`
   归还 frame buffer，释放该 buffer 供后续帧使用。

4. `esp_camera_deinit()`
   反初始化摄像头，释放所有资源。

5. `esp_camera_sensor_get()`
   获取底层 sensor 指针，用于调用传感器级 API。

6. `camera_config_t`
   初始化配置结构体，包含所有引脚、时钟、格式参数。

7. `camera_fb_t`
   frame buffer 结构体，成员：`buf`、`width`、`height`、`len`、`format`。

8. `PIXFORMAT_JPEG` / `PIXFORMAT_RGB565` / `PIXFORMAT_GRAYSCALE`
   三种常用像素格式枚举。

9. `FRAMESIZE_QQVGA` / `FRAMESIZE_QVGA` / `FRAMESIZE_VGA`
   三种常用分辨率枚举。

10. `CAMERA_GRAB_WHEN_EMPTY` / `CAMERA_GRAB_LATEST`
    两种抓取策略枚举。

---

## 后续扩展复习点

1. 显示优化（已独立成文：[显示优化指南](显示优化指南.md)）
   OLED 分块刷新、SPI DMA 传输、差异刷新和双缓冲等方案的设计与取舍。

2. JPEG 编码与 MJPEG 视频流
   学习如何将连续 JPEG 帧封装成 MJPEG 流，
   通过 HTTP 或 WebSocket 推送到浏览器。

3. 传感器级高级配置
   通过 `sensor_t *` 接口调节白平衡、曝光、增益、特效等参数。

4. DMA 缓冲与 PSRAM 分配策略
   深入理解 camera 驱动的内存管理、
   `fb_location` 选择依据以及分辨率与内存的权衡。

5. 双核任务拆分
   将图像采集放在一个核心，
   图像处理（如 AI 推理）放在另一个核心，
   减少丢帧。

6. 不同摄像头模组的差异
   对比 OV2640、OV3660、OV5640 在寄存器、支持分辨率、
   JPEG 能力上的区别，整理适配模板。
