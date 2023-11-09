# 라즈베리파이에서 네오픽셀 적용기

> 주의: putty를 사용하는 경우 처음 실행하면 잘 안될 수 있음 -> 라즈베리파이 모니터를 사용해서 로컬에서 돌려보면 정상적으로 작동

### NeoPixels on Raspberry Pi

[NeoPixels on Raspberry Pi](https://learn.adafruit.com/neopixels-on-raspberry-pi/python-usage)

무작정 링크의 첫단에 들어가 install을 하려 했다.

$ `sudo pip3 install rpi_ws281x adafruit-circuitpython-neopixel`

→ setuptools-scm이 없다면서 에러 발생

\~_기본 실습용 라즈비안 이미지가 너무 옛날 버전이라서 그런지 파이썬 버전이 낮아서 pip install, apt-get update를 비롯한 대부분의 명령어가 다 에러나는 상황_ \~

> #### **이제부터 하나하나씩 해결해보자,,,🐾**

### Installing CircuitPython Libraries on Raspberry Pi

네오픽셀 라이브러리를 설치하기 위해서 필요한 기본 셋업을 다룬 다음 문서를 보고 차례로 설치하며 setuptools 문제를 해결해보자,,,

[CircuitPython on Linux and Raspberry Pi](https://learn.adafruit.com/circuitpython-on-raspberrypi-linux/installing-circuitpython-on-raspberry-pi)

```sh
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install python3-pip
```

#### 👊 apt-get update 왜 안될까

apt-get update를 하다보면 404 Error가 뜨면서 사이트에서 다운이 되지 않는 문제 발생,, 다음 링크 내용으로 해결할 수 있다

[\[Raspbian\] sudo apt-get update ERROR](https://baked-corn.tistory.com/37)

→ $ `sudo vi /etc/apt/sources.list`

```python
deb http://mirror.ox.ac.uk/sites/archive.raspbian.org/archive/raspbian stretch main contrib non-free rpi
deb-src http://mirror.ox.ac.uk/sites/archive.raspbian.org/archive/raspbian stretch main contrib non-free rpi
```

sources.list 파일을 위와 같이 수정해준다(기존내용 삭제 가능)

그리고 다시

$ `sudo apt-get update`

$ `sudo apt-get upgrade`

를 실행해보면 정상적으로 동작한다. (시간 꽤걸림)

$ `sudo apt-get install python3-pip`

$`sudo pip3 install --upgrade setuptools`

**이번엔 또 뭐가 문제임? → python 3.6이상 필요함 ㅇㅇ** (문제 없으면 넘어가도 됨)

#### 👊 python을 업데이트 해보자

[라즈베리파이4 파이썬 3.8 설치와 pip3 install 에러를 해결하기까지](https://redfox.tistory.com/11)

이 링크 덕분에 pip까지 성공..!

**cmd에서 아래 내용만 입력하면 됨**



```sh
$ cd ~
$ sudo apt-get install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev
$ wget https://www.python.org/ftp/python/3.8.7/Python-3.8.7.tgz
$ sudo tar xzf Python-3.8.7.tgz
$ cd Python.3.8.7
$ ./configure
$ make
$ sudo make install
$ pip3 -V, pip3 list 로 잘 설치되었는지 확인
$ python3 -m pip install --upgrade pip
```

이제 pip3 install \~\~를 에러없이 진행할 수 있다.

### NeoPixels on Raspberry Pi

[NeoPixels on Raspberry Pi](https://learn.adafruit.com/neopixels-on-raspberry-pi/python-usage)

다시 처음 링크로 돌아와서

$ `sudo pip3 install rpi_ws281x adafruit-circuitpython-neopixel`

$ `sudo python3 -m pip install --force-reinstall adafruit-blinka`

돌릴 테스트코드 작성

`test.py`

```python
import board
import neopixel
pixels = neopixel.NeoPixel(board.D18, 30)
pixels[0] = (255, 0, 0)
```

핀은 다음과 같이 연결한다. ![](https://images.velog.io/images/oliviarla/post/3457a62a-f639-4dcd-beb5-9e5a00b14e8c/image.png)

$ `sudo python3 test.py` 돌려서 LED하나 나오면 성!공!

이렇게 설치된 네오픽셀을 다양하게 활용할 수 있다

### Example Code

아래는 무지개 돌리기 코드 rainbow.py

```python
# SPDX-FileCopyrightText: 2021 ladyada for Adafruit Industries
# SPDX-License-Identifier: MIT

# Simple test for NeoPixels on Raspberry Pi
import time
import board
import neopixel


# Choose an open pin connected to the Data In of the NeoPixel strip, i.e. board.D18
# NeoPixels must be connected to D10, D12, D18 or D21 to work.
pixel_pin = board.D18

# The number of NeoPixels
num_pixels = 30

# The order of the pixel colors - RGB or GRB. Some NeoPixels have red and green reversed!
# For RGBW NeoPixels, simply change the ORDER to RGBW or GRBW.
ORDER = neopixel.GRB

pixels = neopixel.NeoPixel(
    pixel_pin, num_pixels, brightness=0.2, auto_write=False, pixel_order=ORDER
)


def wheel(pos):
    # Input a value 0 to 255 to get a color value.
    # The colours are a transition r - g - b - back to r.
    if pos < 0 or pos > 255:
        r = g = b = 0
    elif pos < 85:
        r = int(pos * 3)
        g = int(255 - pos * 3)
        b = 0
    elif pos < 170:
        pos -= 85
        r = int(255 - pos * 3)
        g = 0
        b = int(pos * 3)
    else:
        pos -= 170
        r = 0
        g = int(pos * 3)
        b = int(255 - pos * 3)
    return (r, g, b) if ORDER in (neopixel.RGB, neopixel.GRB) else (r, g, b, 0)


def rainbow_cycle(wait):
    for j in range(255):
        for i in range(num_pixels):
            pixel_index = (i * 256 // num_pixels) + j
            pixels[i] = wheel(pixel_index & 255)
        pixels.show()
        time.sleep(wait)


while True:
    rainbow_cycle(0.001)  # rainbow cycle with 1ms delay per step
```

실행할 때는 꼭 sudo로 실행해야한다고 한다...

&#x20;$ `sudo python3 rainbow.py`&#x20;

<figure><img src="https://images.velog.io/images/oliviarla/post/4dcad81f-8008-4775-b727-c49062cc9fcd/KakaoTalk_20211021_194658093%20(1).gif" alt=""><figcaption><p>영롱..</p></figcaption></figure>
