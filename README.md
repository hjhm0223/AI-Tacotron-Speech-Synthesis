# readme
IPTV 게임 속 Tacotron 모델을 이용하여 음성합성 기술 적용


<<Tacotron 코드 사용법>>
 
### 0. 환경 설정하기
 https://hub.docker.com/r/hyunjee/hyunjee01/
우분투의 터미널에서 도커이미지를 다운받습니다. 아래 코드를 실행하면 됩니다.

### 1
docker pull hyunjee/hyunjee01
cs
 
0-1.       다운로드 된 도커이미지를 다음 코드로 실행시킵니다.
1
2
3
4
5
nvidia-docker run -it \
    --net=host -e DISPLAY -v /tmp/.X11-unix -p 8888:8888 \
    -v /workspace:/workspace \
    f46a0c4dc7b5 /bin/bash
 
Colored by Color Scripter
cs
 GUI 권한을 주고 도커 이미지를 실행시킵니다.. (이때 노란 부분은 각자의 디렉토리 상황에 맞게 변경합니다.)
 
1.코드 실행하기
 https://github.com/carpedm20/multi-speaker-tacotron-tensorflow
   이곳에서 코드를 받습니다.

1
git clone https://github.com/carpedm20/multi-speaker-tacotron-tensorflow.git
cs
 
  지금부터 clone을 받은 multi-speaker-tacotron-tensorflow/ 위치가 root 디렉토리입니다.
 (git에서 코드를 받아도 되고 첨부파일에 있는 multi-speaker-tactron-tensorflow를 사용해도 무관)
1-2. 기존에 tensorflow가 깔려있다면 

1
2
3
pip uninstall tensorflow
 
pip install tensorflow==1.3
cs
  기존 tensorflow를 삭제하고 tensorflow를 재 설치해 버전을 1.3으로 맞춰줍니다.

FFmpeg도 필수이므로 FFmpeg도 다음 코드를 이용해 설치를 합니다.
1
pip install ffmpeg
cs

 파이썬 버전은 3.6 이상으로 맞추면 됩니다. 아마 3.6.4 일 가능성이 높습니다.
파이썬 버전 확인은 터미널에서 

1
python --version
cs
으로 확인할 수 있습니다.

1-3. 다음 명령어를 통해 필수 라이브러리를 설치합니다.

1
2
pip3 install -r requirements.txt
python -c "import nltk; nltk.download('punkt')"
cs

1-4. librosa의 경우 버전 에러가 생길 수 있으므로 먼저 다음 명령어를 통해 설치합니다.

1
pip install librosa==0.5.1
cs
 
1-5. 음성<->텍스트 쌍 정보가 저장되어 있는 alignment.json 파일과 audio폴더를 
      ./datasets/yuinna/ 경로에 저장을 합니다. (zip 파일에 첨부) 
 < 용량문제때문에 일부의 음성파일만 있습니다. 만약 제대로된 데이터를 넣고 싶다면 수정이 필요함>


1-6. 마지막으로 학습에 사용될 numpy 파일을 만듭니다. numpy 파일로 트레이닝에 사용되기 때문입니다.

1
pip install jamo
pip install unidecode
pip install inflect
python3 -m datasets.generate_data ./datasets/yuinna/alignment.json
cs

만약 git에서 파일을 받은 경우 unicode error가 나옵니다. 이 경우
./datasets/generate_data.py파일에서 


다음과 같이 with open부분을 위와 같이 변경해주어야 합니다.
 “ ,’r+’, encoding =’utf-8’ “ 인수를 open에 추가해야한다.


### 2. 모델 학습하기
 
 모델의 중요 파라미터는
1
hparams.py
cs
에 있습니다. 파라미터를 바꾸고 싶으면 해당 파일에서 변경하면 됩니다.

2-1. 유인나 음성의 경우, single speaker model이기 때문에 single speaker로 트레이닝 시킨다.
 multi-speaker-tacotron-tensorflow 경로 안에서 

1
pip install tinytag

python3 train.py --data_path=datasets/yuinna
cs
로 트레이닝을 시작합니다.

2-2. 만약 학습중이던 모델을 다시 학습하려면(ex. Yuinna_2018-07-16_05-07-38) 
                                                                                             - 실제 생성되는 날짜로 변경필요함.

1
python3 train.py --data_path=datasets/yuinna --load_path logs/yuinna_2018-07-16_05-07-38
cs

으로 다시 학습을 시켜줍니다.
2-3. 학습을 하면 log 폴더에 새로운 폴더가 생기며

해당 폴더 안에 checkpoint 파일과 학습과 관련된 checkpoint 파일이 생성됩니다.

### 3. 음성 만들기

 만약 현재 100만번까지 학습이 되었는데 80만번 때의 checkpoint로 음성을 만들고 싶을 때는
logs 폴더 안에 있는 ex.yuinna_2018-07-16_05-07-38 폴더 안에 있는 checkpoint 파일에서 노란 색으로 밑줄친 값들을 변경하면 됩니다.
<  프로젝트를 수행하면서 생성된 checkpoint를  첨부했으며 checkpoint라는 폴더안에 
   ‘ yuinna_2018-07-16_05-07-38’폴더를 
   ‘multi-speaker-tacotron-tensorflow/logs’에 넣는다 >
1
2
model_checkpoint_path: "model.ckpt-800000"
all_model_checkpoint_paths: "model.ckpt-800000"
cs

 3-1. 웹 데모로 음성 만들기
  만약 직접만들어낸 chkpt를 사용하고 싶다면 logs뒤에 디렉토리를 변경하면 됩니다.
1
python3 app.py --load_path logs/yuinna_2018-07-16_05-07-38 --num_speakers=1
cs
코드를 통해서는 웹 데모를 이용해서 음성을 만들 수 있습니다. (저장된 wav 파일은 

1
/multi-speaker-tacotron-tensorflow/web/audio/
cs
경로에 있습니다.)

 3-2. 아래의 명령어를 통해 음성 파일 만들기

synthesizer.py 파일을 열어서 맨 아래 

부분에서 374번째 줄에 있는 default=” “ 여기서 콜론 안에 만들고자하는 텍스트를 입력해줍니다.

이후 
1
python3 synthesizer.py --load_path logs/yuinna_2018-07-16_05-07-38
cs

로 synthesizer.py를 실행시킨다음 음성파일을 만들어줍니다.

만들어진 음성 파일은

1
/multi-speaker-tacotron-tensorflow/samples/
cs
다음 경로에 저장되어 있으며 음성 파일로 쓰면 됩니다.

