# G14_video_sticker_app


# 평가 루브릭

아래의 기준을 바탕으로 프로젝트를 평가합니다.

## 평가문항	상세기준

### 1. 웹캠을 통한 스티커앱을 실행하고 다각도로 문제점을 분석하여 보았다.
- 거리, 인원수, 각도 등 다양한 측면에서의 문제점과 기술적 대안을 정리하였다.

### 2. 스티커앱 초기버전이 가진 예외사항들을 분석하여 수정하였다.
- 프레임 좌표범위 예외처리 관련 오류를 수정하였다.

### 3. 칼만 필터를 웹캠 스티커앱에 적용하여 스티커의 안정성을 높여 보았다.
- 칼만 필터를 적용한 스티커앱을 작성하여 실행해 보고 안정성 여부를 확인하였다.


# ★ 프로젝트 (2) 어디까지 만들고 싶은지 정의하기

### 1. 웹캠을 이용한 실시간 스티커앱을 만들어주세요.

- 코드를 webcam_sticker.py에 저장함

### 2. 스티커앱을 실행하고 얼굴을 찾지 못하는 거리를 기록해주세요.

- 일반적으로 약 15ch ~ 1m 30cm 범위 사이에서 얼굴 인식이 가능하다고 하는데, 
- ***실제로 측정했을 때는 20cm ~ 1m 50m 범위 사이에서 얼굴 인식이 가능했다.***

### 3. yaw, pitch, roll 각도의 개념을 직접 실험해 보고, 정상적으로 스티커앱이 동작하는 범위를 기록해주세요.

- 일반적인 허용 범위는 아래와 같다고 알려져 있습니다.

    1. yaw (y축 기준 회전 → 높이 축) : -45 ~ 45도
    2. picth (x축 기준 회전 → 좌우 축) : -20 ~ 30도
    3. roll (z축 기준 회전 → 거리 축) : -45 ~ 45도
    

- ***실제 측정해 본 결과는 좀 더 못 미치는 것으로 측정되었다.***

    1. ***yaw : -50 ~ 50도***
    2. ***picth : -20 ~ 20도***
    3. ***roll : -50 ~ 50도***


### 4. 만들고 싶은 스티커앱의 스펙(허용 거리, 허용 인원 수, 허용 각도, 안정성)을 정해주세요.

- 기준의 이유를 어떻게 정했는지가 중요합니다. → 서비스 관점, 엔지니어링 관점으로 설명하면 좋습니다.

    1. ***거리 : 30cm ~ 60m → 너무 멀면 스티커가 거의 식별되지 않으므로 손으로 찍는다는 가정하에서 좀 더 가깝게 설정했다.***
    2. ***인원 수 : 4명 → 4인이 초과되면 화면에 개개인이 잘 나오지 않으므로 스티커를 붙여도 효과가 덜하기 때문이다.***
    3. ***허용 각도 : yaw : -50 ~ 50도, pitch : -20 ~ 20도, roll : -50 ~ 50도 → 편안하게 화면을 바라보면서 스티커가 억지스럽게 붙어있지 않는 각도도 설정했다.***
    4. ***안정성 : 위 조건을 만족하면서 FPPI (false positive per image) 기준 < 0.003, MR (miss rate) < 1 300장당 1번 에러 = 10초=30*10에 1번 에러 → 평범한 사람이 인식할 수 있는 FPS는 30정도 이므로 이 정도면 충분하다고 생각한다.***


# ★ 프로젝트 (3) 스티커 Out Bound 예외처리 하기

- 이전 스텝에서 만든 기본 웹캠 스티커앱의 문제점들을 찾아서 보완해 가도록 합시다.

### 1. 스티커앱에서 발생하는 예외 상황을 꼼고하게 기재해 주세요. 

- 영상에서 왕관을 씌우는 부분이 좌우 경계 밖으로 나가거나, 위 아래 경계를 벋어나면 스티커가 없어진다. 
- 얼굴이 카메라의 왼쪽 경제를 벗어나게 되면 프로그램이 멈추면서 아래와 같은 에러 메시지가 난다.

~~~
- cv2.error: opencv(4.4.0) /tmp/pip-req-build-xgme2194/opencv/modules/core/src/arithm.cpp:666: error: (-209:sizes of input arguments do not match) the operation is neither 'array op array' (where arrays have the same size and the same number of channels), nor 'array op scalar', nor 'scalar op array' in function 'arithm_op'
~~~ 


### 2. 문제가 어디에서 발생하는지 코드에서 확인합니다.

- 얼굴이 카메라 왼쪽 경계를 벗어나서 detection 되는 경우 refined_x 의 값이 음수되어, 
- img_bgr[..., refined_x:...] 에서 numpy array의 음수 index에 접근하게 되므로 예외가 발생한다.

### 3. Out bound 오류(경계 밖으로 대상이 나가서 생기는 오류)를 해결해 주세요.

- newaddsticker.py의 img2sticker 메소드에서 아래 부분을 수정해 주었다. 

~~~
    if refined_y < 0:
       img_sticker = img_sticker[-refined_y:]
       refined_y = 0

    if refined_x < 0:
       mg_sticker = img_sticker[:, -refined_x:]
       refined_x = 0
    elif refined_x + img_sticker.shape[1] >= img_orig.shape[1]:
       img_sticker = img_sticker[:, :-(img_sticker.shape[1]+refined_x-img_orig.shape[1])]
~~~


### 4. 다른 예외는 어떤 것들이 있는지 정의해 주세요. 어떤 것이 문제가 되는지 스스로 정해봅시다.

- 고개를 ***좌우***로 또는 ***상하***로 기울여도 왕관이 함께 기울이지지 않고 일정하게 수직으로 서 있어서 매우 어색하다. → 왕관의 모양도 같이 변형해 주는 로직이 필요하다.
- 고객을 ***앞으로 숙이거나, 뒤로 젖히면*** 왕관이 없어진다. → 얼굴의 코 위치를 인식해서 왕관의 위치를 찾으므로 얼굴의 코 위치가 식별이 되지 않으면 왕관을 붙힐 수가 없기 때문이다. 
- 화면에서 ***거리가 멀어지면*** 왕관이 없어진다. → dlib detector_hog의 이미지 피라미드 수를 변경해서 성능을 향상 시켜 본다. 


# ★ 프로젝트 (4) 스티커앱 분석 - 거리, 인원 수, 각도, 시계열 안정성

### 1. 멀어지는 경우에 왜 스티커앱이 동작하지 않는지 분석한다. 

- detection, landmark, blending 단계 중 dlib detection이 문제이다. 
- 거리가 멀어지면 detector_hog 단계에서 bbox 가 출력되지 않는다. 


### 2. detector_hog의 문제를 해결해 본다. 

- dlib_rects = detector_hog(img_rgb_vga, 1) # ← ***이미지 피라미드 수를 변경***하여 성능을 향상시킨다. 
- 이 방법을 활용하여 img2sticker 메소드를 수정했다. 
- 수정 후에 webcam_sticker.py 를 다시 한번 실행하여 스티커앱이 잘 실행되는지 확인해봤다. 

### 3. 위에서 새롭게 시도한 방법의 문제점은 무엇인가?

- 속도가 현저히 느려집니다. 기존 30ms/frame 에서 120ms/frame 으로 약 4배 느려짐 → 실시간 구동이 불가능하다. 

### 4. 실행시간을 만족할 수 있는 방법을 찾아본다.

- hog 디텍터를 딥러닝 기반 디텍터로 변경할 수 있다. 
- hog 학습 단계에서 다양한 각도에 대한 hog 특징을 모두 추출해서 일반화 하기 어렵기 때문에 딥러닝 기반 검출기의 성능이 훨씬 좋다. 
- opencv 는 intel cpu 을 사용할 때 dnn 모듈이 가속화를 지원하고 있으므로, mobilenet과 같은 작은 backbone 모델을 사용하고 ssd를 사용한다면 충분히 만족할 만한 시간과 성능을 얻을 수 있다. 

### 5. 인원 수, 각도 등 각 문제에 대해서 1 - 4번을 반복해본다. 

# ★ 프로젝트 (5) 칼만 필터 적용하기

### 1. 카메라 앞에서 가만히 있을 때 스티커의 움직임을 관찰해 본다. 어떤 문제가 발생하나요?

- 가만히 있어도 스티커의 크리가 일정하게 유지되지 않고, 떨리는 것처럼 보이는 현상이 발생한다. 

### 2. 이론 강의에서 배운 칼만 필터를 적용해서 스티커 움직임을 안정화시켜 주세요.

- 칼만 필터를 구현한 모듈인 kalman.py와 이를 이용하여 tracker를 구현한 kpkf.py를 이용하여 
- 칼만필터를 적용한 webcam_sticker_kf.py를 함께 첨부한다. 
- 실행해 보면 현재는 웹캠이 아니라 샘플 동영상에 칼만필터가 적용된 것을 확인할 수 있다.
- 동영상 입력을 웹캠으로 바꾸면 우리가 만들고 있는 웹캠에도 동일하게 적용할 수 있다. 
- webcam_sticker_kf.py를 참고하여 자신만의 webcam_sticker.py를 완성해 본다. 

### 3. 칼만 필터를 적용한 최종 결과물 : ./images/result.mp4

- 결과물을 화면 캡쳐한 이미지 첨부

![result](https://user-images.githubusercontent.com/39249809/101380427-d31efb80-38f8-11eb-8430-9048899b7158.png)