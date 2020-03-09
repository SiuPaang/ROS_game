点云定义：
结合激光测量和摄影测量原理得到点云，点云是点的集合
而点云中的点包括三维坐标（XYZ）、激光反射强度（Intensity）和颜色信息（RGB）等

1、安装pcl  git clone https://github.com/PointCloudLibrary/pcl.git
2、安装压缩包成功之后（csdn上也有将下面的release也有命名为build的）
>> cd pcl
>> mkdir release
>> cd release
>> cmake -DCMAKE_BUILD_TYPE=None -DCMAKE_INSTALL_PREFIX=/
	usr -DBUILD_GPU=ON -DBUILD_apps=ON -DBUILD_examples=ON -DCMAKE_INSTALL_PREFIX=/usr ..
>> make -j8(-j8也能用-j6\-j4\-j2代替)
>> sudo make install

3、源码安装完之后在cmakelists.txt里添加如下语句 find_package(PCL 1.9 REQUIRED) 
然后在对每一个cpp文件编译时的target_link_libraries里加入${PCL_LIBRARIES}
例如：target_link_libraries(pcl_segmentate ${catkin_LIBRARIES} ${OpenCV_LIBRARIES} ${PCL_LIBRARIES})


点云的数据结构是PointCloud,它是一个类，包含下面的数据域：
width(int)以及height(int)
点云分为有组织以及无组织两种:
无组织：width就是云中点的个数，height为1
有组织：width是点云数据集一行上点的个数，height为总行数
也可以使用cloud.isOrganized()进行判断是否有组织.

is_dense(bool)
判断云中的点是否包含无效值(infinite或者NaN)，或者说points的数据是否有限
数据有限、不包含无效值为true，反之为false

header(pcl::PCLHeader)
指定点云的获取时间

sensor_origin_(Eigen::Vector4f)
代表了相应传感器的朝向，可选，多数情况不会使用
sensor_orientation_(Eigen::Quaternion)
代表了相应传感器的位置坐标，可选，多数情况不会是用

points(std::vector)
存储的是一个动态数组(数据类型为PointT),即
pcl::PointCloud<pcl::PointXYZ> cloud;
std::vector<pcl::PointXYZ> data = cloud.points;

而点云中的点的数据结构可以有下面多种形式：
（1）pcl::PointCloud<pcl::PointXYZ>
PointXYZ 成员：float x，y，z;表示了xyz3D信息，可以通过points[i].data[0]或points[i].x访问点X的坐标值
（2）pcl::PointCloud<pcl::PointXYZI>
PointXYZI成员：float x, y, z, intensity; 表示XYZ信息加上点的密集程度。
（3）pcl::PointCloud<pcl::PointXYZRGB>
PointXYZRGB 成员：float x,y,z,rgb; float rgb; 表示XYZ信息加上RGB信息
（4）pcl::PointCloud<pcl::PointXYZRGBA>
PointXYZRGBA 成员：float x , y, z; uint32_t rgba; 表示XYZ信息加上RGBA信息，A是alpha，透明度
（5）pcl::PointCloud<pcl::Normal>
Normal 成员：float normal_x,normal_y,normal_z,curvature;表示给定点所在曲面上的法线方向以及对应曲率的测量值

除了上面这些点的类型，还有其他标准的pcl类型，用户也能自定义类型。
详情请参见：https://blog.csdn.net/qq_41685265/article/details/104448675?ops_request_misc=%7B%22request%5Fid%22%3A%22158316545819725256763929%22%2C%22scm%22%3A%2220140713.130056874..%22%7D&request_id=158316545819725256763929&biz_id=0&utm_source=distribute.pc_search_result.none-task

pcl::PointCloud 与  sensor_msgs::PointCloud
pcl::PointCloud是用以进行数据处理的类型，而sensor_msgs::PointCloud是ROS中进行消息传递的数据类型


PCL与ROS：需要包含头文件<pcl_conversions>
ROS提供的消息中点云的数据类型一般为sensor_msgs::PointCloud2
1、ROS转PCL数据格式 -sensor_msgs::PointCloud2 转 pcl::PCLPointCloud2
使用函数 pcl_conversions::toPCL (sensor_msgs::PointCloud2, pcl::PCLPointCloud2) 
		   -sensor_msgs::PointCloud2 转 pcl::PointCloud<pcl::PointXYZ>
使用函数 pcl::fromROSMsg (sensor_msgs::PointCloud2, pcl::PointCloud<pcl::PointXYZ>) 

2、PCL转ROS数据格式 -pcl::PCLPointCloud2 转 sensor_msgs::PointCloud2
使用函数 pcl_conversions::fromPCL (pcl::PCLPointCloud2, sensor_msgs::PointCloud2)
		   -pcl::PointCloud<pcl::PointXYZ> 转 sensor_msgs::PointCloud2
使用函数 pcl::toROSMsg (pcl::PointCloud<pcl::PointXYZ>, sensor_msgs::PointCloud2)

pcl::PCLPointCloud2与pcl::PointCloud<pcl:PointXYZ>等，
详情请参见：https://blog.csdn.net/liukunrs/article/details/80319952?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task


Pcl::PointCloud和pcl::PointCloud::Ptr类型（指针类型）的转换：
PointCloud::Ptr—>PointCloud
pcl::PointCloud<pcl::PointXYZ> cloud;
pcl::PointCloud<pcl::PointXYZ>::Ptr cloud_ptr(new pcl::PointCloud<pcl::PointXYZ>);
cloud=*cloud_ptr;

PointCloud—>PointCloud::Ptr
pcl::PointCloud<pcl::PointXYZ>::Ptr cloud_ptr(new pcl::PointCloud<pcl::PointXYZ>);
pcl::PointCloud<pcl::PointXYZ> cloud;
cloud_ptr=cloud.makeShared();

PCD文件与其写入读取
#include <pcl/io/pcd_io.h>
PCD是用来储存一张点云照片的文件，后缀名为.pcd;
PCD文件的写入：
pcl::PointCloud<pcl::PointXYZ> cloud;
pcl::io::savePCDFileASCII("地址/pcd_file_name.pcd", cloud);

PCD文件的读取：
pcl::PointCloud<pcl::PointXYZ>::Ptr cloud(new pcl::PointCloud<pcl::PointXYZ>);
pcl::io::loadPCDFile<pcl::PointXYZ>("地址/pcd_file_name.pcd", *cloud);


体素滤波：体素滤波相当于图像处理的采样
pcl::PointCloud<pcl::PointXYZI>::Ptr cloud(new pcl::PointCloud<pcl::PointXYZI>);
pcl::PointCloud<pcl::PointXYZI>::Ptr cloud_filter(new pcl::PointCloud<pcl::PointXYZI>);
pcl::VoxelGrid<pcl::PointXYZI> sor;
sor.setInputCloud (cloud);
sor.setLeafSize (0.1f, 0.1f, 0.1f);//相当于采样的尺寸
sor.filter(*cloud_filter);















