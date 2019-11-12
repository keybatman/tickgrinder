# TickGrinder Algorithmic Trading Platform

[![Join the chat at https://gitter.im/TickGrinder/Lobby](https://badges.gitter.im/TickGrinder/Lobby.svg)](https://gitter.im/TickGrinder/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
![](https://camo.githubusercontent.com/79318781f189b2ee91c3a150bf27813c386afaf2/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f72757374632d6e696768746c792d79656c6c6f772e737667)
![](https://tokei.rs/b1/github/Ameobea/tickgrinder)
![](https://tokei.rs/b1/github/Ameobea/tickgrinder?category=files)

TickGrinder is a high performance algorithmic trading platform written primarily in Rust.  It is designed with the goal of efficiently processing event-based market data as quickly as possible in order to automatically place and manage trades.

*Currently this platform is only compiles and runs on Linux-based systems.  Windows functionality is planned for the future but no set schedule has been defined for its implementation.*

# Overview
The basis of the platform is written in Rust.  It consists of several distinct modules that operate independently but communicate with each other via a custom messaging protocol implemented on top of Redis Pub/Sub.  It is designed to be extensible and robust, capable of being used to trade any market consisting of event-based streaming Tick data.

Disclaimer: **This platform is currently in active, early pre-alpha development and is in no way ready for any kind of trading.**  I do not take any responsibility for your trading actions using this platform nor any financial losses caused by errors in this application.

## Tick Processors
The primary module is the Tick Processor.  Multiple tick processors can be spawned, one for each symbol/data stream that is being processed by the platform.  Their purpose is to convert live data into trading signals as quickly as possible.

Each time a tick arrives, a series of conditions are evaluated.  These conditions can be anything: a SMA crossing a threshold, the current timestamp being greater than a certain number, any evaluable expression that you can program can serve as a condition.  These conditions are designed to be dynamically set by the Optimizer module during trading operations.

## Optimizer/Strategies
The Optimizer is a module that controls the conditions evaluated by the Tick Processors to open trades and interact with the broker API.  Only one Optimizer is meant to run at once and it interacts with all Tick Processors that may be alive in the platform.

Optimizers, using whatever strategies you define, set the parameters and enable/disable trading conditions on the Tick Processors dynamically.  They accomplish this by sending and receiving Commands to and from the Tick Processors, interacting with the database, or using any external data sources you may find useful.  This application currently uses PostgreSQL as the main storage backend.

The strategies that are evaluated by the optimizer are meant to be written by you, the user!  I'm keeping my strategies secret, but I will provide a sample strategy in the future for reference.

## MM (Management/Monitoring) Interface
The MM is the interface between the platform and the user.  It contains controls for manually managing platform components, monitoring bot activity, starting backtests, and pretty much everything else.  It exists in the format of a simple NodeJS Express webserver and talks with the main communication system of the platform using a Websocket<->Redis bridge system.

It includes a custom charting module using Highcharts that can be extended with custom indicators to produce truly specialized charts for bot data.  The configurations for these charts can be compiled down into JSON-encoded Macro strings which can then be loaded to instantly re-create charts with unique data sources and settings.

## Broker APIs + Data Downloaders
Several scripts/utilities are included to interact with the FXCM Foreign Exchange API to create a live data feed and access historical market data.  Using scripts located in the `tick_writer` and `data_downloaders` directories, this application can be controlled through Redis and used as a primary data source for the trading platform.

The data downloader script saves tick data for a currency pair into CSV files, downloading chunks of data automatically.  The tick writer script does the same thing but processes live ticks sent over Redis instead.  As I mentioned before, more detailed documentation can be found in the respective directories.

# Installation
**Requirements**:
* Rust/Cargo Nightly
* NodeJS v5.0.0 or greater
* Redis
* PostgreSQL
* libboost headers (`sudo apt-get install libboost-all-dev`)

After cloning the repository, you'll need to copy all instances of files named `conf.default.rs`, `conf.sample.js`, or anything similar to `conf.js/rs` in the same directory and fill out their values as appropriate.

After ensuring that you have a Redis and PostgreSQL server running and accessible to the program, you can make sure that everything is set up correctly by running `make test` in the root directory of the project.  This will automatically pull down all needed dependencies, attempt to build all modules, and run all included tests.  Any encountered errors will be printed to the console and you can use them to debug any issues you're having with the setup.

To use the live trading and data downloading functionalities, you'll need to set up the FXCM Java Application (located in the TickRecorder directory).  Detailed setup and installation instructions for that are located in that folder.  Documentation for the data downloader and `tick_writer` scripts can also be found in their respective directories.

# Contributing
This is an open source project project, so you're more than welcome to fork it and customize it for your own needs.

As for contributing to master, I'm very happy to merge pull requests containing syntax improvements, bugfixes, or small stuff like that.  However, for more significant things such as rewriting large segments of code or adding new features, please file an Issue or contact me privately before putting a lot of work in.

In addition, I will not accept any pull requests solely consisting of stylistic changes such as running files through rustfmt or linting the platform with Clippy.  I'm aware that some of the choices in syntax and style I've used aren't 100% "rusty," but I made the choices I made for a reason.  Things like fixing typos, adding in more clear or detailed comments/documentation, or re-implementing functions in a more concise or efficient method are very welcome, however.

# Closing Remarks
If you've got any feedback or comments on the project, I'd love to hear it!  I'm always working on developing my skills as a programmer, so any sage advice from seasoned veterans (or questions from eager beginners) are very welcome.

If you find this project useful, exciting, or have plans to use this in production, **please** let me know!  I'd maybe be willing to work with you to make sure that your needs are met and improve the platform in the process.

TickGrinder是一个高性能的算法交易平台，主要在 Rust 中编写。 为了自动地处理和管理交易，本文设计了基于事件的市场数据快速处理方法。

目前这个平台只在基于linux的系统上编译和运行。 为将来规划 Windows 功能，但还没有为它的实现定义任何设置计划。

## 概述
这个平台的基础是用 Rust 编写的。 它由几个独立的模块独立运行，但是通过定制的消息协议在 Redis pub/平台上实现。 它被设计为可以扩展性和鲁棒性，能够用于交易任何基于事件的流令牌数据市场。

免责声明：*平台目前处于活跃状态，早期的alpha开发，不适合任何种类的交易。* 如果你使用这里平台，我不承担任何责任，也不承担这里应用程序中错误造成的任何财务损失。

## 处理模块
主 MODULE 是Tick处理器。 可以生成多个分号处理器，每个符号/数据流都可以由平台处理。 他们的目的是尽可能快地把实时数据转换成交易信号。

每次到达时，都会评估一系列条件。 这些条件可以是任何情况： 通过一个阈值，当前时间戳是 GREATER 而不是某个数字，可以编程的任何evaluable表达式都可以用作条件。 这些条件被设计成在交易操作期间由优化器 MODULE 动态设置。

## 优化器/策略
优化器是一个 MODULE，它控制由with处理器评估的条件以打开交易并与代理API交互。 只有一个优化器同时运行，它与所有可以能在平台中活动的处理器交互。

优化器，使用你定义的任何策略，设置参数，并在标记处理器上动态启用/禁用交易条件。 它们通过发送和接收令牌处理器，与数据库交互，或者使用任何有用的外部数据源来完成。 这个应用程序目前使用PostgreSQL作为主要的存储后端。

优化器计算的策略应该由你编写，但我将在未来提供一个示例策略来参考。

## (管理/监视) 接口
MM是平台与用户之间的接口。 它包含手动管理平台组件。监控bot活动。启动backtests和几乎所有其他组件的控件。 系统采用简单的NodeJS Express web server格式，使用 web socket <-> Redis桥接系统与平台的主通信系统进行通信。

它包括一个使用for的自定义图表 MODULE，可以扩展自定义指示器来生成数据的专用图表。 可以将这些图表的配置编译成json编码的宏字符串，然后将它的加载到特定的数据源和设置。

## 代理 api + 数据下载程序
包括几个脚本/工具来与FXCM外汇API交互，创建实时数据提要并访问历史市场数据。 这个应用程序可以通过 tick_writer 和 data_downloaders 目录中的脚本，通过Redis控制，并作为交易平台的主要数据源使用。

下载数据下载程序脚本将货币对的刻度数据保存到CSV文件中，自动下载数据块。 tick脚本执行相同的操作，但是处理通过Redis发送的动态滴答。 如前所述，在各个目录中可以找到更详细的文档。

## 安装
- 需求:
Rust/Cargo
NodeJS v5.0.0或者更高版本
Redis
PostgreSQL
libboost标头( sudo apt-get install libboost-all-dev )

在克隆了存储库之后，需要复制名为 conf.default.rs。conf.sample.js 或者相同目录中的任何类似 conf.js/rs的文件。

确保运行并可以访问程序后，可以通过在项目的root 目录中运行 make test 确保正确设置了一切。 这将自动关闭所有需要的依赖项，尝试构建所有模块，并运行所有包含的测试。 任何遇到的错误都会打印到控制台，你可以使用它们来调试你使用安装的任何问题。

使用实时交易和数据下载功能，你需要设置 FXCM Java应用程序( 位于TickRecorder目录中)。 详细的安装和安装说明位于该文件夹中。 数据下载器和 tick_writer 脚本的文档也可以在各自的目录中找到。

这是一个开放源码项目项目，所以你非常欢迎 fork 并自定义你自己的需求。

我非常乐意合并拉请求，包含语法改进，错误修改，或者小东西等。 然而，如果重写大部分代码或者添加新特性，请在放置许多工作之前与我联系。

除了通过rustfmt或者linting运行文件，并使用 Clippy，我不接受任何由样式更改构成的请求请求。 我知道，我使用的语法和风格的一些选择不是 100%"rusty"，但是我做了一个原因做出了选择。 非常欢迎修复错误，添加更清晰或者详细的评论/文档，或者在更简单或者有效的方法中实现。

## 结束语
如果你对这个项目有任何反馈或者意见，我很愿意听到！

如果你觉得这个项目有用，令人兴奋，或者计划在生产中使用这个，请让我知道。 我可以能愿意与你一起工作，以确保满足你的需求并提高过程中的平台。
