

# 이슈 1: notebook 패키지 필수 설치 여부

준비된 venv 가상 환경에서, cursor의 경우, 바로 ipynb 노트북의 play 버튼을 눌러 실행이 되는 반면
antigravity는 자꾸 jupyter notebook 패키지를 설치하라고 한다.

```
Running cells with '.venv (Python 3.14.0)' requires the notebook package.
```

ipykernel 패키지는 가상환경 내부에 pip으로 이미 설치 된 상태임.

## 원인

이 현상은 두 에디터가 Jupyter 커널과 통신하는 방식(Architecture)의 차이 때문에 발생합니다.

1. 원인 분석

현재 구성하신 .venv (Python 3.14) 환경을 확인해보니 ipykernel은 설치되어 있지만, notebook이나 jupyter 패키지는 설치되어 있지 않습니다.

  - Cursor의 방식 (Raw Kernel): Cursor는 (그리고 VS Code의 일부 모드도) ipykernel 패키지만 있으면 ZeroMQ라는 프로토콜을 통해 파이썬 커널과 직접 통신할 수 있습니다. 즉, 무거운 Jupyter 웹 서버를 띄우지 않고도 코드 실행 결과를 주고받을 수 있도록 최적화되어 있어 notebook 패키지가 없어도 잘 동작합니다.

  - Antigravity의 방식 (Jupyter Server): Antigravity(또는 표준 VS Code Jupyter 확장)는 안정성과 Jupyter의 모든 기능(인터랙티브 위젯, 확장 기능 등)을 완벽하게 지원하기 위해, 내부적으로 로컬 Jupyter Server를 구동하여 연결하는 방식을 사용합니다. 이 과정에서 파이썬 환경 내에 notebook 패키지가 설치되어 있어야 서버를 시작할 수 있는데, 현재 없기 때문에 에러가 발생하는 것입니다.

2. 해결 방법

Antigravity에서 작업을 계속하시려면 에러 메시지가 제안한 대로 notebook 패키지를 추가로 설치해주시면 됩니다. 터미널에서 아래 명령어를 실행해 주세요.

```bash
pip install notebook
```

(참고: Python 3.14는 최신 개발 버전이라 일부 패키지 호환성 이슈가 있을 수 있으나, ipykernel이 잘 설치된 것으로 보아 notebook도 설치가 가능할 것으로 예상됩니다.)

## 추가 궁금증: notebook 패키지 설치 후 Cursor의 동작

**Q. 가상환경에 `notebook` 패키지를 설치하면, Cursor는 어떤 방식으로 연결하나요?**

**A. 여전히 ZeroMQ (Raw Kernel) 방식을 우선 사용합니다.**

Cursor(및 최신 VS Code)는 `notebook` 패키지 설치 여부와 관계없이, 로컬 환경에서는 **Raw Kernel (ZeroMQ)** 연결을 최우선으로 시도합니다. 이 방식이 더 빠르고 가볍기 때문입니다. `notebook` 패키지가 설치되었다고 해서 강제로 Jupyter Server 모드로 전환되지 않습니다.

**Q. Cursor만 쓴다면 무거운 Jupyter 서버는 실행되지 않나요?**

**A. 네, 실행되지 않습니다.**

`notebook` 패키지가 설치되어 있더라도, Cursor가 Raw Kernel 방식으로 연결한다면 무거운 Jupyter 웹 서버(`jupyter-notebook` 또는 `jupyter-server` 프로세스)는 구동되지 않습니다. 대신 단순히 파이썬 커널 프로세스(`python -m ipykernel_launcher`)만 실행되므로 **여전히 가벼운 상태**를 유지합니다.

결론적으로, Antigravity를 위해 `notebook`을 설치하더라도 Cursor 사용성이나 성능에는 부정적인 영향을 미치지 않습니다.
