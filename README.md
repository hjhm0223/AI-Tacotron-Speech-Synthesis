# readme
Tacotron 모델을 이용하여 게임 속 음성합성 기술 적용

## 개발환경
- ubuntu
- docker
- python 3.6 or later
- tensorflow 1.3

## Tacotron 코드 사용법
 
### 0. 환경 설정
우분투 터미널에서 [도커이미지](https://hub.docker.com/r/hyunjee/hyunjee01/) 다운로드 후 도커 이미지 실행 (GUI 권한 주기)

```
$ docker pull hyunjee/hyunjee01
$ nvidia-docker run -it --net=host -e DISPLAY -v /tmp/.X11-unix -p 8888:8888 -v /workspace:/workspace f46a0c4dc7b5 /bin/bash
```
 
### 1.코드 실행하기
[Tacotron 소스](https://github.com/carpedm20/multi-speaker-tacotron-tensorflow) 다운로드
```
$ git clone https://github.com/carpedm20/multi-speaker-tacotron-tensorflow.git
```

1.1 tensorflow 1.3 설치
 ``` 
 $ pip uninstall tensorflow
 $ pip install tensorflow==1.3
 ```
1.2 `FFmpeg` 라이브러리 설치
 ```
 $ pip install ffmpeg
 ```
1.3 Python 3.6 이상 버전 설치
 ```
 $ python --version
 ```
1.4 필수 라이브러리 설치
 ```
 pip3 install -r requirements.txt
 python -c "import nltk; nltk.download('punkt')"
 ```
1.5 `librosa`의 경우 버전 에러가 발생힐 수 있으므로 먼저 다음 명령어를 통해 설치
 ```
 $ pip install librosa==0.5.1
 ```
1-6. 음성<->텍스트 쌍 정보가 저장되어 있는 `alignment.json` 파일과 audio폴더를 `./datasets/yuinna/` 경로에 저장. (zip 파일에 첨부, 용량문제때문에 일부의 음성파일만 존재)
1-7. 마지막으로 학습에 사용될 `numpy` 파일 생성 (모델 트레이닝에 사용)
 ```
 pip install jamo
 pip install unidecode
 pip install inflect
 python3 -m datasets.generate_data ./datasets/yuinna/alignment.json
 ```

> git에서 파일을 받은 경우 unicode error 발생 시 `./datasets/generate_data.py` 파일에서 `with open` 부분에 인수 ` ,’r+’, encoding =’utf-8’ ` 추가


### 2. 모델 학습
 
모델의 중요 파라미터는 `hparams.py` 에서 확인

2-1. 유인나 음성의 경우, single speaker model이기 때문에 single speaker로 트레이닝
 ```
 $ pip install tinytag
 $ python3 train.py --data_path=datasets/yuinna
 ```
2-2. 만약 학습 중이던 모델을 다시 학습하려면(ex. Yuinna_2018-07-16_05-07-38, 실제 생성되는 날짜로 변경)
 ```
 $ python3 train.py --data_path=datasets/yuinna --load_path logs/yuinna_2018-07-16_05-07-38
 ```
2-3. 학습을 하면 log 폴더에 새로운 폴더가 생기며 해당 폴더 안에 checkpoint 파일과 학습과 관련된 checkpoint 파일이 생성


### 3. 음성 만들기

만약 현재 100만번까지 학습이 되었는데 80만번 때의 checkpoint로 음성을 만들고 싶을 때는 logs 폴더 안에 있는 ex.yuinna_2018-07-16_05-07-38 폴더 안에 있는 checkpoint 파일에서 노란 색으로 밑줄친 값들을 변경
(프로젝트를 수행하면서 생성된 checkpoint를  첨부했으며 checkpoint라는 폴더안에 ‘yuinna_2018-07-16_05-07-38’ 폴더를 ‘multi-speaker-tacotron-tensorflow/logs’에 넣는다)

```
model_checkpoint_path: "model.ckpt-800000"
all_model_checkpoint_paths`: "model.ckpt-800000"
```

3-1. 웹 데모로 음성 만들기

저장된 wav 파일은 `/multi-speaker-tacotron-tensorflow/web/audio/` 에서 확인
> 만약 직접만들어낸 chkpt를 사용하고 싶다면 logs뒤에 디렉토리를 변경

```
$ python3 app.py --load_path logs/yuinna_2018-07-16_05-07-38 --num_speakers=1
```


3-2. 아래의 명령어를 통해 음성 파일 만들기

`synthesizer.py` 파일을 열어서 맨 아래 374번째 줄에 있는 default=” “ 여기서 콜론 안에 만들고자하는 텍스트를 입력. 이후 synthesizer.py를 실행하여 음성파일 생성 
```
$ python3 synthesizer.py --load_path logs/yuinna_2018-07-16_05-07-38
```
만들어진 음성 파일은 `/multi-speaker-tacotron-tensorflow/samples/`에서 확인
