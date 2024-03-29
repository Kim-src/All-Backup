---
title: 싱크홀(공동) 자동 분석 AI 모델 VisualCavity AI 개발 요약 내용(ver. 0.1.0)
date: 2024-02-24 21:00:00 +09:00
categories: [VisualCavity, AI Model]
tags: [Python, TensorFlow, AI, AI Model, Numpy]
---

<!-- 2024-02-19 글 작성 완료; 2024-00-00 페이지 호출 검토 필요 -->
# ✅ 싱크홀(공동) 자동 분석 AI 모델 VisualCavity AI 개발(ver. 0.1.0)

<br>

### 🔔 1. Introduction
### 📌 모델 소개
> - 모델명 : VisualCavity (ver. 0.1.0)
> - 모델분류 : 이미지 학습 CNN AI 모델
> - 상세내용 : 싱크홀(공동) 자동 분석 프로그램 제작을 위한 이미지 학습 AI 모델
> - 개발목적 : 3D-GPR 데이터 분석 난이도 하향
> - 주요자료 : GPR 공동 탐사 데이터 및 30만장의 이미지

<br>

### 🔔 2. Methodology
### 📌 개발 환경
> - Google Colab에서 Python 코드를 작성하였습니다.
> - 주로 사용된 Python 라이브러리는 TensorFlow, OpenCV 입니다.

### 📌 이미지 학습을 위한 코딩 순서
>   1. Python 내부 라이브러리 사용
>   2. 학습시킬 이미지 1장에 대한 JPG 및 XML 파일의 경로 설정
>   3. 이미지 전처리를 위한 함수 정의
>   4. 이미지 로드 및 전처리를 위한 함수 작성
>   5. XML 파일에서 이미지 정보 습득을 위한 함수 정의
>   6. XML 파일을 파싱 후 데이터 추출
>   7. CNN 모델 구성(CNN : 컨볼루션 신경망 : Convolution Neural Network)
>   8. 모델 컴파일(Compile : 사람이 작성한 코드를 컴퓨터에게 이해시키는 과정)
>   9. 모델에 입력할 이미지 및 레이블 데이터 세팅
>   10. 이미지 데이터를 모델에 맞게 형태 변환
>   11. 모델 훈련(epochs 활용)

<br>

### 🔔 3. Results : 전체 코드
### 📌 이미지 학습에 사용된 코드 및 간략한 설명
> - 이미지 학습에 사용된 전체적인 코드는 아래와 같습니다.
> - 각 코드마다 간략한 설명을 작성하였습니다.
> - 각 코드와 관련된 상세한 설명은 다음 글에 작성하였습니다.

### 📌 AI 모델의 이미지 학습을 위한 Python 코드
```python
# Python 내부 라이브러리 사용
import xml.etree.ElementTree as ET  # XML 파일을 파싱하기 위한 라이브러리를 사용하였습니다.
import cv2  # 이미지 처리 및 컴퓨터 비전 작업을 위한 OpenCV 라이브러리를 사용하였습니다.
import numpy as np  # 이미지 배열을 위한 라이브러리를 사용하였습니다.
import tensorflow as tf  # 딥러닝 모델 구축을 위한 TensorFlow 라이브러리를 사용하였습니다.
from tensorflow.keras import layers, models  # 모델의 레이어를 적용하기 위해 TensorFlow의 Keras API를 사용하였습니다.
from tensorflow.keras.preprocessing import image  # 이미지 로드 및 전처리를 위한 함수를 사용하였습니다.
from tensorflow.keras.preprocessing.image import img_to_array  # 이미지를 넘파이 배열로 변환하는 함수를 사용하였습니다.

# 학습시킬 이미지 1장에 대한 JPG 및 XML 파일의 경로 설정
image_path = "/content/drive/MyDrive/MyPJT/Cavity_PJT/cavity_sample_img.jpg" # JPG 파일 경로입니다.
xml_path = "/content/drive/MyDrive/MyPJT/Cavity_PJT/cavity_sample_xml.xml" # XML 파일 경로입니다.

# 이미지 전처리를 위한 함수 정의
def preprocess_image(image_path, target_size=(256, 256)):
    img = image.load_img(image_path, target_size=target_size)  # 이미지를 로드하고 target_size로 크기를 조정하였습니다.
    img_array = img_to_array(img)  # 이미지 배열을 Numpy에 맞게 변환하였습니다.
    img_array = np.expand_dims(img_array, axis=0)  # 이미지 배열 관련 배치 차원을 추가하였습니다.
    img_array /= 255.0  # 이미지를 0과 1 사이의 값으로 정규화(중복 제거)하였습니다.
    return img_array  # 이미지 전처리가 완료되면, 전처리된 이미지의 배열을 반환하도록 함수를 정의하였습니다.

# 이미지 로드 및 전처리를 위한 함수 작성
img_array = preprocess_image(image_path) # 상기 정의한 함수에 대한 내용을 작성하였습니다.

# XML 파일에서 이미지 정보 파싱을 위한 함수 정의(파싱 : 필요한 정보를 추출하는 것)
def parse_annotation(xml_path):
    tree = ET.parse(xml_path)  # ElementTree로 XML 파일을 파싱하였습니다.
    root = tree.getroot()  # 파싱된 XML 파일에 루트 엘리먼트를 적용하였습니다.

    filename = root.find('filename').text  # 이미지의 데이터의 이름을 추출하는 함수를 작성하였습니다.
    object_name = root.find('.//object/name').text  # 객체의 이름을 추출하는 함수를 작성하였습니다.

    # 바운딩 박스 좌표 추출(ROI : Region of Interst)
    xmin = float(root.find('.//bndbox/xmin').text)
    ymin = float(root.find('.//bndbox/ymin').text)
    xmax = float(root.find('.//bndbox/xmax').text)
    ymax = float(root.find('.//bndbox/ymax').text)

    return filename, object_name, xmin, ymin, xmax, ymax # 이미지 파싱이 완료되면 관련 정보를 추출하도록 함수를 정의하였습니다.

# XML 파일을 파싱 후 데이터 추출
filename, object_name, xmin, ymin, xmax, ymax = parse_annotation(xml_path)

# CNN 모델 구성(CNN : 컨볼루션 신경망 : Convolution Neural Network)
model = models.Sequential()
model.add(layers.Conv2D(32, (3, 3), activation='relu', input_shape=(256, 256, 3)))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Conv2D(64, (3, 3), activation='relu'))
model.add(layers.MaxPooling2D((2, 2)))
model.add(layers.Conv2D(64, (3, 3), activation='relu'))
model.add(layers.Flatten())  # AI 모델에 평탄화(Flatten) 레이어를 추가하기 위해 3D 특성 맵을 1D 텐서로 평탄화하였습니다.
model.add(layers.Dense(64, activation='relu'))
model.add(layers.Dense(1, activation='sigmoid'))  # 이진 분류를 위한 출력 레이어를 추가하였습니다.

# 모델 컴파일(Compile : 사람이 작성한 코드를 컴퓨터에게 이해시키는 과정)
model.compile(optimizer='adam',
              loss='binary_crossentropy',  # 이진 분류를 위한 손실 함수를 사용하였습니다.
              metrics=['accuracy'])  # 정확도(accuracy)를 평가 지표(metrics)로 사용하였습니다.

# 모델에 입력할 이미지 및 레이블 데이터 세팅
X_train = np.array([img_array])  # 이미지 배열을 넘파이 배열로 변환하게 하였습니다.
y_train = np.array([1])  # 이진 분류의 경우 레이블을 넘파이 배열로 변환하게 하였습니다.

# 이미지 데이터를 모델에 맞게 형태 변환
X_train = np.squeeze(X_train, axis=0)

# 모델 훈련(Epochs 활용)
model.fit(X_train, y_train, epochs=5, batch_size=1)
```

<br>

### 🔔 4. Conclustions
### 📌 AI 모델의 이미지 학습 현황 판단 방법
> - Python을 이용한 CNN AI 모델의 이미지 학습이 완료되었으며 이 이미지 학습은 머신 러닝의 일환입니다.
> - 머신 러닝 중 AI 이미지 학습 결과는 Epoch로 판단할 수 있습니다.
> - 머신 러닝에서의 Epoch=1은 신경망(Neural Network)이 전체 데이터 세트를 한 번 통과하는 과정을 의미합니다.
> - 본 AI 모델의 Epochs는 5로 설정했기 때문에 신경망이 전체 데이터 세트를 다섯 번 통과하였습니다.
> - 아래는 5번의 Epoch로 판단할 수 있는 AI 모델의 이미지 학습 현황입니다.

### 📌 Epoch로 알아보는 AI 모델의 이미지 학습 현황
> - Epoch 1/5 : AI 모델이 학습용 데이터를 2초 동안 한 번 통과하였고 손실 정도는 0.6287이며 정확도는 100% (= 1)입니다.
> - Epoch 2/5 : AI 모델의 학습 정확도가 100% 유지되었고 손실 정도는 0에 가깝습니다.
> - Epoch 3/5 ~ 5/5 : AI 모델의 학습 정확도가 100%이고 손실 정도는 0입니다.
> - Epoch 값 자체로만 봤을 때는 AI 모델에 이미지가 완벽히 학습되었다고 볼 수 있습니다.

### 📌 AI 모델의 이미지 학습 결과 해석
> - 정량적인 Epoch 결과는 완벽하지만 정성적으로 평가하였을 때 이 모델은 문제가 있습니다.
> - 다양성 부족 : 학습용 이미지 데이터들이 유사하거나 동일한 경우 AI 모델이 쉽게 데이터 패턴을 학습했을 것입니다.
> - 과적합(Overfitting) : AI 모델이 이미지 데이터에 과도하게 fitting 되어 있어서 새로운 이미지에 대한 일반화가 이루어지지 않았을 수 있습니다.
> - AI 모델의 이미지 학습 결과를 보다 정량적으로 검토하기 위해서는 Training 데이터만이 아니라 Validation 데이터 역시 활용되어야 합니다.
> - 부록에 작성되어있는 것처럼 AI 모델의 버전을 업그레이드 한 뒤 Validation 데이터를 이용하여 정량적인 평가를 할 필요성이 있습니다.
> - 다음 버전에 대한 자료는 다음 글에 작성하였으며 아래는 AI 모델의 이미지 학습 결과에 대한 출력 내용입니다.

<img src="https://github.com/Kim-src/Images/assets/150884526/13d1bca2-cc95-4c9f-be4c-32cb624a64f8" class="img" alt="figure">

<br>

### 🔔 5. Reference
### 📌 GitHub Repository 주소
> <a href="https://github.com/Kim-src/VisualCavity">싱크홀(공동) AI 자동 분석 프로그램 깃허브 링크</a>

<br>

### 🔔 6. Appendix
### 🚀 개발 현황 : VisualCavity AI (현재 ver. 0.1.0)
> - v0.1.0 : 이미지 1장 학습 완료 (2024-01-09)
> - v0.2.0 : 이미지 100장 학습 완료
> - v0.3.0 : 이미지 1,000장 학습 완료
> - v0.3.3 : 이미지 1,000장 학습에 대한 검토 완료
> - v0.3.7 : 학습한 이미지 1,000장을 가지고 다른 이미지를 대조한 뒤 버전 비교 완료
> - v0.4.0 : 이미지 10,000장 학습에 대한 검토 완료
> - v0.4.3 : 이미지 10,000장 학습에 대한 검토 완료
> - v0.4.7 : 학습한 이미지 10,000장을 가지고 다른 이미지를 대조한 뒤 버전 비교 완료
> - v0.5.0 : 이미지 100,000장 학습에 대한 검토 완료
> - v0.5.3 : 이미지 100,000장 학습에 대한 검토 완료
> - v0.5.7 : 학습한 이미지 100,000장을 가지고 다른 이미지를 대조한 뒤 버전 비교 완료
> - v0.6.0 : 이미지 300,000장(보유중인 이미지 개수) 학습에 대한 검토 완료
> - v0.6.3 : 이미지 300,000장 학습에 대한 검토 완료
> - v0.6.7 : 학습한 이미지 300,000장을 가지고 다른 이미지를 대조한 뒤 버전 비교 완료
> - v0.7.0 : 이미지가 학습된 AI 모델을 이용하여 50m 길이의 탐사 데이터 분석
> - v0.7.1 : 탐사 데이터 분석 성공률 10% 달성  
>   (수많은 에러 발생 가능성 증가 예상)
> - v1.0.0 : AI를 이용한 싱크홀 분석 모델 생성  
>   (AI 모델을 이용한 싱크홀 분석 프로그램에 적용될 예정)

<br>
<br>
<br>
