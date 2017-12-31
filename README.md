# VRA

Thực hiện chương trình

Cài đặt YOLO trên hệ điều hành Windows

Yêu cầu cài đặt các chương trình:

-	Visual Studio 2015 (https://go.microsoft.com/fwlink/?LinkId=615448&clcid=0x409)

-	CUDA 8.0 (https://developer.nvidia.com/cuda-downloads)

-	OpenCV 3.2.0 vc14 (https://sourceforge.net/projects/opencvlibrary/files/opencv-win/3.2.0/opencv-3.2.0-vc14.exe/download)

-	GPU CC >= 3.0

Cấu hình máy tính sử dụng để train:

-	CPU: Intel Core i5-4460 3.2GHz

-	RAM: 8GB

-	GPU: GTX 1050 6033MB

Cài đặt darknet:

B1. Clone project darknet từ github: 

git clone https://github.com/AlexeyAB/darknet.git

B2. Chỉnh sửa một số thông số môi trường cho phù hợp các chương trình đã cài đặt ở trên (OpenCV, CUDA) trong file \darknet\build\darknet\darknet.vcxproj

B3. Chạy file solution \darknet\build\darknet\darknet.sln

B4. Build Project với tùy chọn Release và x64 để có được chương trình darknet (\darknet\build\darknet\x64\darknet.exe).

Chuẩn bị dữ liệu

Cài đặt Yolo_mark (https://github.com/AlexeyAB/Yolo_mark)

Đây là phần mềm sử dụng để xác định bounding box cho các ảnh train. Phần mềm này phù hợp cho Yolo v2.

B1. Clone project Yolo_mark từ github: 

git clone https://github.com/AlexeyAB/Yolo_mark.git

B2. Chỉnh sửa một số thông số môi trường cho phù hợp các chương trình đã cài đặt ở trên (OpenCV, CUDA) trong file \Yolo_mark\ yolo_mark.vcxproj

B3. Chạy file solution \Yolo_mark\yolo_mark.sln

B4. Build Project với tùy chọn Release và x64 để có được chương trình Yolo_mark (\Yolo_mark\x64\Release\yolo_mark.exe).

Xác định bounding box cho dữ liệu train

B1. Xóa tất cả file trong folder x64/Release/data/img.

B2. Copy file ảnh cần xác định bounding box vào folder này (x64/Release/data/img).

B3. Chỉnh số lượng đối tượng trong file x64/Release/data/obj.data

Do đồ án chỉ detect 1 đối tượng nên file obj.data có cấu trúc như sau:

classes= 1

train  = data/train.txt

valid  = data/test.txt

names = data/obj.names

backup = backup/

B4. Đặt tên cho đối tượng vào file x64/Release/data/obj.names

thanhlong

B5. Chạy ứng dụng bằng cách chạy file: x64\Release\yolo_mark.cmd

B6. Dữ liệu train gồm 50 hình. Tiến hành chọn đối tượng, ưu tiên chọn những trái nguyên vẹn, ít bị che khuất:

 

Chương trình sẽ tạo file txt dùng để lưu vị trí của bounding box, tên file trùng với tên ảnh: 

 



Nội dung file:

0 0.373437 0.620139 0.384375 0.759722

0 0.733984 0.557639 0.332031 0.695833

[số thứ tự của object] [vị trí giữa của object theo chiều X] [vị trí giữa của object theo chiều Y] [kích thước của object theo chiều X] [kích thước của object theo chiều Y]

Thông tin dữ liệu train sẽ được lưu vào file \x64\Release\data\train.txt.

Tách dữ liệu trên thành 40 hình train và 10 hình test. 10 hình test được lưu đường dẫn vào file \x64\Release\data\test.txt (tên file này đã được cung cấp ở file obj.data).

Chuẩn bị các file config cho Yolo v2 

obj.data

obj.names

yolo-thanhlong.cfg

Config cho YoloV2 bao gồm 3 file, 2 file obj.data và obj.names đã được tạo ở trên. File yolo-thanhlong.cfg được chỉnh sửa từ một config có sẵn là yolo-voc.cfg. 

	Nội dung chỉnh sửa gồm:

-	batch=12 là sử dụng 12 trong mỗi bước trainning.

-	subdivisions=12 là chia 12 của batch cho mỗi bước trainning. Các tùy chọn này được thiết lập tùy thuộc phần cứng máy tính (card đồ họa) để tránh tình trạng CUDA out of memory error.

-	classes=1 là số lượng object muốn train.

-	filters=(classes + 5)*5 là số lượng filter, ở đây là 30 do classes = 1.

Để bắt đầu train cần file convolutional layers có sẵn:  .

Tổng hợp các thành phần cần thiết để train:

 

Trainning

B1. Copy các file trong Yolo_mark\data vào \darknet\build\darknet\x64\data

B2. Chạy lệnh sau trên cmd để dùng darknet train dữ liệu (không xuống dòng):

darknet.exe detector train data/obj.data data/yolo-thanhlong.cfg data/darknet19_448.conv.23

 



Màn hình load config ban đầu:

 

Chương trình sẽ resize ảnh và train theo file config đã dùng:

 



GPU Usage khi train

 

Khi nào thì dừng train?

Darknet đưa ra rất nhiều thông số quan trọng của mỗi bước train:

 

Tuy nhiên để xác định khi nào cần dừng train ta lưu ý đến 2 thông số được bôi ở trên:

- 381 – Số lần lặp (dựa trên số batch mà ta thiết lập ở trên)

- 0.272537 avg – Giá trị lỗi trung bình – Càng nhỏ càng tốt

Theo đề xuất của tác giả thì khoảng 0.0x avg thì có thể dừng train, hoặc nếu giá trị này không còn giảm nữa qua các vòng lặp.

Hoặc chương trình sẽ dừng ở 45000 lần lặp.

Để dừng train ta chỉ cần tắt chương trình.

Dữ liệu trọng số .weights sẽ được lưu vào \darknet\build\darknet\x64\backup

 

Test model

Để test hình data/test/test_thanhlong.jpg (là hình không nằm trong 50 hình train), với model đã train backup/yolo-thanhlong_1000.weights Chạy lệnh: 

darknet.exe detector test data/obj.data data/yolo-thanhlong.cfg backup/yolo-thanhlong_1000.weights data/test/test_thanhlong.jpg

