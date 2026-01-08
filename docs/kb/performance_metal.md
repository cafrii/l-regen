
# 이슈

일반 tensorflow 보다, GPU 가속 기능을 활용한 metal 용 tensorflow 가 동작이 더 느림.

사용한 스크립트

https://raw.githubusercontent.com/hunkim/DeepLearningZeroToAll/refs/heads/master/tf2/tf2-06-2-softmax_zoo_classifier.py

csv 로 읽은 데이터를 multinomial classification 학습.
입력 샘플 수: 119
에포크 1000


## 실험 1

GPU 고려 없는 표준 tensorflow 사용

```
rm -rf .venv*
uv venv .venv-std --python 3.11

. .venv-std/bin/activate

uv pip install ipykernel tensorflow
  ...
  + tensorflow==2.20.0

```
이 런타임을 커널로 사용한 jupyter 노트북으로 실행.

fit() 함수 수행에 걸린 시간: 15.8초

## 실험 2

```
deactivate

uv venv .venv-metal --python 3.11

. .venv-metal/bin/activate

uv pip install ipykernel
uv pip install tensorflow==2.16.2 tensorflow-metal==1.2.0
# 이 tf 버전은 metal이 공식 지원되는 최신 버전임.
```

동일한 학습 수행 결과: 47.6초 


# 원인 분석

맥북(Apple Silicon) 환경에서 TensorFlow Metal을 사용할 때 작은 규모의 데이터셋에서 성능이 더 떨어지는 현상은 매우 **전형적인 사례**입니다. 실험하신 결과(15.8초 vs 47.6초)는 오류가 아니라, GPU 가속의 구조적 특성 때문에 발생하는 현상입니다.

그 이유를 4가지 핵심 포인트로 정리해 드립니다.

---

## 1. 데이터 전송 오버헤드 (Communication Overhead)

GPU는 한 번에 엄청난 양의 연산을 처리하는 데 특화되어 있지만, 연산을 시작하기 전 **CPU(메모리)에 있는 데이터를 GPU 메모리로 복사**하는 과정이 필요합니다.

* **실험 상황:** 데이터 샘플이 119개로 매우 적습니다.
* **문제:** 데이터를 GPU로 옮기는 데 걸리는 시간(Overhead)이, GPU가 실제로 연산하는 시간보다 훨씬 길어집니다. CPU는 데이터를 옮길 필요 없이 바로 계산하기 때문에 이 과정이 생략됩니다.

## 2. 커널 런칭 및 스케줄링 지연 (Kernel Launch Latency)

TensorFlow가 GPU에서 연산(행렬 곱셈 등)을 수행하려면 '커널(Kernel)'이라고 불리는 작은 프로그램들을 GPU에 명령어로 내려보내야 합니다.

* **CPU:** 명령을 즉시 실행합니다.
* **GPU:** 명령을 큐(Queue)에 쌓고, 드라이버를 거쳐 GPU 코어에 할당하는 프로세스가 필요합니다.
* 연산량이 적을 때는 이 "준비 동작"이 실제 "운동"보다 더 많은 시간을 잡아먹게 됩니다. 1,000번의 에포크 동안 매번 이 준비 과정을 반복하므로 지연 시간이 누적된 것입니다.

## 3. 낮은 계산 밀도 (Low Computational Intensity)

GPU가 효과를 보려면 **"병렬 처리할 작업이 압도적으로 많아야"** 합니다.

* 사용하신 `softmax_zoo_classifier`는 매우 단순한 선형 모델(Softmax)입니다.
* 입력 데이터와 모델 파라미터가 너무 작아서 GPU의 수천 개 코어 중 아주 일부만 사용되고 나머지는 놀게 됩니다. 마치 100명이 탈 수 있는 대형 버스(GPU)에 승객 1명(데이터)을 태우고 운행하는 것과 같습니다. 이 경우 오토바이(CPU)가 훨씬 빠릅니다.

## 4. 라이브러리 최적화 차이

실험 1에서 사용하신 `tensorflow==2.20.0`은 최신 표준 버전이며 CPU 최적화(AMX 등)가 잘 되어 있습니다. 반면, `tensorflow-metal`은 Apple의 Metal API를 거쳐야 하는 추가 레이어가 존재하며, 특정 버전 조합에 따라 CPU와 GPU 간의 동기화(Sync) 과정에서 병목 현상이 발생하기도 합니다.

---

### 요약 및 결론

| 구분 | CPU (실험 1) | GPU/Metal (실험 2) |
| --- | --- | --- |
| **장점** | 작은 데이터, 단순한 모델에서 즉각 반응 | 대규모 데이터, 깊은 레이어(Deep Learning)에서 압도적 |
| **지연 시간** | 거의 없음 | 데이터 전송 및 커널 호출 지연 발생 |
| **적정 규모** | 샘플 수 수만 개 이하 | 샘플 수 수십만 개 이상 또는 고해상도 이미지 |

### 테스트 제안

GPU의 진가를 확인하고 싶으시다면 다음 중 하나를 시도해 보세요:

1. **데이터 증폭:** 입력 데이터를 `tf.repeat` 등을 이용해 10만 개 이상으로 늘려보세요.
2. **모델 복잡도 증가:** 단순 Softmax가 아닌, `Dense` 레이어를 5개 이상 쌓은 깊은 신경망으로 수정해 보세요.
3. **이미지 처리:** MNIST나 CIFAR-10 같은 이미지 데이터셋으로 학습해 보시면 Metal이 CPU보다 훨씬 빨라지는 지점을 찾으실 수 있습니다.

혹시 모델의 레이어를 늘리거나 데이터 크기를 키워서 다시 테스트해보는 데 도움이 필요하신가요?
