# TestSQS

## 设计目标

- 创建一个 SQS 队列，名称TestServerless，使用 SQS 触发器触发 Lambda 函数
- 创建一个 Dispatcher Lambda 函数，用于向 SQS 队列发送 数据
- 创建一个 Workder Lambda 函数，消费 SQS 消息并回调 Step Functions；各阶段耗时通过消息/回调 Output 传回客户端（不再使用 DynamoDB 记录）
- 创建一个 StepFuntions 流程，阻塞等待 Worker Lambda 的完成信号。
- 创建一个 API Gateway，触发 StepFunctions 流程的执行，并同步等待流程完成后返回
- 编写一个 StepFunctions_test.go 测试模块，用于测试整个系统的功能和性能；耗时拆分不依赖 DynamoDB，改为从 callback Output 的时间戳计算；状态机总耗时由 Start/Stop 计算。
- 生成一个 Readme.md 来讲解如何使用该项目，并画图展示程序架构。

## 编写要求

- 使用 sam 进行构建和部署，使用Dockerfile进行本地构建
- 使用 Go 语言编写 Lambda 函数
- 使用 AWS SDK for Go v2
- SQS 和 Lambda 函数必须在同一个 AWS Region（`aws_region`）
- Lambda 架构需要与镜像构建架构一致（`x86_64`/`arm64` 均可）；本项目已在模板里做成参数 `FunctionArchitecture` 可配置，避免不匹配导致运行时报错。
- 在测试日志中增加每条消息发送和接收的时间戳和消息队列名字。
- 将 AWS SDK 客户端移到 handler 外部初始化，可减少热启动时的开销。
- 参照现有代码的风格进行编写，包括命名规范、错误处理和日志记录等。新生成代码以后，删除多余文件。

## 测试用例

- 测试整个状态机流程是否能够正确执行
- 测试整个状态的响应时间，完成整个状态机后进行下一次测试。一共测试 10 次，计算平均响应时间
