# F
全国大学生嵌入式比赛——软通教育无人水质勘测船
一、硬件代码架构
主要分为多个功能模块，每个模块负责不同的功能：
电机驱动模块：drv_boat_motor.h 和 drv_boat_motor.c
水泵控制模块：water_pump.h 和 water_pump.c
传感器数据采集模块：包括 pH 值传感器（drv_ph.h 和 drv_ph.c）、浑浊度传感器（drv_ts.h 和 drv_ts.c）、温湿度传感器（sht20.h 和 sht20.c）和 GPS 模块（uart_gps.h 和 uart_gps.c）
通信模块：uart_4g.h 和 uart_4g.c
避障模块：ultra_servo.h 和 ultra_servo.c
主程序 iot_boat_example.c 负责创建线程并协调各个模块的工作。
主要的驱动模块
（1）电机驱动模块
  功能概述：该模块用于控制小船的左右电机，实现小船的前进、左转、右转和停止操作。
  代码文件：drv_boat_motor.h，drv_boat_motor.c。
  关键函数：
    boat_motor_init()：初始化双电机驱动，包括 PWM 控制器和方向控制 GPIO。
    boat_motor_set_state()：设置单个电机的状态和转速。
    boat_move_forward()：小船前进。
    boat_turn_left()：小船左转。
    boat_turn_right()：小船右转。
    boat_stop()：小船停止。
（2）水泵控制模块
  功能概述：该模块用于控制水泵的抽水和放水操作。
  代码文件： water_pump.h， water_pump.c：
  关键函数：
  water_pump_init()：初始化抽水和放水的 GPIO 引脚。
  pump_water_dir()：设置为抽水方向。
  drain_water_dir()：设置为放水方向。
  turnoff_pump_water()：关闭水泵。
（3）传感器数据采集模块
  pH 值传感器
  功能概述：采集水质的酸碱度（pH 值）。
  代码文件：drv_ph.h ， drv_ph.c
  关键函数：
    ph_adc_dev_init()：初始化 ADC。
    ph_adc_get_voltage()：获取 ADC 电压值。
    ph_get_value()：返回 PH 计算值。
  浑浊度传感器
  功能概述：采集水质的浑浊度（TS）。
  代码文件：drv_ts.h ， drv_ts.c
  关键函数：
    ts_adc_dev_init()：初始化 ADC。
    ts_adc_get_voltage()：获取 ADC 电压值。
    ts_get_value()：获取浑浊度值。
  温湿度传感器（SHT20）
  功能概述：采集环境的温度和湿度。
  代码文件：sht20.h ， sht20.c
  关键函数：
    sht20_init()：初始化 SHT20。
    sht20_read_data()：读取温度和湿度数据。
（4） GPS 模块
  功能概述：获取小船的地理位置信息。
  代码文件：uart_gps.h ，uart_gps.c
  关键函数：
    gps_init()：初始化 GPS 串口。
    gps_example()：解析经纬度信息。
    gps_example_latDecimal()：传出纬度信息。
    gps_example_lonDecimal()：传出经度信息。
（5）通信模块（4G/NB - IoT）
  功能概述：该模块用于实现与外界的通信，接收远程控制命令并发送传感器数据。
  代码文件：uart_4g.h，uart_4g.c。
  关键函数：
    nb_iot_init()：初始化串口。
    get_paras_value()：解析 JSON 命令。
    nb_iot_process_command()：处理远程控制命令。
（6）避障模块
  功能概述：该模块用于实现小船的自动避障功能。
  代码文件：ultra_servo.h：，ultra_servo.c。
  关键函数
    Ultra_Init()：初始化超声波传感器。
    Servo_Init()：初始化舵机。
    Servo_SetAngle()：设置舵机角度。
    Ultra_Distance()：超声波测距。
    UltraServo_StartTask()：自动避障任务启动。
二、软件代码架构
 api文件中有apiFun.ets，这个代码的设计是为了生成一个与华为云建立连接的TOKEN，通过GET获取到华为云平台的值，通过POST向华为云的发送信息下发命令操作。主要是函数为后面操控小船和数据显示的页面服务，调用utils/MsgReportedData.ets中的定义的接口进行数据解析和数据上传。

 pages文件夹中有三个文件页面，Index.ets文件是app进入页面按下登入后跳到LoadingPage.ets文件跳转页面3秒系统自动跳转到MainPage.ets页面中显示主页的UI界面，一共有三个界面第一个是主页界面显示数据和操控小船，第二个是map地图显示界面，显示高德地图信息经纬度信息，地图上蓝色的点为我当前所在位置，地图可以进行黄色标记可以获取到经纬度和发送自动巡航命令
 
Veiws文件夹中三个.ets文件，其中HomeComponent.ets文件是数据显示和，操控小船的页面UI界面，主要调用model/ValueItem.ets定义的结构变量信息依次显示获取到的信息值，然后通过四个按键控制小船行进，两个按键控制水泵。MapComponet.ets文件是显示地图ArkUI组件通过webviewController向地图页面注入JavaScript代码显示地图信息，有三个按键，开始为自动巡航功能，停止为结束自动巡航，更新为输入的经纬度信息在地图中更新坐标。MineComponent.ets文件是我的页面显示APP和账户的一些信息。

model文件夹中的文件有三个.ets文件，第一个文件MineltemList.ets，该文件是在Views/MineComponent.ets所调用的列表接口，第二个文件TabItem.ets是定义在pages/MainPage.ets所调用的页面切换接口，第三个文件ValueItem是定义给model/ValueItem.ets文件中依次从云平台的获取到的信息进行显示图片和数据在主页页面显示。

utils文件夹中MsgReportedData.ets文件是定义的接口解析获取云平台的信息，Values.ets定义了变量的类型提供到Views/HomeComponent.ets中。

map.html文件主要是写了一个接入高德地图的信息，然后在这里进行北斗卫星的2000 中国大地坐标系（CGCS2000）和高德地图使用的是 GCJ-02 坐标系，经纬度信息的互相转换，以及定位的信息显示，一些按键功能的实现，小船行驶的轨迹显示。
三、K210代码架构
各模块关键输入输出变量详解
硬件初始化模块：输入变量：无；输出变量：lcd：初始化后的 LCD 对象，用于显示图像。uart：初始化后的串口对象，用于数据通信。sensor：配置好的摄像头对象，用于图像采集。clock：时钟对象，用于计算 FPS
模型配置模块：输入变量：labels：类别标签列表，定义可识别的目标类别；anchor：锚点参数元组，用于 YOLOv2 模型；model_path：模型文件路径（代码中为/sd/det.kmodel）；输出变量：kpu：初始化后的 KPU 对象，用于运行神经网络模型
主循环处理模块：输入变量：sensor：摄像头对象，用于捕获图像；kpu：初始化的 KPU 模型，用于推理；lcd：LCD 对象，用于显示结果；uart：串口对象，用于发送数据；输出变量：dect：检测结果列表，包含目标信息；img：标注后的图像，用于显示；串口发送的 JSON 数据。

检测结果处理模块：输入变量：dect：检测结果列表，每个元素包含目标信息；labels：类别标签列表，用于映射类别索引；img：原始图像对象，用于绘制标注；uart：串口对象，用于发送数据；输出变量：img：标注后的图像（绘制了检测框和标签）；串口发送的 JSON 数据（根据识别类别）。
串口通信模块：输入变量：class_label：识别到的类别标签（如 'can'、'bottle' 等）；输出变量：JSON 格式的字符串，通过串口发送（如{"services":[{"service_id":"s1","rubbish":{"v":"a can"}}]}）
分层流程说明
顶层系统：将整个系统划分为硬件初始化、模型配置和主循环处理三个主要模块，明确模块间的关系。
硬件初始化：按顺序初始化 LCD、GPIO、LED、串口和摄像头，配置摄像头的像素格式、分辨率等参数，为图像采集做准备。
模型配置：定义识别类别和锚点参数，加载训练好的模型并初始化 YOLOv2 模型，为目标检测做准备。
主循环处理：系统的核心循环，包括垃圾回收、图像采集、模型推理、结果处理和显示，形成一个闭环。
检测结果处理：对模型推理得到的检测结果进行遍历处理，绘制检测框和标签，并根据识别类别执行相应操作。
串口通信：根据识别到的类别，构建 JSON 格式的数据并通过串口发送，实现与其他设备的通信。
