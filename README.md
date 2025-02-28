
### “아니, 쏘카존에 반납하려니까 만석이라 다른 곳에 주차했더니 패널티라네요.”

![뉴스](https://user-images.githubusercontent.com/19771164/148502420-82ad41be-a154-4dd5-9c88-707636e15700.png)(http://www.newsworker.co.kr/news/articleView.html?idxno=113012)

- 쏘카 반납 단계에서 예상치 못하게 주차공간이 없을 때 불편함을 겪는 사용자들이 존재
- → **“우리가 주차장의 상황을 알려주면 되지 않을까?”**

### 주차장 상황 및 주차 가능 여부 정보 제공
                 




## 주차장 '학습'에는 어떤 요소들이 중요할까?

- ### **실내 주차장 / 실외 주차장**이 중요할 것이라 판단  

     

- ### 주차장의 종류와 관계없이 **카메라 각도**가 중요할 것이라 판단

    

- ### 주차장 자체가 중요할 것이라 판단
<br>


## 어떤 모델을 사용할까?  

### [Yolo v5](https://github.com/ultralytics/yolov5)

- 현재 Object detection에서 뛰어난 성능을 보이고 있는 Yolo v5를 pretrained model로 활용  

- 각각 중요하다고 생각하는 요소에 맞춰 detect하도록 fine tuning
<p align="left">
  <img width="500" height="300" src="https://user-images.githubusercontent.com/19771164/148505749-88dab2d1-7871-4ce5-b999-add485a4cbdb.png" alt="Sublime's custom image"/>
</p>

## 어떤 데이터를 사용할까?

- 실내 데이터의 경우 블랙박스보다는 cctv 이미지가 더욱더 의미가 있을 것이라 판단
- 문제는! 보안상 문제로 직접적으로 전달받을 수 있는 데이터셋이 없음  
-> 팀원 모두가 라벨링을 수작업으로 진행하기로 결정
- Dataset
    - **[PKLot Dataset](https://public.roboflow.com/object-detection/pklot)**
    - **[CNR Park](http://cnrpark.it/)**
    - **[Microsoft COCO 2017 Dataset](https://public.roboflow.com/object-detection/microsoft-coco-subset)**
  
---  
<br><br>
## 1차 시도(실외 vs 실내)

### 실내 주차장

| ![실내주차장.jpg](https://user-images.githubusercontent.com/19771164/148506256-470dfb1b-0229-4fe1-a433-0a4a8750f53c.jpg) | 
|:--:| 
|__실내주차장 예시.jpg__|
   
- 공개되어있는 실내 주차장 데이터셋을 찾을 수 없음
- 모델을 돌릴 수 있을 정도의 양질의 데이터셋을 찾을 수 없음
- 개인정보 문제 등 여러 보안문제가 발생할 수 있음
- → 그래서 공개되어있는 실외 주차장 데이터셋을 최대한 활용하기로 결정

### 실외 주차장

- [PKLot 데이터](https://public.roboflow.com/object-detection/pklot)
  - train데이터(8,691장)  
valid 데이터(2,483장)  
test 데이터(1,243장) 
    
    - Federal University of Parana에서 공개한 오픈 데이터로 자유로운 활용이 가능
    - Yolo에서 필요한 labels의 자료가 있어 잘할 수 있을 것이라 판단
 
 |<img width="800px" height="400px" src="https://user-images.githubusercontent.com/19771164/148506281-70aab702-af54-4465-b46f-bdd7454c616a.png" alt="Sublime's custom image"/>|
 |:--:|
 |*PKLot 데이터 예시*|  
 
- 학습 결과 (epochs 20 / batches 32)

 |<img width="400px" height="400px" src="https://user-images.githubusercontent.com/19771164/148506719-c5d9d3f3-4175-47cc-b496-d6f5c8a52573.jpg" alt="Sublime's custom image"/>| 
 |:--:|
 |*학습 결과: 1='car' class*|

    
### 1차 시도의 결론 및 피드백

1. 실내 데이터는 개인정보의 문제로 인해 부족하기 때문에 실외 데이터로 진행
2. 라벨링을 일관되게 설정하는 것이 중요함을 알게 됨
3. PKLot 데이터가 train이 8,000장이지만 4,000장씩 2개의 카메라만 존재해서 생각보다 빠른 시간(5epoch에서)내에 overfitting에 도달  
  따라서 다른 데이터에 적용되기 어려움
  
|![PKlot이미지.jpg](https://user-images.githubusercontent.com/19771164/148506977-39bdaf18-35e7-463a-b6ff-c59d9abb62fd.png)| 
|:--:|
|*1차 학습 모델를 이용한 다른 주차장 이미지에서의 inference 
위: PKLot 학습과 비슷한 주차장 / 아래: 사진은 상이한 주차장|

4. 카메라 각도가 비슷한 사진들은 80%정도의 정확성으로 detect 하는 것을 보니 **카메라 각도**가 중요한 feature이지 않을까?  

---  
<br><br>

## 2차 시도(카메라 각도에 따른 학습)

### 데이터셋

- CNR park
    - 총 9개의 카메라로 한 주차장의 이미지를 수집한 데이터
    - 각 카메라는 다른 각도를 가지고 있기 때문에, 각 카메라 별로 따로 학습을 시켜주어 테스트 진행
    - 아래 이미지: CNR Park 데이터 예시

|![cnr8_ex](https://user-images.githubusercontent.com/54520828/148553485-7451cb47-de90-4b13-9472-f42e430382df.jpg)|![cnr2_ex](https://user-images.githubusercontent.com/54520828/148553494-c4c2ed6e-b77b-4669-bbd8-7a769e35b206.jpg)| 
|:--:|:--:|
|8번 카메라에서 찍은 주차장|2번 카메라에서 찍은 주차장|

        
- 데이터 라벨링
    - 1차 시도에서 사용한 PKLot 데이터 라벨링 방식을 바탕으로 일관된 라벨링을 진행
    - [LabelImg](https://github.com/tzutalin/labelImg)을 (PASCAL VOC format, Yolov5 format 등을 지원)을 이용

    |![cnr8_ex](https://user-images.githubusercontent.com/19771164/148529084-24b532e1-e4ee-4b04-9174-770ff7f17eaf.jpg)|
    |:--:|
    |이미지 라벨링 LabelImg 프로그램|
- 학습
     - 2번 카메라 
      
            
- 학습 및 결과
     - 2번 카메라 데이터
          - 2015/11/13 - 2015/11/29 train데이터(260장)  
2015/12/01 - 2015/12/05 valid데이터(43장)  
2016/01/14 - 2016/01/16 test데이터(43장)
     - 2번 카메라 결과 ( epochs 50 / batch 30 )

|![cam2_result](https://user-images.githubusercontent.com/54520828/148554652-698f60f0-1e7a-4878-bb3c-44cff4ad46e5.png)|
|:--:|
|왼쪽: 모델 적용 전 & 오른쪽: 모델 적용 후|

   - 8번 카메라 데이터
      - 2015/11/13 - 2015/11/29 train데이터(371장)  
        2015/12/01 - 2015/12/05 valid데이터(38장)  
        2016/01/14 - 2016/01/16 test데이터(40장)
     - 8번 카메라 결과 ( epochs 50 / batch 30 )

|![cam8_result](https://user-images.githubusercontent.com/54520828/148556087-15f77be8-38a8-4a5c-958c-a5e256d38fea.png)|
|:--:|
|왼쪽: 모델 적용 전 & 오른쪽: 모델 적용 후|

   - 9번 카메라 데이터
      - 2015/11/13 - 2015/11/29 train데이터(407장)  
        2015/12/01 - 2015/12/05 valid데이터(43장)  
        2016/01/14 - 2016/01/16 test데이터(40장)
     - 9번 카메라 결과 ( epochs 50 / batch 30 )

|![cam9_result](https://user-images.githubusercontent.com/54520828/148556729-7f54fe7d-6a26-4854-8467-ea37bf63d460.png)|
|:--:|
|왼쪽: 모델 적용 전 & 오른쪽: 모델 적용 후|


        
### 2차 시도의 결론 및 피드백

1. 한 주차장을 잘 학습한다면 해당 주차장의 다른 시간대에서 충분히 좋은 결과를 낼 수 있음
2. 비슷한 각도라고 판단되는 8번 카메라 모델로 다른 주차장에서 아쉬운 결과를 보이는데, 차와 빈 자리를 논리적으로 찾는 것이 아니라 라벨링 되어있는 그 부분의 **지형지물**까지 학습하는 것으로 보임. (=차가 주차되어있는 각도와 각 쏘카존 고유의 상황이 중요할 수도 있음)
3. 그렇다면 데이터가 많이 부족한 상황에서, 하나의 모델로 많은 쏘카존을 커버하기 위해 사용할 수 있는 **어떤 기법**은 무엇이 있을까?
    
|![8번카메라_testinference](https://user-images.githubusercontent.com/54520828/148635624-e902c5d1-1ac4-4697-b7c0-4a5d677507a0.png)|
|:--:|
|8번 카메라에서 나온 데이터를 학습한 모델로 다른 주차장 예시에서의 inference|
    
        
 --- 
 <br>
 
## 3차 시도(데이터 최적화)

데이터가 부족한 상황, 우린 어떻게 대처할 수 있을까?

→ 주차장 고유의 상황이 중요하다면, 서로 다른 주차장에서의 데이터는 서로 커버할 수 없을까?

- 데이터가 부족하니 data augmentation을 진행해보자
    - Yolov5의 데이터 증강 기법 중 default 값에 적용되지 않는 방법들로 학습 진행(→image rotation, shear, mixup 등)
    - image rotation, shear, mixup의 기법을 사용한 결과, **같은 데이터셋**에서는 좋은 성능을 나타냄
 
- 학습 데이터셋: CNR Park images from camera 9
    - train: 데이터 수집의 첫 7일(Day1: 2015-11-21, Day7: 2015-12-3, 하루 평균 20장 업로드)
    - valid: 데이터 수집의 8, 9일 째

|![3차_cnr_예시1_norotation](https://user-images.githubusercontent.com/54520828/148557420-7bbd6362-7af8-406a-a14e-f0222a575a9f.jpg)|![3차_cnr_예시1_rotation](https://user-images.githubusercontent.com/54520828/148557429-4d5a13dc-b096-4e64-91ea-ad7fc9f698e8.jpg)|
|:--:|:--:|
|예시1: rotation, shear, mixup 적용하지 않은 경우|예시1: rotation, shear, mixup 적용한 경우|
|![3차_cnr_예시2_norotation](https://user-images.githubusercontent.com/54520828/148557434-85e4e200-a51e-4009-82da-01e8b0c1c805.jpg)|![3차_cnr_예시2_rotation](https://user-images.githubusercontent.com/54520828/148557440-2ee1a441-fa16-44ec-a902-4b0d025adb85.jpg)|
|예시2: rotation, shear, mixup 적용하지 않은 경우|예시2: rotation, shear, mixup 적용한 경우|


### 3차 시도 결과 및 피드백

- 데이터가 많지 않은 상황에서 image augmentation은 성능을 향상시킬 수 있는 중요한 기법으로 사용될 수 있음
- 하지만 어떤 image augmentation 기법이, 얼마나 사용되어야 하는지는 각 데이터셋마다 다름
- 따라서, 하나의 모델을 사용하는 것은 많은 종류의 쏘카존을 보완해야하는 ‘확장성’의 문제에 부딪히게 됨

---
<br>

## 4차 시도(확장성)

### 앙상블

- 주차장을 학습하는 것이 가장 좋은 방법이지만 많은 자원(시간, 인력)이 필요로 하기 때문에, 이를 해결하고 서비스 형태가 되기 위해서 바로 투입이 가능한 형태의 모델이 필요하다고 판단
- 주차장의 지형지물이 아닌 car를 학습한 모델을 학습
- COCO 데이터 12,000장 학습한 모델
    - **[Microsoft COCO 2017 Dataset](https://public.roboflow.com/object-detection/microsoft-coco-subset)**
        
        COCO Dataset(121,408장)에서 차가 존재하는 이미지만을 사용해서 12,000장의 이미지로 학습을 진행

- COCO dataset + 데이터 증강기법이 적용된 COCO dataset
    - 결과


|![4차예시1_](https://user-images.githubusercontent.com/54520828/148635383-fc047ab4-7dc5-44d8-a505-7a749e97fb14.jpeg)|![4차예시1_앙상블](https://user-images.githubusercontent.com/54520828/148635385-a73ad9bc-5f0c-4f9e-883e-31f74207ea07.jpeg)|
|:--:|:--:|
|예시1: Yolov5 적용|예시1: 앙상블 모델 적용|
|![4차예시2_](https://user-images.githubusercontent.com/54520828/148635386-31d81aef-4258-4e76-b3f3-6e81a4a755e1.jpeg)|![4차예시2_앙상블](https://user-images.githubusercontent.com/54520828/148635387-f0f13539-f188-49eb-876b-5d3416028270.jpeg)|
|예시2: Yolov5 적용|예시2: 앙상블 모델 적용|
|![4차예시3_](https://user-images.githubusercontent.com/54520828/148635389-d466cecc-586e-49fb-8e8b-bde0a3688538.jpeg)|![4차예시3_앙상블](https://user-images.githubusercontent.com/54520828/148635392-aaba7fea-22f7-4789-b01a-053c8211a507.jpeg)|
|예시3: Yolov5 적용|예시3: 앙상블 모델 적용|
|![4차예시4_](https://user-images.githubusercontent.com/54520828/148635393-d8928b3f-a923-4bc7-90bd-d174318f6e31.png)|![4차예시4_앙상블](https://user-images.githubusercontent.com/54520828/148635394-fc465d55-a812-4468-87c2-2d99866a0309.png)|
|예시4: Yolov5 적용|예시4: 앙상블 모델 적용|


### 4차 시도 결과 및 피드백

- ‘car’만 12,000장 학습한 모델과 해당 데이터를 augmentation한 모델을 앙상블했을 때 결과가 상당히 발전
- 차의 개수가 확인이 되면 빈 자리의 여부도 알려줄 수 있음
     - 빈자리 수 = 전체 자리 수 - 차의 갯수
<br>

## 확장성을 위한 추가 기능들

### Person Privacy Filter(개인정보보호)
- '개인 정보'는 현대 가장 중요한 이슈임
- 사진에서 사람을 분류하고 그 사람을 blur 처리해서 개인 정보 문제에 대응
- 학습
     - Microsoft COCO 2017 Dataset 데이터 10,000장의 person 데이터를 학습
- 결과

|![car_detect.png](https://user-images.githubusercontent.com/19771164/148554241-a2c9ea50-f59a-4afc-b941-de3be005979e.gif)|
|:--:|
|Person Privacy Filter 적용: 데이터는 동반신기의 미로틱 무대|
<br>

### Time(정확성)
- 주차장의 사진이 어느 시점에 찍혔는지 알아야, 사용자가 정확한 주차 가능 여부를 파악할 수 있음
- 사진 위에 detect 시점의 시간 정보를 추가

|![car_detect.png](https://user-images.githubusercontent.com/19771164/148534298-58374ffa-2dda-4a22-8bd0-b949d9e7e55a.jpg)|
|:--:|
|이미지 좌측 상단에서 시간과 남아있는 자리를 알 수 있음|


---
<br><br>

## 5차 시도(OpenCV)
- 4차 결과 이후 딥러닝을 통한 주차장 detection이 결국 200장이 넘는 데이터와 라벨링의 필요함을 알게 되고, labeling등의 막대한 인권비가 듦
- 보통 실내 주차장의 경우 자리 유무에 대한 '센서'가 존재
-> 적은 비용으로 기존 주차장 카메라를 활용해서 센서를 대체할 수 없을까? <br>
- Computer vision을 쉽게 할 수 있도록 도와주는 OpenCV를 활용해서 빈자리를 찾자


|![opencv1](https://user-images.githubusercontent.com/19771164/150927523-b3214df8-e45e-4926-8efb-195da34697c0.png)|![ImageMedian](https://user-images.githubusercontent.com/19771164/150927635-1aa0db9c-fbad-416b-9613-82ff290b808a.png)|
|:--:|:--:|
|주차장 labeling|image에 Medianblur|
|![ImageDilate](https://user-images.githubusercontent.com/19771164/150927640-dd69a5c9-6ff5-4dff-947e-095c6e5baa40.png)|![Image](https://user-images.githubusercontent.com/19771164/150927616-56d75f26-5b28-4b1f-bd0d-8b998dbbf0cf.png)|
|image Dilate|결과물|

1. 이미지에서 주차장 구역을 cv2.polylines를 활용해서 나눈 다각형의 좌표를 저장
2. 이미지를 Grayscale로 전환한 후 Gaussianblur를 통해 이미지 윤곽선을 추출
3. Medianblur를 이용해 윤곽선을 선명하게 설정
4. Dilate를 통해 윤곽선 픽셀들을 3x3커널을 통해 확장
5. 물체들이 black&white로 구분되고 black픽셀과 white pixel의 비율을 통해 빈자리를 추출 

### 5차 시도 결과 및 피드백

- 부족한 데이터 속에서 한 번의 라벨링과 OpenCV image 변환들을 통해서 주차장을 detection할 수 있음
- 하지만 빛이 강하게 비친 부분이나 반사된 부분에서의 경계면이 생겨 해당 부분의 윤곽선의 한계가 존재
- 실내 주차장이나 특이한 지형 지물에 대한 라벨링의 고민이 필요하고, 상황에 맞는 임계값을 주는 것이 중요할 것이라 판단
- 딥러닝을 통한 주차장의 많은 자원이 드는 것을 OpenCV를 통해 어느 정도 해결해줄 수 있을 것이라 

---
<br><br>
## 결과물

### 웹사이트 구조
- 실질적인 사용자 경험 개선을 위해서 우리가 찾은 주차 공간을 알리고자 웹사이트를 구현

|<img width="400px" height="400px" src="https://user-images.githubusercontent.com/19771164/148534847-c80c57de-382f-484c-b3cb-40e6443043b9.png" alt="Sublime's custom image"/>|
|:--:|
|TEAM DI 웹서비스 사이트 구조|
    
   
        
- 데이터 베이스 종류
    - SOCAR_ZONE
    : 쏘카존 이름, 쏘카존 위도, 쏘카존 경도, 주소, 현재 주차장 빈자리 수, 주차장 만차 수
    - Client
    : 이메일, 비밀번호, 이름
    - File
    : 쏘카존 이름, 주차장 사진
    
### 최종 결과물

[teamdi.xyz](http://teamdi.xyz/) <- 클릭

- 공식 id / 공식 비밀번호  
abcd@naver.com / 123

|![car_detect.png](https://user-images.githubusercontent.com/19771164/148516059-91da5206-12af-45cb-9737-20854d1df2bb.gif)|
|:--:|
|Client 사용 영상|
|![car_detect.png](https://user-images.githubusercontent.com/19771164/148644010-813db6fc-26c4-48c8-b538-ba3c87e31564.gif)|
|TEAM DI 결과물 시연 영상|


---
<br><br>
### 코드 예시
[Yolov5](https://github.com/ultralytics/yolov5)를 기반으로 만들었으며 목적에 맞게 수정하였습니다.

```python

z_name = '한성대입구역 5번출구'
source_path = '/content/drive/MyDrive/socar_zones/'+z_name+'/'
!python detect_car.py --source '{source_path}' --zone_name '{z_name}' --conf 0.4 --weights /content/drive/MyDrive/models/COCO_car_20ep.pt /content/drive/MyDrive/models/person_detect.pt

!python img_to_db.py '{z_name}'
```

- Colab : 코랩에서 사용한 대표 파일
- yolov5 : Customize한 yolov5
- templates : html & css
- static : 사진 및 자료

- models  
       - 2camera_260train.pt : 2번 카메라 모델  
       - 8camera_350train.pt : 8번 카메라 모델  
       - 9camera_350train.pt : 9번 카메라 모델  
       - person_detect.pt : COCO 10,000장 사람 모델  
       - car_detect_20ep.pt : COCO 12,000장 차 모델  
       - car_detect_aug_20ep.pt : augmentation COCO 12,000장 차 모델  
       - car_detect_aug2_30ep.pt : augmentation2 COCO 12,000장 차 모델  
- blur  
       - utils.plots.py
- put DB  
       - img_to_db.py
- web  
       - index.html : 메인 페이지  
       - map.html : 지도 페이지  
       - app.py : 서버  
       - clients : clients DB  
       - socar_zone : socar_zone DB  
       - DB : Mongo 
--------
### 참조
- [Yolov5 github 링크](https://github.com/ultralytics/yolov5)
- [Data Augmentation hyperparameters 최적화](https://github.com/ultralytics/yolov5/issues/607)
- [CNR Park 데이터셋](http://cnrpark.it/)
----------
