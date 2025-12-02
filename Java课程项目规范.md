# Java局域网聊天程序开发规范

## 一、全局基础规范

### 1. 环境与编码规范

- 编码格式：所有文件统一UTF-8
- 换行与缩进：
  - 缩进：4个空格（禁止Tab），IDE需设置“制表符转为空格”；
  - 大括号位置：类、方法、循环/条件语句的左大括号紧跟声明后（不换行），右大括号单独占一行；
- 依赖限制：仅使用JDK原生类，允许使用的包明确：
  - 服务器端：java.net、java.io、java.util；
  - 客户端：java.net、java.io、java.util、javax.crypto、javax.swing；
  - 禁止引入任何第三方Jar包（如commons-codec、fastjson等）。

### 2. 包名与目录结构规范
```
com.chat                       // 根包（所有类统一在此包下）
├─ server                      // 开发者1专属包（服务器端）
│  ├─ ServerCore.java          // 服务器主类（启动、监听、核心逻辑）
│  ├─ ClientHandler.java       // 客户端连接处理线程类
│  └─ ServerConstants.java     // 服务器端常量类
├─ client                      // 客户端相关包
│  ├─ util                     // 开发者2专属包（连接+加密工具）
│  │  ├─ NetworkClient.java    // 网络通信抽象接口（新增）
│  │  ├─ ClientSocketUtil.java // Socket通信实现类（原工具类适配接口）
│  │  ├─ AESUtil.java          // AES加解密工具类
│  │  └─ ClientConstants.java  // 客户端常量类
│  ├─ ui                       // 开发者3专属包（UI组件）
│  │  ├─ ChatClientUI.java     // 客户端UI主类（单例，实现UICallback接口）
│  │  └─ UIConstants.java      // UI常量类
│  └─ logic                    // 开发者4专属包（逻辑衔接）
│     ├─ ChatClientLogic.java  // 逻辑主类（单例，依赖抽象接口）
│     ├─ MessageParser.java    // 消息解析工具类
│     └─ UICallback.java       // UI回调抽象接口（新增）
└─ common                      // 公共常量包（开发者2维护，全员引用）
    └─ Constants.java          // 全局公共常量
```
- 约束：禁止跨包存放类，禁止跨包引用private方法/变量，对外提供的功能必须通过public接口暴露；
- 适配约束：核心逻辑层（logic）仅依赖抽象接口（NetworkClient、UICallback），不依赖具体实现类（ClientSocketUtil、ChatClientUI），为后续前端/通信协议切换预留扩展入口。

### 3. 命名规范
| 类型         | 命名规则                                  | 示例                                  | 禁止示例                          |
|--------------|-------------------------------------------|---------------------------------------|-----------------------------------|
| 类名         | 帕斯卡命名法，体现功能，无模糊命名        | ServerCore、ChatClientUI、AESUtil     | Server、UI、Tool                  |
| 接口名       | 帕斯卡命名法，以功能为核心，可选前缀“I”    | NetworkClient、UICallback、IConnection | Callback、Client                  |
| 方法名       | 驼峰命名法，动词开头，明确操作对象        | startServer()、encryptContent()、appendChatMessage() | doSomething()、fun()、send()      |
| 变量名       | 驼峰命名法，见名知义，禁止拼音/缩写（循环变量i/j/k除外） | onlineUserMap、targetNickname、localIp | nicheng、usr、ipAddr（应为ipAddress） |
| UI组件名     | 前缀+功能（驼峰），前缀固定               | lblLocalIp、txtIp、txtNickname、btnConnect、listOnlineUsers | localIpLabel、ipInput、sendBtn    |
| 常量名       | 全大写+下划线分隔，格式“模块_功能_描述”    | COMMON_MESSAGE_SEPARATOR、CLIENT_DEFAULT_IP | messageSeparator、defaultIp       |
| 线程名       | 格式“模块-功能-序号/标识”                  | "Server-Accept-Thread"、"Client-Handler-张三" | "Thread1"、"接收线程"              |

### UI组件前缀固定表
| 组件类型       | 前缀 | 应用场景                  |
|----------------|------|---------------------------|
| JLabel         | lbl  | 文本标签（如本机IP显示、提示） |
| JTextField     | txt  | 输入框（如昵称、IP、端口、消息输入） |
| JButton        | btn  | 按钮（如连接、发送、断开） |
| JList          | list | 列表组件（如在线用户列表） |
| JRadioButton   | rdo  | 单选按钮（如群聊/私聊切换） |
| JTextArea      | txt  | 文本区域（如消息显示区） |
| JPanel         | pnl  | 面板（如连接面板、聊天面板） |

### 4. 代码注释规范
- 类注释：必须包含“作者、创建日期、功能描述、核心依赖、注意事项”，语言为中文；
- 接口注释：除类注释内容外，需额外说明“接口用途、实现类要求、方法调用约束”，明确适配场景；
- 公共方法注释：必须包含“功能描述、参数说明（类型+约束）、返回值说明、异常说明（若有）”，使用Javadoc格式；
- 行内注释：以下场景必须加（禁止冗余注释）：
  1. 复杂逻辑分支（如消息解析的类型判断）；
  2. 关键参数赋值（如密钥处理、超时设置）；
  3. 异常处理逻辑（处理思路说明）；
  4. 接口适配相关代码（如回调触发、接口绑定）；
- 注释禁忌：禁止注释显而易见的代码（如`i++;//i自增`），禁止注释过时代码（直接删除）。

### 5. 常量规范
#### 公共常量（com.chat.common.Constants）
| 常量名称                | 类型   | 取值/规则                                  | 用途                          |
|-------------------------|--------|-------------------------------------------|-------------------------------|
| MESSAGE_SEPARATOR       | String | "#"                                       | 消息分隔符（不可修改）        |
| MAX_MESSAGE_LENGTH      | int    | 1000                                      | 单条消息最大长度（不含消息头） |
| MESSAGE_TYPE_GROUP      | String | "GROUP"                                   | 群聊消息类型标识              |
| MESSAGE_TYPE_PRIVATE    | String | "PRIVATE"                                 | 私聊消息类型标识              |
| MESSAGE_TYPE_USER_LIST  | String | "USER_LIST"                                | 用户列表更新消息标识          |
| MESSAGE_TYPE_CONNECT_SUCCESS | String | "CONNECT_SUCCESS"                   | 连接成功响应标识              |
| MESSAGE_TYPE_CONNECT_FAILED | String | "CONNECT_FAILED"                     | 连接失败响应标识              |
| MESSAGE_TYPE_DISCONNECT | String | "DISCONNECT"                               | 断开通知消息标识              |
| DEFAULT_PORT            | int    | 8888                                      | 默认监听/连接端口             |
| CONNECT_TIMEOUT         | int    | 5000                                      | 连接超时时间（毫秒）          |
| AES_KEY                 | String | "chatroom_key_123"                            | AES加密密钥（固定，禁止修改）  |
| BASE64_CHARSET          | String | "UTF-8"                                    | Base64编码字符集              |
| NICKNAME_MIN_LENGTH     | int    | 1                                         | 昵称最小长度                  |
| NICKNAME_MAX_LENGTH     | int    | 10                                        | 昵称最大长度                  |
| NICKNAME_FORBIDDEN_CHARS | String | "#,"                                     | 昵称禁止字符（不可包含）      |

#### 服务器端常量（com.chat.server.ServerConstants）
| 常量名称                | 类型   | 取值/规则                                  | 用途                          |
|-------------------------|--------|-------------------------------------------|-------------------------------|
| MAX_CLIENT_COUNT        | int    | 50                                        | 最大客户端连接数              |
| SERVER_THREAD_PREFIX    | String | "Client-Handler-"                          | 客户端处理线程名称前缀        |

#### 客户端常量（com.chat.client.util.ClientConstants）
| 常量名称                | 类型   | 取值/规则                                  | 用途                          |
|-------------------------|--------|-------------------------------------------|-------------------------------|
| DEFAULT_IP              | String | "127.0.0.1"                                | 客户端默认连接IP              |
| LOCAL_IP_NOT_CONNECTED  | String | "未接入局域网"                              | 未联网时的IP提示文本          |
| LOCAL_IP_TIP_PREFIX     | String | "本机IP："                                  | IP显示前缀                    |

#### UI常量（com.chat.client.ui.UIConstants）
| 常量名称                | 类型   | 取值/规则                                  | 用途                          |
|-------------------------|--------|-------------------------------------------|-------------------------------|
| WINDOW_WIDTH            | int    | 800                                       | 窗口宽度（像素）              |
| WINDOW_HEIGHT           | int    | 600                                       | 窗口高度（像素）              |
| COLOR_GROUP_CHAT        | Color  | Color.BLACK                               | 群聊消息颜色                  |
| COLOR_PRIVATE_CHAT      | Color  | Color.BLUE                                | 私聊消息颜色                  |
| COLOR_TIP               | Color  | Color.GRAY                                | 普通提示文本颜色              |
| COLOR_ERROR             | Color  | Color.RED                                 | 错误提示文本颜色              |
| BTN_CONNECT_TEXT        | String | "连接"                                    | 连接按钮默认文本              |
| BTN_DISCONNECT_TEXT     | String | "断开"                                    | 连接按钮断开状态文本          |
| BTN_SEND_TEXT           | String | "发送"                                    | 发送按钮文本                  |
| RDO_GROUP_CHAT_TEXT     | String | "群聊"                                    | 群聊单选按钮文本              |
| RDO_PRIVATE_CHAT_TEXT   | String | "私聊"                                    | 私聊单选按钮文本              |
| TXT_NICKNAME_HINT	      | String | "请输入昵称（1-10 字符，不含 #和逗号）"	  | 昵称输入框提示文本            |
| TXT_IP_HINT             | String | "请输入服务器IP"                           | IP输入框提示文本              |
| TXT_PORT_HINT           | String | "请输入端口（默认8888）"                    | 端口输入框提示文本            |
| TXT_MSG_INPUT_HINT      | String | "请输入消息（最多1000字符）"                | 消息输入框提示文本            |

### 6. 异常处理规范（明确处理原则和要求）
- 捕获原则：仅捕获已知异常（如SocketException、IOException），禁止捕获无处理逻辑的Exception；
- 处理方式分模块明确：
  1. 工具类（开发者2）：捕获异常后，通过返回值（false、null、特定字符串）告知调用方，同时输出规范日志，禁止抛出未捕获异常；
  2. 服务器端（开发者1）：捕获异常后，输出ERROR日志，关闭对应Socket资源，避免影响其他客户端；
  3. 客户端逻辑层（开发者4）：捕获异常后，通过UICallback接口通知UI显示友好提示（不暴露技术细节），同时输出日志；
  4. UI层（开发者3）：捕获事件处理异常，输出日志，确保UI不崩溃；
- 日志输出格式：`[时间] [模块] [级别] 内容`，异常日志需包含“异常类型、异常信息、堆栈信息”。

### 7. 线程安全规范（明确要求，避免隐患）
- 服务器端：
  1. 在线用户列表必须使用ConcurrentHashMap（线程安全），禁止使用HashMap；
  2. 多线程操作共享资源（如用户列表）需确保原子性；
  3. 每个客户端连接对应独立的ClientHandler线程，线程需设置规范名称；
- 客户端：
  1. UI更新必须在EDT线程中执行（使用SwingUtilities.invokeLater()）；
  2. 接收消息线程需设为守护线程（setDaemon(true)）；
  3. 工具类（ClientSocketUtil、AESUtil）需设计为无状态静态类或接口实现类，确保线程安全；
  4. 接口实现类的多线程调用需保证线程安全，禁止共享非线程安全对象；
- 禁止做法：禁止多线程共享非线程安全对象（如ArrayList），禁止在UI线程中执行阻塞操作（如Socket接收）。

### 8. 消息格式规范（全员统一，不可修改）
| 消息类型                | 格式模板                                  | 说明                                  |
|-------------------------|-------------------------------------------|---------------------------------------|
| 群聊消息                | GROUP#发送者昵称#消息内容                  | 服务器接收后广播给所有在线客户端      |
| 私聊消息                | PRIVATE#发送者昵称#接收者昵称#消息内容     | 服务器仅转发给目标昵称客户端          |
| 用户列表更新消息        | USER_LIST#用户1,用户2,用户3（逗号分隔）    | 服务器在客户端连接/断开时广播        |
| 连接成功响应消息        | CONNECT_SUCCESS#昵称                       | 服务器返回给客户端，确认连接成功      |
| 连接失败响应消息        | CONNECT_FAILED#失败原因                    | 服务器返回给客户端，说明连接失败原因  |
| 断开通知消息            | DISCONNECT#昵称                            | 客户端断开时，服务器广播给其他客户端  |

- 约束：
  1. 消息拆分必须使用`String.split(MESSAGE_SEPARATOR, -1)`（保留空参数）；
  2. 昵称禁止包含逗号和#，消息内容长度≤MAX_MESSAGE_LENGTH（1000字符）；
  3. 所有消息传输时，加密后需用Base64编码（避免特殊字符乱码）；
  4. 消息格式为前端与服务器、客户端各模块的核心契约，禁止任何模块自定义格式。

### 9. 层间解耦与适配规范（新增，支持前端切换）
- 核心原则：**UI层 ↔ 逻辑层 → 工具层 → 服务器端**，仅通过规范定义的抽象接口交互，禁止层间直接依赖具体实现类；
- 接口契约优先级：抽象接口（NetworkClient、UICallback）的方法名、参数类型、返回值、异常反馈为核心契约，一旦确定禁止擅自修改；
- 适配入口：后续切换前端（如Web、其他GUI框架）或通信协议（如WebSocket）时，仅需：
  1. 实现UICallback接口（接收逻辑层通知）；
  2. 实现NetworkClient接口（处理网络通信）；
  3. 按消息格式规范收发数据；
  无需修改逻辑层、工具层、服务器端核心代码；
- 依赖倒置要求：高层模块（逻辑层）依赖抽象接口，不依赖底层具体实现（如Socket、Swing组件）；底层实现类（ClientSocketUtil、ChatClientUI）依赖并实现抽象接口。

## 二、开发者分工专属规范（明确职责、接口、约束）

### （一）开发者1：服务器端核心（com.chat.server包）

#### 1. 核心职责
- 服务器启动/停止，监听指定端口；
- 客户端连接管理（接入、断开、数量限制）；
- 在线用户列表维护（添加、移除、广播更新）；
- 消息解析与转发（群聊广播、私聊定向转发）；
- 服务器稳定性保障（资源释放、线程安全、异常处理）；
- 兼容多客户端前端/通信协议（仅依赖消息格式规范，不绑定具体客户端实现）。

#### 2. 核心类接口规范（仅定义接口，无实现）
##### （1）ServerCore类（单例模式）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| getInstance()         | 无                                        | ServerCore | 获取单例实例                              | 必须使用双重检查锁实现单例，确保线程安全    |
| startServer(int port) | port：监听端口（1-65535）                  | void       | 启动服务器，监听指定端口                  | 端口非法抛出IllegalArgumentException；端口占用输出日志并启动失败；启动后输出INFO日志 |
| stopServer()          | 无                                        | void       | 停止服务器                                | 需关闭ServerSocket和所有客户端Socket，清空用户列表，输出INFO日志 |
| broadcastMessage(String message, Socket excludeSocket) | message：待广播消息；excludeSocket：需排除的客户端Socket | void | 广播消息（排除指定客户端） | 消息为空不处理；仅转发符合格式的消息 |
| sendPrivateMessage(String message, String targetNickname) | message：私聊消息；targetNickname：目标用户昵称 | boolean | 定向发送私聊消息 | 目标用户离线返回false；消息格式错误返回false |
| addOnlineUser(String nickname, Socket socket) | nickname：用户昵称；socket：客户端Socket | boolean | 添加在线用户 | 昵称已存在返回false；参数非法返回false；添加成功后触发用户列表广播 |
| removeOnlineUser(String nickname) | nickname：用户昵称 | void | 移除在线用户 | 移除后关闭对应Socket，触发用户列表广播 |
| broadcastUserList()    | 无                                        | void       | 广播在线用户列表                          | 按USER_LIST消息格式拼接用户列表，广播给所有客户端 |

##### （2）ClientHandler类（线程类）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| 构造方法              | Socket socket：客户端Socket；ServerCore serverCore：服务器实例 | - | 初始化客户端处理线程 | 必须设置线程名称（SERVER_THREAD_PREFIX + 昵称） |
| run()                 | 无                                        | void       | 线程执行逻辑（接收消息、处理消息） | 需校验昵称合法性；接收客户端消息并转发；客户端断开时调用removeOnlineUser；异常处理后释放资源 |
| validateNickname(String nickname) | nickname：待校验昵称 | boolean | 校验昵称合法性 | 按NICKNAME_MIN_LENGTH、NICKNAME_MAX_LENGTH、NICKNAME_FORBIDDEN_CHARS校验 |
| handleClientMessage(String message) | message：客户端消息 | void | 处理客户端消息 | 解析消息类型，按群聊/私聊分别转发；格式错误输出ERROR日志 |

#### 3. 约束与禁忌
- 禁止使用非线程安全的集合存储在线用户；
- 禁止未处理异常导致服务器崩溃；
- 禁止客户端断开后未释放Socket资源；
- 禁止消息格式解析错误时影响其他客户端；
- 必须保证昵称唯一（同一昵称无法重复连接）；
- 禁止绑定具体客户端通信协议（如Socket），仅按消息格式处理数据，支持后续客户端协议扩展。

#### 4. 自测标准（可验证的场景）
| 测试场景                | 操作步骤                                  | 预期结果                                  |
|-------------------------|-------------------------------------------|-------------------------------------------|
| 端口非法测试            | 调用startServer(0)或startServer(65536)     | 抛出IllegalArgumentException，服务器不启动 |
| 端口占用测试            | 启动两个服务器实例，监听同一端口           | 第二个实例启动失败，输出“端口已被占用”日志 |
| 昵称重复测试            | 两个客户端使用相同昵称连接                | 后连接客户端收到CONNECT_FAILED响应，连接失败 |
| 群聊转发测试            | 客户端发送GROUP格式消息                   | 所有在线客户端均收到该消息                |
| 私聊转发测试            | 客户端发送PRIVATE格式消息给目标用户       | 仅目标用户收到消息，其他客户端无响应      |
| 最大连接数测试          | 启动超过MAX_CLIENT_COUNT（50）个客户端连接 | 超出的客户端收到“服务器繁忙”响应，连接失败 |
| 异常断开测试            | 强制关闭客户端进程                        | 服务器移除该用户，广播用户列表更新        |

### （二）开发者2：客户端连接+加密工具（com.chat.client.util包）
#### 1. 核心职责
- 定义网络通信抽象接口（NetworkClient），屏蔽具体协议差异；
- 实现Socket通信协议（ClientSocketUtil实现NetworkClient接口）；
- AES加解密工具封装（加密、解密、Base64编码）；
- 本机局域网IP获取工具实现；
- 工具类/接口实现类的线程安全、异常处理、兼容性保障；
- 常量维护（公共常量、客户端专属常量）；
- 预留后续通信协议扩展入口（如WebSocket，仅需新增NetworkClient接口实现类）。

#### 2. 核心接口与类规范
##### （1）NetworkClient接口（新增，抽象网络通信能力）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| connect(String ip, int port, String nickname) | ip：服务器IP；port：服务器端口；nickname：客户端昵称 | boolean | 连接服务器 | 校验IP、端口、昵称合法性；连接超时时间为CONNECT_TIMEOUT（5秒）；成功返回true，失败返回false；不抛异常 |
| sendMessage(String message) | message：待发送消息（符合全局消息格式，已加密+Base64编码） | boolean | 发送消息到服务器 | 未连接返回false；消息为空返回false；发送成功返回true，失败返回false；不抛异常 |
| receiveMessage()       | 无                                        | String | 接收服务器消息 | 未连接返回null；接收超时返回空字符串；接收失败返回null；返回结果为“加密+Base64编码”的原始字符串 |
| disconnect()           | 无                                        | void       | 断开与服务器的连接                          | 关闭网络资源（Socket/流等），重置连接状态；输出INFO日志 |
| isConnected()          | 无                                        | boolean    | 获取当前连接状态                          | 实时返回连接是否有效（资源未释放且连接正常） |
| getLocalIp()           | 无                                        | String | 获取本机局域网IPv4地址                      | 成功返回IP字符串；未联网返回LOCAL_IP_NOT_CONNECTED；输出INFO日志 |

##### （2）ClientSocketUtil类（原工具类，修改为NetworkClient接口实现类）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| getInstance()         | 无                                        | NetworkClient | 获取单例实例 | 单例模式，确保全局唯一网络通信实例；禁止实例化多个对象 |
| connect(String ip, int port, String nickname) | 同NetworkClient接口 | boolean | 实现Socket连接逻辑 | 内部调用validateConnectParams()校验参数；设置Socket超时；连接成功后初始化输入输出流 |
| sendMessage(String message) | 同NetworkClient接口 | boolean | 实现Socket发送逻辑 | 先通过isConnected()校验状态；发送失败触发disconnect()；不抛异常 |
| receiveMessage()       | 同NetworkClient接口 | String | 实现Socket接收逻辑 | 循环读取输入流；捕获Socket异常返回null；不阻塞主线程 |
| disconnect()           | 同NetworkClient接口 | void | 实现Socket断开逻辑 | 按“输入流→输出流→Socket”顺序关闭资源；重置成员变量 |
| isConnected()          | 同NetworkClient接口 | boolean | 实现连接状态判断 | 判定条件：Socket非空、未关闭、已连接、流未关闭 |
| getLocalIp()           | 同NetworkClient接口 | String | 实现本机IP获取 | 逻辑与原工具类一致，适配接口返回要求 |
| validateConnectParams(String ip, int port, String nickname) | ip：服务器IP；port：端口；nickname：昵称 | boolean | 校验连接参数合法性 | 按常量规范校验端口（1-65535）、昵称；IP非空校验；非法返回false并输出日志 |

##### （3）AESUtil类（静态工具类，禁止实例化）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| encrypt(String content) | content：待加密内容（可为空） | String | AES加密，返回Base64编码结果 | 内容为空返回空字符串；加密失败返回“[加密失败]”；密钥使用Constants.AES_KEY，算法模式AES/ECB/PKCS5Padding |
| decrypt(String encryptedContent) | encryptedContent：加密后的Base64字符串（可为空） | String | AES解密，返回原始内容 | 内容为空返回空字符串；解密失败返回“[解密失败]”；先Base64解码再解密 |

#### 3. 约束与禁忌
- 接口方法禁止擅自修改（方法名、参数、返回值），如需扩展需通过接口继承；
- 实现类必须完全实现接口所有方法，禁止遗漏；
- 工具类/实现类必须设计为无状态或单例，确保线程安全；
- 禁止抛出未捕获异常，所有异常内部处理并通过返回值告知调用方；
- 禁止加密/解密时不处理特殊字符（如空字符串、超长字符串）；
- 禁止连接时不设置超时，导致无限阻塞；
- 禁止未关闭网络资源导致泄露；
- 禁止在实现类中添加接口外的公共方法，避免调用方依赖具体实现。

#### 4. 自测标准
| 测试场景                | 操作步骤                                  | 预期结果                                  |
|-------------------------|-------------------------------------------|-------------------------------------------|
| 接口实现完整性测试      | 检查ClientSocketUtil是否实现NetworkClient所有方法 | 无遗漏方法，所有方法符合接口定义 |
| 加密解密一致性测试      | 输入任意字符串（含空、特殊字符），调用encrypt后调用decrypt | 解密结果与原始内容一致                    |
| 连接参数非法测试        | 输入非法IP（如“256.0.0.1”）、非法端口（如88888）、非法昵称（含#） | connect()返回false，输出对应错误日志 |
| 连接超时测试            | 服务器未启动，调用connect()                | 5秒后返回false，输出“连接超时”日志         |
| 断开连接测试            | 连接成功后调用disconnect()                 | 连接状态isConnected()返回false，输出“已断开连接”日志 |
| 本机IP获取测试          | 连接/断开局域网，调用getLocalIp()          | 联网时返回正确IP，断网时返回指定提示文本   |

### （三）开发者3：客户端UI（com.chat.client.ui包）
#### 1. 核心职责
- 客户端Swing UI组件设计与实现（按规范布局）；
- 实现UICallback接口，接收逻辑层的状态/消息通知（适配前端切换）；
- UI组件命名与接口暴露（提供getter方法）；
- 基础交互逻辑实现（按钮状态切换、输入校验、列表选择）；
- UI样式统一（颜色、字体、大小、间距）；
- 线程安全的UI更新（严格在EDT线程中执行）；
- 本机IP显示、消息区分显示（群聊/私聊颜色）；
- 所有用户操作通过逻辑层接口触发，禁止直接调用工具层。

#### 2. 核心类接口规范
##### ChatClientUI类（单例模式，实现UICallback接口）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| getInstance()         | 无                                        | ChatClientUI | 获取单例实例 | 必须使用单例模式，确保唯一UI窗口；实例化后自动调用initUI()和绑定逻辑层回调 |
| initUI()              | 无                                        | void | 初始化UI组件、布局、默认状态 | 组件齐全（按目录结构要求）；布局使用BorderLayout+FlowLayout，禁止绝对布局；设置窗口大小为WINDOW_WIDTH×WINDOW_HEIGHT |
| getTxtNickname()	    | 无	                                    | JTextField  |  获取昵称输入框组件	 |  默认提示文本为 TXT_NICKNAME_HINT；输入长度限制为 NICKNAME_MIN_LENGTH~NICKNAME_MAX_LENGTH（1-10 字符）；禁止输入 NICKNAME_FORBIDDEN_CHARS（# 和逗号）|
| getTxtIp()            | 无                                        | JTextField | 获取IP输入框组件 | 默认值为DEFAULT_IP；提示文本为TXT_IP_HINT |
| getTxtPort()          | 无                                        | JTextField | 获取端口输入框组件 | 默认值为DEFAULT_PORT；提示文本为TXT_PORT_HINT |
| getBtnConnect()       | 无                                        | JButton | 获取连接/断开按钮 | 默认文本为BTN_CONNECT_TEXT；连接成功后改为BTN_DISCONNECT_TEXT |
| getListOnlineUsers()  | 无                                        | JList<String> | 获取在线用户列表组件 | 单选模式；默认空列表；用户列表更新时需在EDT线程中执行 |
| getTxtMsgDisplay()    | 无                                        | JTextArea | 获取消息显示区组件 | 不可编辑；自动换行；带垂直滚动条 |
| getTxtMsgInput()      | 无                                        | JTextField | 获取消息输入框组件 | 提示文本为TXT_MSG_INPUT_HINT；回车触发发送；输入长度≤MAX_MESSAGE_LENGTH |
| getBtnSend()          | 无                                        | JButton | 获取发送按钮 | 默认禁用（连接成功后启用）；文本为BTN_SEND_TEXT；输入为空时禁用 |
| getRdoGroupChat()     | 无                                        | JRadioButton | 获取群聊单选按钮 | 默认选中；文本为RDO_GROUP_CHAT_TEXT |
| getRdoPrivateChat()   | 无                                        | JRadioButton | 获取私聊单选按钮 | 文本为RDO_PRIVATE_CHAT_TEXT；选中后需先选择用户列表 |
| getLblLocalIp()       | 无                                        | JLabel | 获取本机IP显示标签 | 默认文本为“本机IP：查询中...”；查询后更新为“本机IP：XXX”；颜色为COLOR_TIP |
| updateUserList(List<String> userList) | userList：在线用户列表 | void | 更新在线用户列表 | 需在EDT线程中执行；清空原有列表后添加新用户 |
| appendMessage(String sender, String content, boolean isPrivate) | sender：发送者；content：消息内容；isPrivate：是否私聊 | void | 追加消息到显示区 | 群聊消息颜色为COLOR_GROUP_CHAT，私聊为COLOR_PRIVATE_CHAT；格式为“发送者（群聊/私聊）：内容”；需在EDT线程中执行 |
| showTip(String tip, boolean isError) | tip：提示文本；isError：是否错误提示 | void | 显示提示信息 | 错误提示颜色为COLOR_ERROR，普通提示为COLOR_TIP；提示显示在UI显眼位置（如IP标签下方） |
| // UICallback接口实现方法（新增） |                                            |            |                                           |                                           |
| onConnectStatusChanged | boolean isConnected、String tip、boolean isError | void | 连接状态变更通知 | 必须在EDT线程中执行；更新btnConnect文本和状态、btnSend启用/禁用；调用showTip()显示提示 |
| onMessageReceived     | String sender、String content、boolean isPrivate | void | 收到消息通知 | 必须在EDT线程中执行；调用appendMessage()追加消息；滚动条自动定位到最后一行 |
| onUserListUpdated     | List<String> userList | void | 用户列表更新通知 | 必须在EDT线程中执行；调用updateUserList()更新列表 |
| onSendMessageResult   | boolean success、String tip | void | 消息发送结果通知 | 必须在EDT线程中执行；失败时调用showTip()；成功时清空txtMsgInput |

#### 3. UI布局规范
- 整体布局：BorderLayout（北-连接面板、中-聊天面板、南-输入面板）；
- 连接面板（北）：FlowLayout（new FlowLayout(FlowLayout.LEFT, 10, 10)），组件排列顺序为「txtNickname → txtIp → txtPort → btnConnect」；lblLocalIp 换行显示在下方（左对齐），与上方组件垂直间距 10 像素；
- 聊天面板（中）：BorderLayout（西-用户列表面板、东-消息显示区）；
  - 用户列表面板：FlowLayout，包含“在线用户”标签和listOnlineUsers（设置固定宽度200像素）；
  - 消息显示区：txtMsgDisplay（占满剩余空间）；
- 输入面板（南）：FlowLayout，包含rdoGroupChat、rdoPrivateChat、txtMsgInput、btnSend；
- 组件间距：所有组件间距统一为10像素；
- 字体规范：所有组件字体统一为“微软雅黑”12号字。

#### 4. 约束与禁忌
- 禁止使用绝对布局（setBounds）；
- 禁止UI组件命名不符合前缀规范；
- 禁止未提供组件getter方法；
- 禁止UI更新未在EDT线程中执行；
- 禁止样式不统一（颜色、字体、间距不一致）；
- 禁止直接调用工具层（ClientSocketUtil、AESUtil）的方法，所有操作必须通过逻辑层接口触发；
- 禁止未实现UICallback接口或接口方法缺失；
- 禁止逻辑层直接操作UI组件，所有UI更新必须通过UICallback接口触发。

#### 5. 自测标准
| 测试场景                | 操作步骤                                  | 预期结果                                  |
|-------------------------|-------------------------------------------|-------------------------------------------|
| UI初始化测试            | 调用initUI()并显示窗口                    | 组件齐全、布局符合规范、默认状态正确      |
| 接口实现测试            | 检查ChatClientUI是否实现UICallback所有方法 | 无遗漏方法，所有方法在EDT线程中执行 |
| 连接状态切换测试        | 逻辑层调用onConnectStatusChanged(true, "连接成功", false) | btnConnect文本改为“断开”，btnSend启用，显示成功提示 |
| 用户列表更新测试        | 逻辑层调用onUserListUpdated(Arrays.asList("张三", "李四")) | 列表显示正确，无卡顿，线程安全            |
| 消息显示测试            | 逻辑层调用onMessageReceived("张三", "测试消息", false) | 颜色区分正确，格式正确，显示正常          |
| 本机IP显示测试          | 启动UI后等待IP查询结果                    | 正确显示本机IP或断网提示，排版正常        |
| 输入长度限制测试        | 在txtMsgInput中输入超过1000字符            | 无法输入，或提示“消息过长”                |

### （四）开发者4：客户端逻辑衔接（com.chat.client.logic包）
#### 1. 核心职责
- 定义UICallback抽象接口（规范逻辑层与UI层的交互）；
- 衔接UI层与工具层（通过抽象接口调用，不依赖具体实现）；
- 消息格式拼接与解析（按全局规范）；
- 聊天模式切换逻辑（群聊/私聊）；
- 接收消息线程管理（守护线程）；
- 连接状态管理与UI同步（通过UICallback接口）；
- 确保核心逻辑不绑定Swing或Socket，支持后续前端/通信协议无缝切换。

#### 2. 核心接口与类规范
##### （1）UICallback接口
```java
package com.chat.client.logic;

import java.util.List;

/**
 * UI回调接口（逻辑层 → UI层的通知标准）
 * 后续任何前端（Swing/Web/其他GUI）必须实现此接口，才能与逻辑层对接
 */
public interface UICallback {
    /**
     * 连接状态变更通知
     * @param isConnected 连接状态（true=已连接，false=未连接/断开）
     * @param tip 提示文本
     * @param isError 是否错误提示
     */
    void onConnectStatusChanged(boolean isConnected, String tip, boolean isError);

    /**
     * 收到消息通知
     * @param sender 发送者昵称
     * @param content 消息内容（已解密）
     * @param isPrivate 是否为私聊消息
     */
    void onMessageReceived(String sender, String content, boolean isPrivate);

    /**
     * 在线用户列表更新通知
     * @param userList 最新在线用户列表
     */
    void onUserListUpdated(List<String> userList);

    /**
     * 消息发送结果通知
     * @param success 发送是否成功
     * @param tip 失败时的提示文本（成功时可为空）
     */
    void onSendMessageResult(boolean success, String tip);
}
```

##### （2）ChatClientLogic类（单例模式，依赖抽象接口）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| getInstance()         | 无                                        | ChatClientLogic | 获取单例实例 | 单例模式，初始化时通过工厂模式获取NetworkClient实例（默认ClientSocketUtil） |
| registerUICallback    | UICallback callback | void | 注册UI回调接口 | 仅允许绑定一个回调实例，重复调用覆盖；注册后立即通知UI“未连接”状态 |
| unregisterUICallback  | 无                                        | void | 解除UI回调接口绑定 | 断开连接时必须调用，避免内存泄露；解绑后不再发送任何UI通知 |
| connect(String ip, int port, String nickname) | ip：服务器IP；port：端口；nickname：昵称 | void | 发起服务器连接（异步） | 调用NetworkClient.connect()；连接结果通过onConnectStatusChanged通知UI；参数非法直接通知UI错误 |
| disconnect()          | 无                                        | void | 断开服务器连接（同步） | 调用NetworkClient.disconnect()；通过onConnectStatusChanged通知UI；停止接收消息线程 |
| sendMessage(String content, boolean isPrivate, String targetNickname) | content：消息内容；isPrivate：是否私聊；targetNickname：私聊目标昵称（非私聊传null） | void | 发送聊天消息（异步） | 校验内容长度（≤MAX_MESSAGE_LENGTH）；按模式拼接消息格式；加密后调用NetworkClient.sendMessage()；结果通过onSendMessageResult通知 |
| startReceiveThread()  | 无                                        | void | 启动接收消息线程 | 线程为守护线程；循环调用NetworkClient.receiveMessage()；接收后解密、解析、通过UICallback通知UI |
| stopReceiveThread()   | 无                                        | void | 停止接收消息线程 | 中断接收线程，避免阻塞；输出INFO日志 |
| getCurrentConnectStatus | 无 | boolean | 获取当前连接状态 | 调用NetworkClient.isConnected()，实时返回 |
| getCurrentChatMode()  | 无                                        | String | 获取当前聊天模式 | 返回Constants.MESSAGE_TYPE_GROUP或Constants.MESSAGE_TYPE_PRIVATE |
| setTargetNickname(String nickname) | nickname：私聊目标昵称 | void | 设置私聊目标昵称 | 仅在私聊模式下有效 |

##### （3）MessageParser类（静态工具类）
| 方法名                | 参数列表                                  | 返回值类型 | 功能描述                                  | 约束与要求                                  |
|-----------------------|-------------------------------------------|------------|-------------------------------------------|-------------------------------------------|
| parseMessageType(String message) | message：待解析消息（已解密） | String | 解析消息类型 | 按MESSAGE_SEPARATOR拆分，返回第一个字段（消息类型）；格式错误返回null |
| parseGroupMessage(String message) | message：群聊消息（已解密） | String[] | 解析群聊消息 | 返回数组[发送者昵称, 消息内容]；格式错误返回null |
| parsePrivateMessage(String message) | message：私聊消息（已解密） | String[] | 解析私聊消息 | 返回数组[发送者昵称, 接收者昵称, 消息内容]；格式错误返回null |
| parseUserListMessage(String message) | message：用户列表消息（已解密） | List<String> | 解析用户列表消息 | 拆分逗号分隔的用户字符串，返回用户列表；格式错误返回空列表 |
| parseConnectSuccessMessage(String message) | message：连接成功消息（已解密） | String | 解析连接成功消息 | 返回昵称；格式错误返回null |
| parseConnectFailedMessage(String message) | message：连接失败消息（已解密） | String | 解析连接失败消息 | 返回失败原因；格式错误返回“连接失败” |

#### 3. 核心逻辑规范
- 连接逻辑：UI触发 → 逻辑层connect() → 校验参数 → 调用NetworkClient.connect() → 连接成功→启动接收线程→通过UICallback通知UI；失败→通过UICallback通知UI错误；
- 发送逻辑：
  - 群聊模式：UI触发 → 校验内容 → 拼接GROUP格式 → 加密编码 → 调用NetworkClient.sendMessage() → 结果通过UICallback通知；
  - 私聊模式：校验目标用户 → 拼接PRIVATE格式 → 加密编码 → 发送 → 结果通知；
- 接收逻辑：接收线程循环 → 调用NetworkClient.receiveMessage() → 解密解码 → 解析消息类型 → 通过UICallback通知UI更新（消息/用户列表）；
- 异常处理：调用工具层时捕获所有异常 → 通过UICallback通知UI显示友好提示 → 必要时触发disconnect()；
- 接口依赖规范：逻辑层仅依赖NetworkClient和UICallback接口，禁止直接引用ClientSocketUtil、ChatClientUI等具体实现类。

#### 4. 约束与禁忌
- 禁止未校验消息长度直接发送；
- 禁止私聊模式下未选择目标用户直接发送；
- 禁止接收消息未解密直接解析；
- 禁止直接操作UI组件，所有UI更新必须通过UICallback接口；
- 禁止接收消息线程非守护线程导致主线程无法退出；
- 禁止依赖具体实现类（如ClientSocketUtil），仅通过抽象接口调用；
- 禁止在逻辑层中硬编码UI相关逻辑（如提示文本、组件状态）；
- 禁止修改抽象接口的方法定义（需4人共同确认）。

#### 5. 自测标准
| 测试场景                | 操作步骤                                  | 预期结果                                  |
|-------------------------|-------------------------------------------|-------------------------------------------|
| 接口依赖测试            | 检查ChatClientLogic是否直接引用ChatClientUI或ClientSocketUtil | 无直接引用，仅依赖UICallback和NetworkClient接口 |
| 连接成功测试            | 注册UICallback后，调用connect("张三", "127.0.0.1", 8888)，服务器正常 | UI收到onConnectStatusChanged(true, "连接成功", false)，接收线程启动 |
| 连接失败测试            | 调用connect("张三", "192.168.1.100", 8888)，服务器未启动 | UI收到onConnectStatusChanged(false, "网络异常，连接失败", true) |
| 群聊发送测试            | 连接成功后，调用sendMessage("测试群聊", false, null) | 消息按GROUP格式加密发送，服务器广播后UI收到onMessageReceived通知 |
| 私聊发送测试            | 连接成功后，调用sendMessage("测试私聊", true, "李四") | 消息按PRIVATE格式发送，仅目标用户收到，UI收到发送成功通知 |
| 接收消息测试            | 服务器发送GROUP格式消息                   | 逻辑层解密解析后，通过onMessageReceived通知UI显示 |
| 断开连接测试            | 调用disconnect()                          | 工具层断开连接，接收线程停止，UI收到断开通知 |

## 三、衔接保障规范
### 1. 契约冻结原则
本规范发布后，以下内容禁止擅自修改，如需调整需4人共同确认并同步更新规范文档：
- 消息格式规范；
- 抽象接口规范（NetworkClient、UICallback的方法名、参数、返回值）；
- 命名规范；
- 常量定义（取值、类型、用途）；
- 包名与目录结构；
- 层间交互原则（解耦要求）。

### 2. 模块对接要求
- 开发者1（服务器端）：仅需保证按规范监听端口、解析/转发消息，无需关注客户端实现；提供清晰日志，方便客户端排查问题；
- 开发者2（工具层）：仅需保证接口实现符合规范，提供清晰的返回值和日志，无需关注UI层和逻辑层；新增通信协议时，仅需新增NetworkClient实现类，不修改现有代码；
- 开发者3（UI层）：仅需保证组件齐全、命名规范、实现UICallback接口，通过逻辑层接口触发操作，无需关注工具层和服务器端；UI更新严格遵循EDT线程要求；
- 开发者4（逻辑层）：仅依赖抽象接口，禁止侵入其他模块代码；核心逻辑不绑定任何前端技术或通信协议；确保线程安全和异常处理闭环。

### 3. 问题定位原则
衔接时出现问题，按以下顺序排查：
1. 先检查是否符合本规范（如消息格式、接口参数、命名、常量取值）；
2. 再定位问题所属模块（连接失败→开发者2/1，UI不更新→开发者3/4，消息转发失败→开发者1/4）；
3. 最后排查模块内部实现，禁止跨模块修改代码（如需修改，需模块负责人确认）；
4. 所有问题需记录在协作文档中，明确原因、解决方案和规范优化点。


## 四、交付物
每位开发者需交付的内容：
1. 按目录结构组织的源代码（符合所有规范，含注释、异常处理、日志输出）；
2. 《模块实现说明文档》：包含核心逻辑流程图、接口实现细节、异常处理说明、自测报告；
3. 《规范符合性自查表》：对照本规范，勾选是否满足所有约束（命名、注释、线程安全、接口实现等）；
4. 测试用例：包含正常场景、异常场景、边界场景的测试步骤和预期结果；
5. 日志输出示例：提供关键流程（如连接、发送消息、接收消息）的日志输出样例。

## 五、附件

### 公共常量（com.chat.common.Constants）
```java
package com.chat.common;

/**
 * 全局公共常量类（跨模块共用，禁止擅自修改）
 * 维护者：开发者2
 * 核心原则：仅存放"服务器+客户端都需直接引用"的常量，取值统一，修改需4人共同确认
 */
public class Constants {

    // ===================== 消息基础规范常量 =====================
    /**
     * 消息字段分隔符（不可修改，否则会导致跨模块消息解析失败）
     */
    public static final String MESSAGE_SEPARATOR = "#";

    /**
     * 单条消息最大长度（不含消息头字段，仅指消息内容）
     */
    public static final int MAX_MESSAGE_LENGTH = 1000;

    // ===================== 消息类型标识常量 =====================
    /**
     * 群聊消息类型标识
     */
    public static final String MESSAGE_TYPE_GROUP = "GROUP";

    /**
     * 私聊消息类型标识
     */
    public static final String MESSAGE_TYPE_PRIVATE = "PRIVATE";

    /**
     * 在线用户列表更新消息标识
     */
    public static final String MESSAGE_TYPE_USER_LIST = "USER_LIST";

    /**
     * 连接成功响应消息标识
     */
    public static final String MESSAGE_TYPE_CONNECT_SUCCESS = "CONNECT_SUCCESS";

    /**
     * 连接失败响应消息标识
     */
    public static final String MESSAGE_TYPE_CONNECT_FAILED = "CONNECT_FAILED";

    /**
     * 客户端断开通知消息标识
     */
    public static final String MESSAGE_TYPE_DISCONNECT = "DISCONNECT";

    // ===================== 网络连接常量 =====================
    /**
     * 默认端口（服务器监听/客户端连接统一使用）
     */
    public static final int DEFAULT_PORT = 8888;

    /**
     * 默认连接IP（客户端默认连接地址）
     */
    public static final String DEFAULT_IP = "127.0.0.1";

    /**
     * 连接超时时间（单位：毫秒）
     */
    public static final int CONNECT_TIMEOUT = 5000;

    // ===================== 加解密相关常量 =====================
    /**
     * AES加密密钥（固定值，禁止修改，否则会导致加密消息无法解密）
     */
    public static final String AES_KEY = "chatroom_key_123";

    /**
     * Base64编码/解码字符集
     */
    public static final String BASE64_CHARSET = "UTF-8";

    // ===================== 昵称校验约束常量 =====================
    /**
     * 昵称最小长度
     */
    public static final int NICKNAME_MIN_LENGTH = 1;

    /**
     * 昵称最大长度
     */
    public static final int NICKNAME_MAX_LENGTH = 10;

    /**
     * 昵称禁止包含的字符（拼接为字符串，用于校验）
     * 包含：消息分隔符#、用户列表分隔符,
     */
    public static final String NICKNAME_FORBIDDEN_CHARS = "#,";

    // ===================== 连接失败原因常量 =====================
    /**
     * 连接失败原因：昵称已被占用
     */
    public static final String CONNECT_FAILED_REASON_DUPLICATE_NICKNAME = "昵称已被占用";

    /**
     * 连接失败原因：服务器繁忙（超过最大连接数）
     */
    public static final String CONNECT_FAILED_REASON_SERVER_BUSY = "服务器繁忙";

    /**
     * 连接失败原因：昵称格式非法（含禁止字符/长度不符）
     */
    public static final String CONNECT_FAILED_REASON_ILLEGAL_NICKNAME = "昵称格式非法（长度1-10，不含#和逗号）";

    /**
     * 连接失败原因：网络异常
     */
    public static final String CONNECT_FAILED_REASON_NETWORK_ERROR = "网络异常，连接失败";

    // ===================== 禁止实例化 =====================
    /**
     * 私有构造方法，禁止创建常量类实例
     */
    private Constants() {
        throw new UnsupportedOperationException("公共常量类禁止实例化");
    }
}
```