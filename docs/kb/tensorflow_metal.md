

# tensorflow-metal 설치 후 tensorflow 설치 에러

## 문제

mac 기기에서 GPU 가속을 위해 tensorflow-metal 설치 후 tensorflow 임포트 에러 발생.

## 간단한 결론

metal 과 동작 검증이 된 tensorflow 구 버전을 설치해야 함.

```
%pip install tensorflow==2.16.2 tensorflow-metal==1.2.0
```


## 상세

```
%pip install tensorflow-metal

import tensorflow as tf
```

에러 메시지
```
NotFoundError                             Traceback (most recent call last)
Cell In[1], line 1
----> 1 import tensorflow as tf

File ~/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow/__init__.py:438
    436 _plugin_dir = _os.path.join(_s, "tensorflow-plugins")
    437 if _os.path.exists(_plugin_dir):
--> 438   _ll.load_library(_plugin_dir)
    439   # Load Pluggable Device Library
    440   _ll.load_pluggable_device_library(_plugin_dir)

File ~/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow/python/framework/load_library.py:151, in load_library(library_location)
    148     kernel_libraries = [library_location]
    150   for lib in kernel_libraries:
--> 151     py_tf.TF_LoadLibrary(lib)
    153 else:
    154   raise OSError(
    155       errno.ENOENT,
    156       'The file or folder to load kernel libraries from does not exist.',
    157       library_location)

NotFoundError: dlopen(/Users/yhlee/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow-plugins/libmetal_plugin.dylib, 0x0006): Library not loaded: @rpath/_pywrap_tensorflow_internal.so
  Referenced from: <8B62586B-B082-3113-93AB-FD766A9960AE> /Users/yhlee/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow-plugins/libmetal_plugin.dylib
  Reason: tried: '/Users/yhlee/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow-plugins/../_solib_darwin_arm64/_U@local_Uconfig_Utf_S_S_C_Upywrap_Utensorflow_Uinternal___Uexternal_Slocal_Uconfig_Utf/_pywrap_tensorflow_internal.so' (no such file), '/Users/yhlee/work/ai/ml_study/l-regen/.venv/lib/python3.11/site-packages/tensorflow-plugins/../_solib_darwin_arm64/_U@local_Uconfig_Utf_S_S_C_Upywrap_Utensorflow_Uinternal___Uexternal_Slocal_Uconfig_Utf/_pywrap_tensorflow_internal.so' (no such file)

```

## 원인 분석

맥북 M4에서 tensorflow-metal을 설치한 후 발생하는 이 에러는 TensorFlow 본체와 Metal 플러그인 간의 버전 불일치로 인해 발생하는 전형적인 라이브러리 경로 문제입니다.

🚨 에러 원인

현재 설치된 TensorFlow 2.20.0은 아주 최신 버전인 반면, 애플에서 제공하는 tensorflow-metal(1.2.0) 플러그인은 내부적으로 이전 버전의 파일 구조(_pywrap_tensorflow_internal.so)를 찾으려고 합니다. 최신 TensorFlow에서는 이 파일의 위치나 이름이 변경되어 경로(RPATH)를 찾지 못하고 튕겨나가는 것입니다.

🛠️ 해결 방법: 추천 버전으로 다운그레이드

현재 Mac(M1/M2/M3/M4) 환경에서 가장 안정적으로 가속 기능을 지원하는 TensorFlow 2.16.0 대 버전으로 맞추는 것이 가장 확실한 해결책입니다.

1. 기존 패키지 삭제 및 재설치 (주피터 노트북 셀에서 실행):

```python
# 기존 충돌 패키지 삭제
%pip uninstall -y tensorflow tensorflow-metal
# 안정적인 조합으로 재설치 (Python 3.11 기준)
%pip install tensorflow==2.16.2 tensorflow-metal==1.2.0
```

2. 커널 재시작 (필수!)

- 상단 메뉴의 Restart 버튼을 눌러 파이썬 프로세스를 완전히 새로 시작해 주세요.

3. 다시 확인

```python
import tensorflow as tf
print("TensorFlow 버전:", tf.__version__)
print("가속 장치 목록:", tf.config.list_physical_devices('GPU'))
```

왜 2.16.2 인가요?

애플의 Metal 플러그인 업데이트 속도가 본체인 TensorFlow 업데이트 속도보다 느리기 때문입니다. 2.16.x 버전은 M 시리즈 칩의 하드웨어 가속 성능을 가장 오류 없이 뽑아낼 수 있는 '검증된 버전(Golden Version)'입니다.


