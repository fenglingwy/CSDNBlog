>本篇文章包含以下内容

>* 蓝牙BLE4.0基础知识的介绍
	* 一、蓝牙常见名称和缩写
	* 二、蓝牙基础知识
	* 三、BLE中心模式流程
	* 四、BLE测试
>* 实现真机蓝牙BLE4.0连接蓝牙模块的通信功能
	* 一、先从结果分析 
	* 二、准备工作 
	* 三、建立中心角色
	* 四、扫描外设
	* 五、连接外设
	* 六、获取服务
	* 七、获取服务中的特征
	* 八、获取特征中的值和描述
	* 九、读取特征中的值和描述
	* 十、读取蓝牙设备的Mac地址
	* 十一、状态改变和发现描述 
	* 十二、发送数据
	* 十三、查看结果
>* 结语

#**蓝牙BLE4.0基础知识的介绍**


----------
>本篇文章视频传送地址：http://edu.csdn.net/course/detail/4534
>本篇GithubDemo：https://github.com/AndroidHensen/iOSBLEDemo

最近刚学的苹果，为了实现蓝牙门锁的项目，找了一天学习了下蓝牙的原理，亲手测试了一次蓝牙的通信功能，结果成功了，那么就把我学习的东西分享一下，当然，安卓BLE4.0也是我负责的，我也会在博客中贡献给大家，最后也有源码下载，欢迎关注我的博客

##**一、蓝牙常见名称和缩写**

- BLE：(Bluetooth low energy)蓝牙4.0设备因为低耗电
- Central：中心设备，发起蓝牙连接的设备(一般是指手机)
- Peripheral：外设，被蓝牙连接的设备(一般是运动手环)
- Service and Characteristic：服务和特征，每个设备会提供服务和特征，类似于服务端的API，但是结构不同，每个设备会有很多服务，每个服务中包含很多特征，这些特征的权限一般分为读(read)，写(write)，通知(notify)几种，就是我们连接设备后具体需要操作的内容
- Description：描述，每个Characteristic可以对应一个或者多个Description用于描述Characteristic的信息或属性

##**二、蓝牙基础知识**

- CoreBluetooth框架的核心其实是俩东西

1. Peripheral
2. Central

![](http://img.blog.csdn.net/20170310225546817?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 这两组api分别对应不同的业务常见

1. 左侧叫中心模式，就是以你的app作为中心，连接其他的外设的场景
2. 右侧称为外设模式，使用外设连接其他中心设备操作的场景

- 服务和特征(service and characteristic)
1. 每个设备都会有1个or多个服务
2. 每个服务里都会有1个or多个特征
3. 特征就是具体键值对，提供数据的地方
4. 每个特征属性分为：读，写，通知等等

- 外设，服务，特征的关系
 
 ![](http://img.blog.csdn.net/20170310225831302?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**三、BLE中心模式流程**
- 1.建立中心角色
- 2.扫描外设(Discover Peripheral)
- 3.连接外设(Connect Peripheral)
- 4.扫描外设中的服务和特征(Discover Services And Characteristics)
    * 4.1 获取外设的services
    * 4.2 获取外设的Characteristics，获取characteristics的值，获取Characteristics的Descriptor和Descriptor的值
- 5.利用特征与外设做数据交互(Explore And Interact)
- 6.订阅Characteristic的通知
- 7.断开连接(Disconnect)

## **四、BLE测试**
- 一台苹果设备，进行真机测试
- 一个蓝牙模块或者外设

#**实现真机蓝牙BLE4.0连接蓝牙模块的通信功能**


----------


这里使用的是蓝牙模块Risym cc2541和苹果手机实现两者的通信功能，根据BLE中心模式流程走就可以了

##**一、先从结果分析**

下面是手机设备NSLog打印输出的结果，从连接到发送数据和接收数据的过程（可右键看大图）

![这里写图片描述](http://img.blog.csdn.net/20170310234642700?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以发现连接成功后，设备有两个服务，第一个服务对应有九个特征值，第二个服务对应有一个特征值，验证了上面的原理是正确的

![这里写图片描述](http://img.blog.csdn.net/20170310234701794?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

根据特征值可以读取出特征的数据，改变通知状态后就能接收数据，下面就是获取描述，接着从蓝牙模块发送数据（11）到手机

##**二、准备工作**

1. 导入依赖库
2. 声明委托
3. 定义变量

```
#import <CoreBluetooth/CoreBluetooth.h>

#define mBLEName @"BT05"

@interface XMGCenterBLEVC () <CBCentralManagerDelegate, CBPeripheralDelegate>
//手机设备
@property (nonatomic, strong) CBCentralManager *mCentral;
//外设设备
@property (nonatomic, strong) CBPeripheral *mPeripheral;
//特征值
@property (nonatomic, strong) CBCharacteristic *mCharacteristic;
//服务
@property (nonatomic, strong) CBService *mService;
//描述
@property (nonatomic, strong) CBDescriptor *mDescriptor;

@end
```

##**三、建立中心角色**

1、程序开始时初始化设备
```
- (CBCentralManager *)mCentral
{
    if (!_mCentral) {
        _mCentral = [[CBCentralManager alloc] initWithDelegate:self
                                                         queue:dispatch_get_main_queue()
                                                       options:nil];
    }
    return _mCentral;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    //1、中心管理者初始化
    [self mCentral];
}
```
2、当程序退出时，记得断开连接

```
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    //停止扫描
    if([self.mCentral isScanning]){
        [self.mCentral stopScan];
    }
    //断开连接
    if(self.mCentral != nil && self.mPeripheral.state == CBPeripheralStateConnected){
        [self.mCentral cancelPeripheralConnection:self.mPeripheral];
    }
}
```

##**四、扫描外设**

```
//2、只要中心管理者初始化,就会触发此代理方法
- (void)centralManagerDidUpdateState:(CBCentralManager *)central
{
    switch (central.state) {
        case CBCentralManagerStateUnknown:
            NSLog(@"CBCentralManagerStateUnknown");
            break;
        case CBCentralManagerStateResetting:
            NSLog(@"CBCentralManagerStateResetting");
            break;
        case CBCentralManagerStateUnsupported:
            NSLog(@"CBCentralManagerStateUnsupported");
            break;
        case CBCentralManagerStateUnauthorized:
            NSLog(@"CBCentralManagerStateUnauthorized");
            break;
        case CBCentralManagerStatePoweredOff:
            NSLog(@"CBCentralManagerStatePoweredOff");
            break;
        case CBCentralManagerStatePoweredOn:
        {
            NSLog(@"CBCentralManagerStatePoweredOn");
            //3、搜索外设
            [self.mCentral scanForPeripheralsWithServices:nil // 通过某些服务筛选外设
                                              options:nil]; // dict,条件
        }
            break;
        default:
            break;
    }
}
```

##**五、连接外设**

这里注意需要交付代理
```
//4、发现外设后调用的方法
- (void)centralManager:(CBCentralManager *)central // 中心管理者
 didDiscoverPeripheral:(CBPeripheral *)peripheral // 外设
     advertisementData:(NSDictionary *)advertisementData // 外设携带的数据
                  RSSI:(NSNumber *)RSSI // 外设发出的蓝牙信号强度
{
    NSLog(@"搜索到name==%@,identifier==%@",peripheral.name,peripheral.identifier);
    //(ABS(RSSI.integerValue) > 35)，RSSI表示信号强度
    //5、发现完之后就是进行连接
    if([peripheral.name isEqualToString:mBLEName]){
        self.mPeripheral = peripheral;
        self.mPeripheral.delegate = self;
        [self.mCentral connectPeripheral:peripheral options:nil];
    }
}
```

##**六、获取服务**

连接成功和失败都会执行对应的代理方法
```
//6、中心管理者连接外设成功
- (void)centralManager:(CBCentralManager *)central // 中心管理者
  didConnectPeripheral:(CBPeripheral *)peripheral // 外设
{
    NSLog(@"%@==设备连接成功", peripheral.name);
    //7、外设发现服务,传nil代表不过滤
    [self.mPeripheral discoverServices:nil];
}

// 外设连接失败
- (void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
    NSLog(@"设备连接失败==%@", peripheral.name);
}

// 丢失连接
- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
    NSLog(@"设备丢失连接==%@", peripheral.name);
}
```

##**七、获取服务中的特征**

```
//8、发现外设的服务后调用的方法
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error
{
    // 是否获取失败
    if (error) {
        NSLog(@"备获取服务失败==%@", peripheral.name);
        return;
    }
    for (CBService *service in peripheral.services) {
        self.mService = service;
        NSLog(@"设备的服务==%@,UUID==%@,count==%lu",service,service.UUID,peripheral.services.count);
        //9、外设发现特征
        [peripheral discoverCharacteristics:nil forService:service];
    }
}
```

##**八、获取特征中的值和描述**

这里必须注意的是：需要将外设的通知打开，否则无法接收数据

```
//10、从服务中发现外设特征的时候调用的代理方法
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error
{
    for (CBCharacteristic *cha in service.characteristics) {
	    //该特征值具有读写权限
        if(cha.properties & CBCharacteristicPropertyWrite && cha.properties & CBCharacteristicPropertyRead){
            self.mCharacteristic = cha;
        }
        NSLog(@"设备的服务==%@,服务对应的特征值==%@,UUID==%@,count==%lu",service,cha,cha.UUID,service.characteristics.count);
        //11、获取特征对应的描述 会回调didUpdateValueForDescriptor
        [peripheral discoverDescriptorsForCharacteristic:cha];
        //12、获取特征的值 会回调didUpdateValueForCharacteristic
        [peripheral readValueForCharacteristic:cha];
    }
    if(nil != self.mCharacteristic){
	    //打开外设的通知，否则无法接受数据
	    [peripheral setNotifyValue:YES forCharacteristic:self.mCharacteristic];
	}
}
```

##**九、读取特征中的值和描述**

这里的回调的方法是特征值发生改变时，也就是说蓝牙发送数据的时候，特征值改变，这里会被调用，相当于接收数据
```
//13、更新描述值的时候会调用
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForDescriptor:(CBDescriptor *)descriptor error:(NSError *)error
{
    NSLog(@"描述==%@",descriptor.description);
}

//14、更新特征值的时候调用，可以理解为获取蓝牙发回的数据
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
{
    NSString *value = [[NSString alloc] initWithData:characteristic.value encoding:NSASCIIStringEncoding];
    NSLog(@"设备的特征值==%@,获取的数据==%@",characteristic,value);
    //这里可以在这里获取描述，由于项目没有用到描述，所以注释了
//    for (CBDescriptor *descriptor in characteristic.descriptors) {
//        self.mDescriptor = descriptor;
//        [peripheral readValueForDescriptor:descriptor];
//    }
}
```
##**十、读取蓝牙设备的Mac地址**
由于安卓是使用Mac地址连接的，而iOS是用UUID进行连接的，所以在iOS连接时，可以先获取Mac地址，存储起来，方便安卓的连接，这样就完成了苹果与安卓的互通

这里只需要修改上面的didUpdateValueForCharacteristic方法中，对获取数据的处理即可

```
#pragma 特征值的获取
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
	//正常的数据返回值ASCII码格式
    NSString *value = [[NSString alloc] initWithData:characteristic.value encoding:NSASCIIStringEncoding];
    //专门处理Mac地址的值
    NSString *macValue = [NSString stringWithFormat:@"%@",characteristic.value];
    
    NSLog(@"获取的数据==%@",value);
    //Mac地址数据
    if([characteristic.UUID.UUIDString isEqualToString:@"2A23"]){
        _macString = [[NSMutableString alloc] init];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(16, 2)] uppercaseString]];
        [_macString appendString:@":"];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(14, 2)] uppercaseString]];
        [_macString appendString:@":"];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(12, 2)] uppercaseString]];
        [_macString appendString:@":"];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(5, 2)] uppercaseString]];
        [_macString appendString:@":"];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(3, 2)] uppercaseString]];
        [_macString appendString:@":"];
        [_macString appendString:[[macValue substringWithRange:NSMakeRange(1, 2)] uppercaseString]];
    }
}

```

##**十一、状态改变和发现描述**
打开外设通知的时候会调用通知状态改变的回调
```
//15、通知状态改变时调用
-(void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
    if(error){
        NSLog(@"改变通知状态");
    }
}

//16、发现外设的特征的描述数组
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverDescriptorsForCharacteristic:(nonnull CBCharacteristic *)characteristic error:(nullable NSError *)error
{
    // 在此处读取描述即可
    for (CBDescriptor *descriptor in characteristic.descriptors) {
        self.mDescriptor = descriptor;
        NSLog(@"发现外设的特征descriptor==%@",descriptor);
    }
}
```

##**十二、发送数据**

1. 这里是使用一个按钮发送数据，数据0x44和0x33对应ACSLL码的D3
2. 这里要注意参数type
	* CBCharacteristicWriteWithoutResponse：表示发送数据到蓝牙设备不需要回调代理方法didWriteValueForCharacteristic
	* CBCharacteristicWriteWithResponse：表示发送数据到蓝牙设备需要回调代理方法didWriteValueForCharacteristic
```
//17、发送数据
- (IBAction)send:(UIButton *)sender {
    Byte dataArr[2];
    //数据D3
    dataArr[0]=0x0044; dataArr[1]=0x0033;
    NSData * myData = [NSData dataWithBytes:dataArr length:2];
    // 核心代码在这里
    if(nil != self.mCharacteristic){
	    [self.mPeripheral writeValue:myData // 写入的数据
	         forCharacteristic:self.mCharacteristic // 写给哪个特征
	                      type:CBCharacteristicWriteWithoutResponse];// 通过此响应记录是否成功写入
	}
}
```
或者也可以将发送数据封装成方法，方便调用

```
#pragma 发送数据
-(void)sendDataToBLE:(NSString *)data{
    NSData* myData = [data dataUsingEncoding:NSUTF8StringEncoding];
    if(nil != self.mCharacteristic){
        [self.mPeripheral writeValue:myData // 写入的数据
                   forCharacteristic:self.mCharacteristic // 写给哪个特征
                                type:CBCharacteristicWriteWithResponse];
    }
}
```
当数据发送成功之后，会执行代理的回调方法

```
-(void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
    if(error){
        NSLog(@"发送数据失败");
    }
    NSLog(@"发送数据成功");
}
```


##**十三、查看结果**

从电脑串口助手可以看到，发送了两次的D3数据，手机也收到了两次11的数据

![](http://img.blog.csdn.net/20170311001214614?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#结语
其实苹果的蓝牙比安卓简单一些，苹果蓝牙都在代理中封装好了方法，只要交付代理之后，实现代理中的方法即可做出相应的动作，整个过程还是很好理解的，我都标识了执行的顺序和思路，大家可以多看看源码，当然，安卓的BLE教程我也是会贡献给大家的，欢迎关注我的博客

[源码下载](http://download.csdn.net/detail/qq_30379689/9777464)