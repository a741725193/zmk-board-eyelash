# ZMK Board - Eyelash Nano

这是一个基于nRF52840的ZMK键盘控制器板，名为eyelash!nano。本仓库包含了该板的定义文件和配置，可用于构建ZMK固件。

## 板子特性

- **MCU**: nRF52840 (ARM Cortex-M4F)
- **连接方式**: USB和蓝牙低功耗(BLE)
- **闪存分区**:
  - SD分区: 0x00000000 - 0x00026000
  - 代码分区: 0x00026000 - 0x000c6000
  - 存储分区: 0x000ec000 - 0x00008000
  - 引导分区: 0x000f4000 - 0x0000c000
- **支持的外设**:
  - ADC
  - USB设备
  - 蓝牙
  - IEEE 802.15.4
  - PWM
  - 看门狗
- **GPIO**: 支持GPIO0和GPIO1
- **接口**: I2C, SPI, UART
- **LED**: 蓝色LED (GPIO0 15)
- **电源管理**: 支持外部电源控制和电池监控

## 目录结构

```
├── .github/workflows/    # GitHub Actions工作流配置
├── boards/arm/eyelash_nano/    # 板子定义文件
│   ├── board.cmake            # 板子构建配置
│   ├── eyelash_nano-pinctrl.dtsi    # 引脚控制定义
│   ├── eyelash_nano.deconfig        # 默认配置
│   ├── eyelash_nano.dts             # 设备树源文件
│   ├── eyelash_nano.dtsi            # 设备树包含文件
│   ├── eyelash_nano.yaml            # 板子元数据
│   ├── eyelash_nano.zmk.yml         # ZMK特定配置
│   ├── Kconfig                      # Kconfig选项
│   ├── Kconfig.board                # 板子Kconfig定义
│   ├── Kconfig.defconfig            # 默认Kconfig配置
│   └── pre_dt_board.cmake           # 预处理设备树配置
├── config/                    # 键盘配置
├── build.yaml                # GitHub Actions构建矩阵配置
└── zephyr/module.yml         # Zephyr模块配置
```

## 构建说明

### 使用GitHub Actions构建

本仓库配置了GitHub Actions工作流，可以自动构建固件。当你推送代码到仓库或创建Pull Request时，工作流会自动运行。

构建矩阵在`build.yaml`文件中定义，目前配置为构建以下组合：

```yaml
include:
  - board: eyelash_nano
    shield: corne_left
  - board: eyelash_nano
    shield: corne_right
```

你可以根据需要修改这个文件，添加更多的板子和配列组合。

### 本地构建

要在本地构建固件，你需要设置ZMK开发环境。请参考[ZMK文档](https://zmk.dev/docs/development/setup)获取详细说明。

#### 配置west.yml

本仓库在`config/west.yml`中已经配置了必要的依赖：

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: eyelash
      url-base: https://github.com/a741725193
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-board-eyelash
      remote: eyelash
      revision: main
  self:
    path: config
```

这个配置确保west可以找到eyelash_nano板的定义。

#### 构建步骤

1. 克隆ZMK仓库
   ```bash
   git clone https://github.com/zmkfirmware/zmk.git
   cd zmk
   ```

2. 将本仓库作为用户配置添加到ZMK
   ```bash
   # 在ZMK根目录下
   mkdir -p app/boards/shields
   git clone https://github.com/a741725193/zmk-board-eyelash.git config
   ```

3. 运行`west update`更新所有依赖
   ```bash
   west init -l config
   west update
   ```

4. 使用west构建固件，指定eyelash_nano作为板子
   ```bash
   west build -b eyelash_nano -d build/corne_left -- -DSHIELD=corne_left
   west build -b eyelash_nano -d build/corne_right -- -DSHIELD=corne_right
   ```

#### 常见问题解决

如果遇到错误提示 "No board named 'eyelash_nano' found"，可能是以下原因：

1. west.yml配置不正确或未被正确加载
2. 未运行`west update`更新依赖
3. zmk-board-eyelash仓库未被正确克隆或路径不正确

解决方法：

1. 确认config/west.yml中包含了zmk-board-eyelash项目
2. 运行`west update`确保所有依赖都已更新
3. 检查boards/arm/eyelash_nano目录是否存在
4. 确保zephyr/module.yml中设置了正确的board_root

## 使用方法

### 配置键盘

在`config`目录中，你可以添加或修改键盘配置文件。对于Corne键盘，已经包含了基本配置：

- `corne.conf`: 键盘配置选项
- `corne.keymap`: 键映射定义

### 刷写固件

构建完成后，你可以使用以下方法之一刷写固件：

1. **UF2方式**：将键盘控制器置于bootloader模式，然后将生成的UF2文件拖放到出现的驱动器中。
2. **nrfjprog方式**：使用nRF命令行工具刷写固件。

## 许可证

本项目采用MIT许可证。详见源代码文件中的许可声明。

## 贡献

欢迎提交Issue和Pull Request来改进这个项目。