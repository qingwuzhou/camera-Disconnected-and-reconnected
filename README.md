# 如何实现知微相机的掉线重连(C#版)

## [<font size=fault color="violet">1.判断相机是否在线的方法（是否掉线）</font>](#判断相机是否在线的方法如下：)

## [<font size=fault color="violet">2.连接相机的方法</font>](#相机连接的方法)

## [<font size=fault color="violet">3.相机参数初始化的方法</font>](#相机初始化的方法)

## [<font size=fault color="violet">4.相机数据采集的方法</font>](#相机数据采集的方法)

## [<font size=fault color="violet">5.内存空间的释放方法</font>](#相机资源释放的方法)



***逻辑流程图***


![image](https://user-images.githubusercontent.com/63906638/233582706-0b4c77d7-3deb-49b6-9d45-7184d1be34de.png)



###<font size=4 color="red">***公共变量的定义***</font>

```c#
 public const String IP = "192.168.30.98";
        public static int camer_num = 0;
        public static int camera_ret;
        public static int connect;
        public static int[] data = new int[1];
        public static int data_ccp;
        public static SWIGTYPE_p_CAMERA_OBJECT camera_obj1 = null;
        public static PhotoInfoCSharp PointCloud_data = null;
        public static PhotoInfoCSharp gray_data = null;
        public static byte[] point_pixel;
        public static byte[] gray_pixel;
        public static PhotoInfoCSharp RGB_data = null;
        public static byte[] RGB_pixel;
        public static int pointsize;
        public static int graysize;
        public static int RGBsize;
```

###<font size=4 color="red">***判断相机是否在线的方法如下：***</font>

```c#
  public static void is_NOnline()
        {

            while (true)
            {
                Console.WriteLine("**************我在判断相机是否掉线*********************");
                DkamSDK_CSharp.GetCameraCCPStatus(Program.camera_obj1, data);
                data_ccp = data[0];

                Console.WriteLine("--------data_ccp-----------:" + data_ccp);
                if (data_ccp >= 0)
                {
                    Console.WriteLine("********************相机在线,数据采集正常*************");
                    Thread.Sleep(1000);
                    capturedata();
                    release_resource();


                }
                else
                {
                    Console.WriteLine("********************相机不在线或者掉线，启动重连*************");
                    Program.Camera_connect();
                    //Console.ReadLine();
                    Program.init_camera();
                    
                }
            }


        }
```

###<font size=4 color="red">***相机连接的方法***</font>

```c#
public static void Camera_connect()
        {
            camer_num = 0;
            camera_ret = -1;
            /*****************
            打印相机日志
            SetLogLevel(int error, int debug, int warnning, int info)
            打开1 关闭0
            *****************/
            DkamSDK_CSharp.SetLogLevel(1, 0, 0, 1);
            //发现局域网内的相机
            camer_num = DkamSDK_CSharp.DiscoverCamera();
            Console.WriteLine("Camer num is=" + camer_num);
            //创建相机

            if (camer_num < 0)
            {
                Console.WriteLine("No camera");
                Console.ReadKey();
            }

            //对局域网内的相机进行排序0：IP 1:series number	
            int sort = DkamSDK_CSharp.CameraSort(0);
            Console.WriteLine("the camera sort result=" + sort);

            while (camer_num < 1)
            { //一直循环等相机到位
                camer_num = DkamSDK_CSharp.DiscoverCamera();
                Console.WriteLine("局域网没有相机或相机未就位");
            }
            for (int i = 0; i < camer_num; i++)
            {
                //显示局域网内相机IP
                Console.WriteLine("ip is=" + DkamSDK_CSharp.CameraIP(i));
                if (String.Compare(DkamSDK_CSharp.CameraIP(i), IP) == 0)
                {
                    camera_ret = i;
                }
            }
            //连接相机，输入相机的索引号
            camera_obj1 = DkamSDK_CSharp.CreateCamera(camera_ret);
            connect = DkamSDK_CSharp.CameraConnect(camera_obj1);
            Console.WriteLine("Connect Camera result：" + connect);
            //相机和PC机是否在同一个网段内
            Console.WriteLine("WhetherIsSameSegment=" + DkamSDK_CSharp.WhetherIsSameSegment(camera_obj1));

        }
```

###<font size=4 color="red">***相机初始化的方法***</font>

```c#
public static void init_camera()
        {
            //获取相机固件版本号
            Console.WriteLine("CameraVerions=" + DkamSDK_CSharp.CameraVerions(camera_obj1));
            //获取SDK版本号
            Console.WriteLine("SDKVersion=" + DkamSDK_CSharp.SDKVersion(camera_obj1));

            //获取相机CCP状态




            //保存XML到本地              
            // Console.WriteLine(DkamSDK_CSharp.SaveXmlToLocal(camera_obj1, "D:\\"));


            //获取连接相机的宽和高(红外)
            SWIGTYPE_p_int width_gray = DkamSDK_CSharp.new_intArray(0);
            DkamSDK_CSharp.GetCameraWidth(camera_obj1, width_gray, 0);
            int width = DkamSDK_CSharp.intArray_getitem(width_gray, 0);

            SWIGTYPE_p_int height_gray = DkamSDK_CSharp.new_intArray(0);
            DkamSDK_CSharp.GetCameraHeight(camera_obj1, height_gray, 0);
            int height = DkamSDK_CSharp.intArray_getitem(height_gray, 0);

            Console.WriteLine("gray width={0}  gray height={1}", width, height);


            //获取连接相机的宽和高(RGB)
            SWIGTYPE_p_int width_rgb = DkamSDK_CSharp.new_intArray(0);
            DkamSDK_CSharp.GetCameraWidth(camera_obj1, width_rgb, 1);
            int widthRGB = DkamSDK_CSharp.intArray_getitem(width_rgb, 0);
            SWIGTYPE_p_int height_rgb = DkamSDK_CSharp.new_intArray(0);
            DkamSDK_CSharp.GetCameraHeight(camera_obj1, height_rgb, 1);
            int heightRGB = DkamSDK_CSharp.intArray_getitem(height_rgb, 0);

            Console.WriteLine("rgb width={0}  rgb height={1}", widthRGB, heightRGB);

            //分配采集点云的大小
            PointCloud_data = new PhotoInfoCSharp();
            //PointCloud_data.pixel = (width * height * 6).ToString();
            pointsize = width * height * 6;
            point_pixel = new byte[pointsize];
            //分配采集红外的大小
            gray_data = new PhotoInfoCSharp();
            graysize = width * height;
            gray_pixel = new byte[graysize];


            //分配采集RGB的大小
            RGB_data = new PhotoInfoCSharp();
            RGBsize = widthRGB * heightRGB * 3;
            RGB_pixel = new byte[RGBsize];


            //设置红外触发模式
            int TirggMode = DkamSDK_CSharp.SetTriggerMode(camera_obj1, 1);
            Console.WriteLine("Tirgger Mode=" + TirggMode);
            //设置RGB触发模式
            int TirggModeRGB = DkamSDK_CSharp.SetRGBTriggerMode(camera_obj1, 1);
            Console.WriteLine("Tirgger Mode RGB=" + TirggModeRGB);

            //开启数据流通道(0:红外 1:点云 2:RGB)
            int streamgray = DkamSDK_CSharp.StreamOn(camera_obj1, 0);
            Console.WriteLine("Stream On Gray=" + streamgray);

            int streampoint = DkamSDK_CSharp.StreamOn(camera_obj1, 1);
            Console.WriteLine("Stream On PointCloud=" + streampoint);

            int streamRGB = DkamSDK_CSharp.StreamOn(camera_obj1, 2);
            Console.WriteLine("Stream On RGB=" + streamRGB);


            //开始接受数据
            int start = DkamSDK_CSharp.AcquisitionStart(camera_obj1);
            Console.WriteLine("AcquisitionStart=" + start);
        }
```

###<font size=4 color="red">***相机资源释放的方法***</font>

```c#
public static void release_resource()
        {
            //释放内存


            Console.WriteLine("point_pixel.Length = {0}", point_pixel.Length);
            //释放内存
            Array.Clear(point_pixel, 0, point_pixel.Length);
            Array.Resize(ref (point_pixel), 0);
            Console.WriteLine("point_pixel.Length = {0}", point_pixel.Length);
            Array.Clear(gray_pixel, 0, gray_pixel.Length);
            Array.Resize(ref (gray_pixel), 0);
            Array.Clear(RGB_pixel, 0, RGB_pixel.Length);
            Array.Resize(ref (RGB_pixel), 0);


            //Array.Clear(point_pixel, 0, point_pixel.Length);
            //Array.Clear(gray_pixel, 0, gray_pixel.Length);

            //Array.Clear(RGB_pixel, 0, RGB_pixel.Length);

            // DkamSDK_CSharp.delete_floatArray(gray_cloud);
            //DkamSDK_CSharp.delete_floatArray(rgb_cloud);
            //断开红外、点云、RGB数据流通道
            DkamSDK_CSharp.AcquisitionStop(camera_obj1);
            int streamoff = DkamSDK_CSharp.StreamOff(camera_obj1, 0);
            Console.WriteLine("StreamOff Gray=" + streamoff);
            int streamoffpoint = DkamSDK_CSharp.StreamOff(camera_obj1, 1);
            Console.WriteLine("StreamOff Point=" + streamoffpoint);
            int streamoffRGB = DkamSDK_CSharp.StreamOff(camera_obj1, 2);
            Console.WriteLine("StreamOff RGB=" + streamoffRGB);
            //断开相机连接
            int dis = DkamSDK_CSharp.CameraDisconnect(camera_obj1);
            Console.WriteLine("CameraDisconnect=" + dis);
            //销毁相机参数
            DkamSDK_CSharp.DestroyCamera(camera_obj1);
            PointCloud_data = null;
            gray_data = null;
            RGB_data = null;
            //camera_obj1 = null;
            GC.Collect();
        }
```



###<font size=4 color="red">***相机数据采集的方法***</font>

```c#
 public static void capturedata()
        {
            int ss = 1;
            int capturepoint = -1;
            int capturegray = -1;
            int capturergb = -1;
            string name1, name2, name3;
            int savepoint = -1;
            int savegray = -1;
            int saveRGB = -1;

            if (connect == 0)
            {

                while (ss < 30 && data_ccp >= 0)
                {


                    Console.WriteLine("--------capture total :" + ss++ + "-------------");

                    //设置曝光模式
                    int SetAutoExposureRGB = DkamSDK_CSharp.SetAutoExposure(camera_obj1, 1, 1);
                    Console.WriteLine("SetAutoExposureRGB=" + SetAutoExposureRGB);
                    int SetAutoExposure = DkamSDK_CSharp.SetAutoExposure(camera_obj1, 1, 0);
                    Console.WriteLine("SetAutoExposure=" + SetAutoExposure);

                    /*******************
                    设置红外曝光时间
                    SetExposureTime(int camera_index, int utime, int camera_cnt)
                    camera_index:相机下标  utime:曝光时间 camera_cnt:CMOS编号
                    ********************/

                    int setexposureTime = DkamSDK_CSharp.SetExposureTime(camera_obj1, 1000, 0);
                    Console.WriteLine("Set ExposureTime=" + setexposureTime);
                    int getexposureTime = DkamSDK_CSharp.GetExposureTime(camera_obj1, 0);
                    Console.WriteLine("Get ExposureTime=" + getexposureTime);
                    //设置RGB曝光时间
                    int setexposureTimeRGB = DkamSDK_CSharp.SetExposureTime(camera_obj1, 1000, 1);
                    Console.WriteLine("Set ExposureTime RGB=" + setexposureTimeRGB);
                    int getexposureTimeRGB = DkamSDK_CSharp.GetExposureTime(camera_obj1, 1);
                    Console.WriteLine("Get ExposureTime RGB=" + getexposureTimeRGB);
                    //刷新缓冲区数据
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 0);
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 2);
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 1);

                    DateTime d1 = DateTime.Now;
                    Console.WriteLine("触发前时间time--------------------------=" + d1.ToString());

                    //Console.WriteLine("重启相机");
                    //DkamSDK_CSharp.SetCommandNodeValue(camera_obj1, "DeviceReboot");

                    //设置相机红外触发模式下的触发帧数
                    int trigger_count = DkamSDK_CSharp.SetTriggerCount(camera_obj1);
                    Console.WriteLine("set gray trigger count =" + trigger_count);
                    // 设置相机RGB触发模式下的触发帧数
                    int rgb_trigger_count = DkamSDK_CSharp.SetRGBTriggerCount(camera_obj1);
                    Console.WriteLine("set RGB trigger count =" + rgb_trigger_count);

                    DateTime c_d1 = DateTime.Now;
                    Console.WriteLine("触发后时间time--------------------------=" + c_d1.ToString());


                    if (rgb_trigger_count == 0)
                    {
                        //采集数据
                        capturepoint = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 1, PointCloud_data, point_pixel, pointsize, 9000000);
                        Console.WriteLine("Capture PointCloud PCD=" + capturepoint);
                        capturegray = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 0, gray_data, gray_pixel, graysize, 9000000);
                        Console.WriteLine("Capture Gray=" + capturegray);
                        capturergb = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 2, RGB_data, RGB_pixel, RGBsize, 9000000);
                        Console.WriteLine("Capture RGB=" + capturergb);
                        DateTime ca_d1 = DateTime.Now;
                        Console.WriteLine("采集结束时间time--------------------------=" + ca_d1.ToString());

                        DkamSDK_CSharp.FilterPointCloudCSharp(camera_obj1, PointCloud_data, point_pixel, pointsize, 0.6);

                        name1 = "point_" + PointCloud_data.block_id + ".pcd";
                        name2 = "gray_" + gray_data.block_id + "_gray.bmp";
                        name3 = "rgb_" + RGB_data.block_id + "_rgb.bmp";
                        //保存点云
                        savepoint = DkamSDK_CSharp.SavePointCloudToPcdCSharp(camera_obj1, PointCloud_data, point_pixel, pointsize, name1);
                        //保存红外
                        savegray = DkamSDK_CSharp.SaveToBMPCSharp(camera_obj1, gray_data, gray_pixel, graysize, name2);
                        Console.WriteLine("Save Gray=" + savegray);
                        //保存RGB
                        saveRGB = DkamSDK_CSharp.SaveToBMPCSharp(camera_obj1, RGB_data, RGB_pixel, RGBsize, name3);
                        Console.WriteLine("Save Gray=" + saveRGB);
                    }


                    //----------------------------------------------------------------------------------------------

                    Console.WriteLine("曝光100000----------------------------------------------------------------------");
                    //设置曝光模式
                    SetAutoExposureRGB = DkamSDK_CSharp.SetAutoExposure(camera_obj1, 1, 1);
                    Console.WriteLine("SetAutoExposureRGB=" + SetAutoExposureRGB);
                    SetAutoExposure = DkamSDK_CSharp.SetAutoExposure(camera_obj1, 1, 0);
                    Console.WriteLine("SetAutoExposure=" + SetAutoExposure);


                    int time = 100000;
                    setexposureTime = DkamSDK_CSharp.SetExposureTime(camera_obj1, time, 0);
                    Console.WriteLine("Set ExposureTime=" + setexposureTime);
                    getexposureTime = DkamSDK_CSharp.GetExposureTime(camera_obj1, 0);
                    Console.WriteLine("Get ExposureTime=" + getexposureTime);
                    //设置RGB曝光时间
                    setexposureTimeRGB = DkamSDK_CSharp.SetExposureTime(camera_obj1, time, 1);
                    Console.WriteLine("Set ExposureTime RGB=" + setexposureTimeRGB);
                    getexposureTimeRGB = DkamSDK_CSharp.GetExposureTime(camera_obj1, 1);
                    Console.WriteLine("Get ExposureTime RGB=" + getexposureTimeRGB);
                    //刷新缓冲区数据
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 0);
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 2);
                    DkamSDK_CSharp.FlushBuffer(camera_obj1, 1);


                    DateTime d2 = DateTime.Now;
                    Console.WriteLine("第二次触发时间time---------------------------------------------=" + d2.ToString());

                    //设置相机红外触发模式下的触发帧数
                    trigger_count = DkamSDK_CSharp.SetTriggerCount(camera_obj1);
                    Console.WriteLine("set gray trigger count =" + trigger_count);
                    // 设置相机RGB触发模式下的触发帧数
                    rgb_trigger_count = DkamSDK_CSharp.SetRGBTriggerCount(camera_obj1);
                    Console.WriteLine("set RGB trigger count =" + rgb_trigger_count);

                    if (trigger_count == 0)
                    {
                        //采集数据

                        capturepoint = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 1, PointCloud_data, point_pixel, pointsize, 150000000);
                        Console.WriteLine("Capture PointCloud PCD=" + capturepoint);
                        capturegray = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 0, gray_data, gray_pixel, graysize, 150000000);
                        Console.WriteLine("Capture Gray=" + capturegray);
                        capturergb = DkamSDK_CSharp.TimeoutCaptureCSharp(camera_obj1, 2, RGB_data, RGB_pixel, RGBsize, 150000000);
                        Console.WriteLine("Capture RGB=" + capturergb);

                        //保存point
                        name1 = "point_" + PointCloud_data.block_id + ".pcd";
                        DkamSDK_CSharp.FilterPointCloudCSharp(camera_obj1, PointCloud_data, point_pixel, pointsize, 0.6);
                        //保存点云
                        savepoint = DkamSDK_CSharp.SavePointCloudToPcdCSharp(camera_obj1, PointCloud_data, point_pixel, pointsize, name1);
                        Console.WriteLine("Save PointCloud PCD=" + savepoint);
                        //保存红外
                        Console.WriteLine("Capture Gray=" + capturegray);
                        name2 = "gray_" + gray_data.block_id + "_gray.bmp";
                        savegray = DkamSDK_CSharp.SaveToBMPCSharp(camera_obj1, gray_data, gray_pixel, graysize, name2);
                        Console.WriteLine("Save Gray=" + savegray);

                        //保存RGB
                        Console.WriteLine("Capture RGB=" + capturergb);
                        name3 = "rgb_" + RGB_data.block_id + "_rgb.bmp";
                        saveRGB = DkamSDK_CSharp.SaveToBMPCSharp(camera_obj1, RGB_data, RGB_pixel, RGBsize, name3);
                        Console.WriteLine("Save Gray=" + saveRGB);

                        // Console.ReadKey();
                        //  Console.ReadKey();
                        Console.WriteLine("******************capture***********************" + ss + "次");
                        Console.WriteLine("重启相机");//模拟相机断线意外掉线
                        DkamSDK_CSharp.SetCommandNodeValue(camera_obj1, "DeviceReboot");
                    }

                    Thread.Sleep(1000);

                    DkamSDK_CSharp.GetCameraCCPStatus(Program.camera_obj1, data);
                    data_ccp = data[0];
                    Console.WriteLine("--------data_ccp-----------:" + data_ccp);
                }



            }

        }
```

### <font size=4 color="red">***main方法***</font>

```c#
static void Main(string[] args)
        {
            Camera_connect();
            init_camera();

            Thread task = new Thread(is_NOnline);
            task.Start();
            // release_resource();
      }
```

