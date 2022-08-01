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

imshow() 함수를 사용하게 되면 waitKey() 함수가 필수적으로 따라와야 한다.
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

### 코드
``` cpp
#include <opencv2/core.hpp>
#include <opencv2/videoio.hpp>
#include <opencv2/highgui.hpp>
#include <iostream>
#include <cmath>

using namespace cv;
using namespace std;

#define PI    3.141592653589793




// 함수 선언부

// 큐브맵
float* GetCubemapCoordinate(float* p, int x, int y, int face, int edge);


// panorama to cubemap
Mat SphToCubemap(Mat* sph, float* p);
Mat CubemapToSph(Mat* cube, Mat* sphrical);


int main(int argc, char* argv[])
{
    Mat img = imread("panorama.png", 1);
    float* p = new float[3];       // x, y, z 좌표를 가지는 구조체 선언


    // panorama to cubemap
    
    Mat cubemap = SphToCubemap(&img, p);
    Mat spherical = CubemapToSph(&cubemap, &img);

    imshow("cubemap", cubemap);
    imshow("spherical", spherical);
    waitKey(0);
    
    delete p;
    return 0;
}



// 구면 좌표계 -> 데카르트 좌표계 투영
float* GetCubemapCoordinate(float* p, int x, int y, int face, int edge)
{
    //    -1 ~1 사이 정규화된 좌표값 내에서 point를 정해주어야 한다.
    //    즉, 2의 길이가 필요하기 때문에 a, b에 2를 곱해준다.
    //    edge < y < 2 * edge 사이 b = 2 * 1. ? ? ? ? ? ? f이고, 2~4의 범위를 갖는다.

    float a = 2 * (1.0 * x / edge);
    float b = 2 * (1.0 *y / edge);

    p[0] = 0;      // x 좌표 초기화
    p[1] = 0;      // y 좌표 초기화
    p[2] = 0;      // z 좌표 초기화

    if (face == 0)    // back
    {
        p[0] = -1.0;
        p[1] = 1.0 - a;
        p[2] = 3.0 - b;
    }
    else if (face == 1) // left
    {
        p[0] = a - 3.0;
        p[1] = -1.0;
        p[2] = 3.0 - b;
    }
    else if (face == 2) // front
    {
        p[0] = 1.0;
        p[1] = a - 5.0;
        p[2] = 3.0 - b;
    }
    else if (face == 3) // right
    {
        p[0] = 7.0 - a;
        p[1] = 1.0;
        p[2] = 3.0 - b;
    }
    else if (face == 4) // top
    {
        p[0] = a - 3.0;
        p[1] = 1.0 - b;
        p[2] = 1.0;
    }
    else if (face == 5) // bottom
    {
        p[0] = a - 3.0;
        p[1] = b - 5.0;
        p[2] = -1.0;
    }
    return p;
}

// Panorama To Cubemap
Mat SphToCubemap(Mat *sph, float* p)
{
    /*
       구면 파노라마 이미지 너비, 높이 구하기
       이 때 cubeWidth는 6칸(Back, Left, Front, Right, Top, Bottom)으로 나누어진 맵의 가로값이다
       즉, 파노라마 이미지의 가로값과 같다
       cubeHeight는 맵의 세로값. 즉 (Top, Left, Bottom) 세 칸을 차지함.
       본 이미지는 1:2 비율을 갖고 4칸:3칸이므로 0.75f를 곱해줌.
    */

    int height = 0.75 * sph->size().width;
    int width = sph->size().width;
    
    // edge(한 큐브맵의 선)는 정사각형의 한 선이므로
    // width / 4
    int face, startidx, range;        
    int edge = sph->size().width / 4;

    // 경도를 나타내는 phi
    // 위도를 나타내는 theta
    float theta, phi;
    float u, v;

    Mat cube;
    cube = Mat::zeros(height, width, sph->type());


    // 큐브맵의 가로(width)는 동일한 상태에서
    // 가로4칸 세로3칸을 수행하기 위해 
    // face가 1(가로) 혹은 4, 5(세로) 일 경우의 범위를 다르게 한다
    for (int i = 0; i < width; i++)
    {
        face = i / edge;

        // left일 경우 가로이기 때문에 top, bottom은 고려할 대상이 아니다
        // face = 0, (1, 4, 5), 2 ,3 으로 진행한다

        if (face == 1)  // left(top, bootm)
        {
            startidx = 0;
            range = 3 * edge;
        }
        else
        {
            startidx = edge;
            range = 2 * edge;
        }

        // face가 left일 때 아래 조건에 걸려 face가 4, 5로 변경되는 것을 막아준다
        int priv_face = face;

        for (int j = startidx; j < range; j++)
        {
            if (j < edge)
                face = 4;     // top
            else if (j >= 2 * edge)
                face = 5;     // bottom

            // 1. 파노라마 이미지를 큐브맵 좌표계에 배치
            //      - top, bootom, front, back, right, left중 어떤 면에 위치했는지
            //      - 큐브맵 좌표를 얻는다
            // 2. (x, y, z)로 표현되는 큐브맵 좌표를 (r, theta, phi)로 표현되는
            //    구면 좌표계로 변환한다
            // 3. 구면 좌표계를 구면 파노라마 좌표게로 변환한다
            // 4. 해당되는 좌표의 정보를 큐브맵 좌표에 대입한다

            // 구면 좌표계 -> 큐브맵의 데카르트 좌표계 투영
            p = GetCubemapCoordinate(p, i, j, face, edge);


            // atan2(a, b)는 웑머에서 (a, b) 까지의 상대적인 각도(위치)
            // phi는 경도로 원점에서 (y, x)까지의 상대적인 각도
            // 이 때 (y, x)는 큐브맵 위의 한 face 위 점이므로 구면 파노라마의 
            // 한 좌표로 표현하기 위해 atan2를 사용한다
            // 
            // theta는 위도이고 -90 ~ 90 범위를 가진다
            // 범위를 벗어나게 하지 않기 위해 x, y 사이 거리를 2번째 인자로 전달한다
            // 원점이 기준이기 때문에 원점과 (z, 거리) 사이의 각도
            phi = atan2(p[1], p[0]);
            theta = atan2(p[2], sqrt((p[0] * p[0]) + (p[1] * p[1])));

            // (phi + PI)/PI 는 phi 만큼 회전한 구면 위의 이미지 상 (x, y) 좌표
            // ((PI/2) - theta)/PI 는 범위 내에서 theta 만큼 횐전한 구면 위의
            // 이미지 상 (x, y, z) 좌표
      
            u = 2 * edge * ((phi + PI) / PI);
            v = 2 * edge * (((PI / 2) - theta) / PI);

            // 구한 구면 파노라마의 좌표를 큐브맵 좌표에 대입
            cube.at<Vec3b>(j, i) = sph->at<Vec3b>(v, u);
            face = priv_face;
        }

    }
    return cube;
}

Mat CubemapToSph(Mat* cube, Mat* original)
{
    /*
    구면 파노라마 이미지의 길이는 큐브 맵의 길이를 따른다.
    구면 파노라마 이미지의 높이는 큐브 맵 길이의 1/2.

    1. 구면 파노라마 이미지 좌표(j, i)를 정규화, 구면 좌표계로 변환한다.
    2. 구면 좌표계에 대응하는 큐브맵의 좌표를 찾는다.
    3. 찾은 큐브맵의 좌표가 어느 face에 있는지 찾는다.
    4. 큐브맵의 좌표를 구면 파노라마 이미지의 좌표(j, i)에 대입한다.
*/
    int Width = cube->size().width;
    float Height = (0.5f) * cube->size().width;

    Mat spherical;
    spherical = Mat::zeros(original->size().height, original->size().width, cube->type());

    /*
        좌표계를 0부터 1로 정규화 한다. (0, 0)
        경도를 나타내기 위한 변수 phi
        위도를 나타내기 위한 변수 theta
    */
    float u, v;
    float phi, theta;
    int cubeFaceWidth, cubeFaceHeight;

    cubeFaceWidth = cube->size().width / 4;
    cubeFaceHeight = cube->size().height / 3;


    for (int j = 0; j < Height; j++)
    {
        /*
            (i = 0, j = 0) 부터 j를 높이까지 증가.
            즉, 구면의 위도 생성.
            왼쪽 아래부터 시작.
        */
        v = 1 - ((float)j / Height);
        theta = v * PI;

        for (int i = 0; i < Width; i++)
        {
            // 위도 상의 한 점(0부터)에서 경도 끝까지 증가
            u = ((float)i / Width);
            phi = u * 2 * PI;

            float x, y, z; // 단위 벡터
            x = cos(phi) * sin(theta) * -1;
            y = sin(phi) * sin(theta) * -1;
            z = cos(theta);

            float xa, ya, za;
            float a;

            a = max(abs(x), max(abs(y), abs(z)));

            /*
                큐브 면 중 하나에 있는 단위 벡터와 평행한 벡터.
                이 때, ya가 -1인지 1인지(Left, Right) 값을 보고 평면을 결정.

                ya가 1 or -1이라면 y벡터의 변화가 없다는 뜻. 즉 xz평면만 고려한다는 의미.
                xa와 za도 동일하게 적용.
            */
            xa = x / a;
            ya = y / a;
            za = z / a;

            int xPixel, yPixel;
            int xOffset, yOffset;


            /*
                1. 정규화를 거친 좌표계이기 때문에 2.f로 나누어준다.
                2. -1 ~ 1로 정규화 되어있는 좌표계에 edge 길이(cubeFaceWidth, cubeFaceHeight)를 곱해준다.
                3. WorldOffset에 적용한다. 이때 Offset은 큐브맵 좌표계에 따른다.
            */
            if (ya == -1)
            {
                //Left
                xPixel = (int)((((xa + 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = cubeFaceWidth;
                yPixel = (int)((((za + 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = cubeFaceHeight;
            }
            else if (ya == 1)
            {
                //Right
                xPixel = (int)((((xa - 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = 3 * cubeFaceWidth;
                yPixel = (int)((((za + 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = cubeFaceHeight;
            }
            else if (za == -1)
            {
                //Top
                xPixel = (int)((((xa + 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = cubeFaceWidth;
                yPixel = (int)((((ya - 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = 0;
            }
            else if (za == 1)
            {
                //Bottom
                xPixel = (int)((((xa + 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = cubeFaceWidth;
                yPixel = (int)((((ya + 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = 2 * cubeFaceHeight;
            }
            else if (xa == 1)
            {
                //Front
                xPixel = (int)((((ya + 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = 2 * cubeFaceWidth;
                yPixel = (int)((((za + 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = cubeFaceHeight;
            }
            else if (xa == -1)
            {
                //Back
                xPixel = (int)((((ya - 1.f) / 2.f)) * cubeFaceWidth);
                xOffset = 0;
                yPixel = (int)((((za + 1.f) / 2.f)) * cubeFaceHeight);
                yOffset = cubeFaceHeight;
            }
            else
            {
                xPixel = 0;
                yPixel = 0;
                xOffset = 0;
                yOffset = 0;
            }
            xPixel = abs(xPixel);
            yPixel = abs(yPixel);

            xPixel += xOffset;
            yPixel += yOffset;

            spherical.at<Vec3b>(j, i) = cube->at<Vec3b>(yPixel, xPixel);
        }
    }
    return spherical;
}
```


ref) 
* https://stackoverflow.com/questions/34250742/converting-a-cubemap-into-equirectangular-panorama
* http://paulbourke.net/panorama/cubemaps/
* https://stackoverflow.com/questions/29678510/convert-21-equirectangular-panorama-to-cube-map
* https://github.com/trotlinebeercan/360-pano-to-cubemap/blob/master/PanoToCube.h