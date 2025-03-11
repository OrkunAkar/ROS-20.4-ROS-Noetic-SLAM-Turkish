[LIDAR KÜTÜPHANELERİ 101 9968236c30e246f29f5e782da5747fad.md](https://github.com/user-attachments/files/19199036/LIDAR.KUTUPHANELERI.101.9968236c30e246f29f5e782da5747fad.md)
# LIDAR KÜTÜPHANELERİ 101

**ROS NOETIC**

Dökümantasyonun amacı Lidar kütüphanelerinin Ros Noetic ortamına entegrasyonudur. Ana [kaynağın](https://blog.csdn.net/zardforever123/article/details/125570551) özet halidir. GTSAM4.0, PCL1.10 temel gereksinim olarak ifade edilebilir. Algoritmaların karakteristik özellik ve modifikasyonlarına yer verilmiştir.

“Bak Kutay, bugün sen ‘arı’ olabil diye çok büyük bedeller ödendi. Bunu sakın unutma olur mu?”

A-LOAM: Temel LOAM kütüphanesinin her sistemde çalışması için optimize edilmiş halidir, ROS-1 ortamında çalışır. Verilerin doğruluğunu arttırmak için IMU’ya ihtiyaç duyar.

[LeGO-LOAM](https://github.com/RobustFieldAutonomyLab/LeGO-LOAM): Karasal kompakt keşif araçları için özel geliştirilmiş bir pakettir, taramalarda sürekli bir zeminin var olduğunu var sayar. IMU gereksinim değil, opsiyondur, IMU kullanımında 9-DOF IMU tavsiye edilir. “Loop Closure” özelliği ilkeldir.

Karakteristik Modifikasyonlar;

i- catkin_ws --> src --> “LeGO-LOAM” --> “CMakeLists.txt”

#find_package(OpenCV REQUIRED QUIET) -->

set(CMAKE_PREFIX_PATH “/usr/include/opencv4”)

find_package(OpenCV 4.0 QUIET)

ii- catkin_ws --> src --> “LeGO-LOAM” --> include --> “utility.h”

#include <opencv/cv.h> --> #include <opencv2/imgproc.hpp>

iii- catkin_ws --> src --> “LeGO-LOAM” --> “LeGO-LOAM” --> src --> “.cpp uzantılı 4 dosya”

Buradaki 4 dosyayının içeriğini “/camera_init” --> “camera_init” olacak şekilde değiştirin.

SC-LEGO-LOAM: LeGO-LOAM ve ScanContext in füzyonu ile elde edilmiştir. LeGO-LOAM'da yetersiz kalan “Loop Closure” özelliği bu kütüphanenin en güçlü yanıdır.

LVI-SAM: Lio-sam ve Vins-Mono teknolojilerini kullanır. Kamera, Lidar, IMU ve GPS kullanarak haritalamadaki hata oranını en aza indirmeyi amaçlar.

LIO-SAM: LeGO-LOAM'ın gelişmiş versiyonudur. GPS (Reach RS+) kullanarak “point cloud” verisi ile füzyon eder. Yüksek doğruluk oranı ile çalışır. 9 axisli IMU’ya ihtiyaç duyar (Microstrain 3DM-GX5-25).

-6-AXIS FIX: just comment the _imuPreintegration related line in module_loam.launch. And set the "odom" as the fixed_frame in rviz. Then you can see the map.

**Genel Kurulum**

- Kuruluma LIO-SAM'den başlamanız yararınıza olacaktır. Devam eden süreçte diğer kütüphaneleri kurmak istediğinizde PCL ve GTSAM sürümleri değişmeyeceğinden “workspace”e aşağıda belirtilen modifikasyonlar ile kurulum yapmanız yeterli olacaktır.
- ”[catkin workspace](https://wiki.ros.org/catkin/Tutorials/create_a_workspace)”, “[ROS Noetic](https://wiki.ros.org/noetic/Installation/Ubuntu)” kurulu kabul edilir (Eğer bahsedilen paketler kurulu değilse önce ROS Noetic kurulumunu yapın). Aşağıdaki komutları kopyala-yapıştır yaparken sorunla karşılaşılması halinde kodları önce boş bir sekmeye yapıştırıp oradan kopyalayıp terminale yapıştırılması tavsiye edilir.

i-[Repo](https://github.com/TixiaoShan/LIO-SAM)'dan temel ROS paketleri indirilir (“kinetic”i, “noetic” olarak değiştir).

sudo apt-get install -y ros-noetic-navigation

sudo apt-get install -y ros-noetic-robot-localization

sudo apt-get install -y ros-noetic-robot-state publisher

- *son kodda hata vermesi halinde = “sudo apt-get update” çalıştırılır

ii-[Repo](https://github.com/TixiaoShan/LIO-SAM)'dan gtsam aynen kurulur.

sudo add-apt-repository ppa:borglab/gtsam-release-4.0

sudo apt install libgtsam-dev libgtsam-unstable-dev

iii-[PCL 1.10](https://github.com/PointCloudLibrary/pcl/archive/refs/tags/pcl-1.10.0.tar.gz) kurulur

- İndirilen dosyayı bulunduğu konuma çıkartın.
- Home dizininde klasör oluşturup (Software), çıkarttığınız dosyayı buraya kopyalayın. Kopyaladığınız dosayaya tıklayın, “Release” adlı yeni bir klasör oluşturun.
- ”Release” klasöründe terminali açın, sırası ile;

> cmake -DCMAKE_BUILD_TYPE=None -DCMAKE_INSTALL_PREFIX=/usr \ -DBUILD_GPU=ON-DBUILD_apps=ON -DBUILD_examples=ON \ -DCMAKE INSTALL_PREFIX=usr ..
> 
> 
> make
> 
> sudo make install
> 

iV-[Lio-sam](https://github.com/TixiaoShan/LIO-SAM.git) git clone ile work space’e kurulur.

cd ~/catkin_ws/src

git clone [https://github.com/TixiaoShan/LIO-SAM.git](https://github.com/TixiaoShan/LIO-SAM.git)

cd ..

catkin_make -j1

**GENEL MODİFİKASYONLAR**

Bazı algoritmalarda buradaki değişiklere ek birkaç değişiklik daha yapılmaktadır, üstte algoritmaların açıklamasında bahsedilmiştir.

i- catkin_ws --> src --> “modifikasyon yapılacak algoritma” --> “CMakeLists.txt”

set(CMAKE_CXX_FLAGS "-std=c++11") --> set(CMAKE_CXX_FLAGS "-std=c++14")

ii- catkin_ws --> src --> “modifikasyon yapılacak algoritma” --> include --> “utility.h”

#include <opencv/cv.h> --> #include <opencv2/opencv.hpp>

#include <opencv2/opencv.hpp> değiştirdiğiniz header’ı pcl satırlarının hemen altına yapıştırın

> -A. ORKUN AKAR
>
