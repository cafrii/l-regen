
# Python 3.14 와 Tensorflow 궁합 문제

## 이슈

tensorflow 패키지 설치 못하는 문제

## 간략한 결론

Python 3.11 을 설치하도록 한다.


## 검토 작업

```
$ pip index versions tensorflow

ERROR: No matching distribution found for tensorflow
```

현재 사용 중인 Python 3.14용 TensorFlow는 아직 출시되지 않았습니다.

확인 결과

- 현재 환경: Python 3.14.0이 설치되어 있습니다. (2025년 10월 출시)
- TensorFlow 지원 현황: 최신 안정 버전인 TensorFlow 2.20.0 및 (tf-nightly). 모두 현재 Python 3.13 까지만 공식 지원하고 있습니다.
- 설치 실패 원인: Python 3.14용 빌드(Wheel 파일)가 PyPI 서버에 없기 때문에 pip install 시 "No matching distribution found" 오류가 발생합니다.

💡 해결 방법

TensorFlow를 사용하시려면 Python 버전을 조금 낮춘 가상 환경을 만드시는 것을 권장합니다.

1. 가장 권장되는 버전: 현재 가장 안정적으로 지원되는 Python 3.11 또는 3.12를 사용하는 것이 좋습니다. (3.13은 최신이라 일부 라이브러리와 충돌이 있을 수 있습니다.)

2. 가상 환경 생성 예시 (pyenv나 conda 사용 시):
```bash
# pyenv 사용 시
pyenv install 3.12.7
pyenv local 3.12.7
python -m venv venv
source venv/bin/activate
pip install tensorflow
```

3. 대안: 꼭 최신 환경이 필요하지 않다면, Google Colab과 같은 환경을 사용하면 이미 구성된 TensorFlow 환경을 바로 사용할 수 있습니다.

TensorFlow 팀은 보통 새로운 Python 버전이 출시된 후 수개월 내에 지원 버전을 업데이트하므로, 3.14 지원은 조금 더 기다려야 할 것으로 보입니다. 

