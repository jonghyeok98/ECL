# Opencv

<br>

## 시작하기

<br>

### visual studio에서 opencv를 사용하기 위해선 다음과 같이 설정해야 한다

> 참고) https://webnautes.tistory.com/1132




### 이미지 불러오기

``` cpp
imread("filename", flag)
```

* filename: 불러올 이미지의 경로
* flag: 옵션
    > 1. `1`, `IMREAD_COLOR`: 이미지 파일을 색 정보(Color)를 포함해서 불러온다, default
    > 2. `0`, `IMREAD_GRAYSCALE`: 이미지 파일을 흑백(Grayscale)으로 변환해서 불러온다
    > 3. `-1`, `IMREAD_UNCHANGED`: 이미지 파일을 색정보에 알파채널(Alpah channel)까지 포함해서 불러온다.

<br>

### Mat Class
> Matirx의 약자로 행렬을 표현하기 위한 데이터

* C++ 스타일의 N차원 고밀도 배열 클래스이며 행렬(2차원)을 비롯해 배열(1, 2, 3차원)을 효율적으로 표현할 수 있다

Mat 클래스는 헤더(Header)와 데이터 포인터(Data Pointer)로 구성되어 있다
> * 헤더: 에는 Mat 클래스에 대한 정보(행렬의 크기, 깊이 등)가 저장
> * 데이터 포인터: 각 데이터가 담겨있는 메모리 주소 정보가 저장

<br>

### 이미지 보여주기


``` cpp
imshow("windowName", image)
```

* windowName: 이미지를 보여줄 윈도우 창의 이름
* image: 보여주고자 하는 <Mat> 형식의 변수 이름

<br>

``` cpp
waitKey(delay time);
```

* delay time: 키 입력을 기다릴 시간
    > 0을 입력하면 무한 대기

> imshow() 함수를 사용하게 되면 waitKey() 함수가 필수적으로 따라와야 한다.
 * waitKey() 함수를 사용하지 않으면 이미지가 노출되자마자 코드가 종료됨

<br>

### 직교 좌표를 구 좌표로 변환

<br>

<img src="./Images/Opencv/pos.png" width = 500>

<br>

``` cpp
radius = sqrt(x*x + y*y + z*z)
theta = atan2(y, x)
phi = acos(z / radius) // or phi = atan2(y, x)
```

<br>



ref) 
* https://stackoverflow.com/questions/34250742/converting-a-cubemap-into-equirectangular-panorama
* http://paulbourke.net/panorama/cubemaps/
* https://stackoverflow.com/questions/29678510/convert-21-equirectangular-panorama-to-cube-map
* https://github.com/trotlinebeercan/360-pano-to-cubemap/blob/master/PanoToCube.h