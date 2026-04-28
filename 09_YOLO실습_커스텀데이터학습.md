# 9. YOLO 실습: 커스텀 데이터 학습 (현업 입문 버전)

> 목표: 내 불량 이미지 데이터로 YOLO 학습/추론을 실제로 1회 수행

---

## 0) 먼저 이해할 것

YOLO 탐지는 "정답 박스 라벨"이 있어야 학습됩니다.
즉, 이미지 + 각 이미지의 라벨(txt)이 필요합니다.

---

## 1) 환경 준비

```bash
pip install ultralytics
```

GPU가 있다면 학습 속도가 훨씬 빠릅니다.

---

## 2) 데이터셋 구조

아래 구조를 정확히 맞추세요.

```
yolo_dataset/
  images/
    train/
    val/
  labels/
    train/
    val/
  data.yaml
```

- `images/train/a.jpg`가 있으면
- `labels/train/a.txt`도 있어야 함

### 라벨(txt) 형식
한 줄당 객체 1개:

```
<class_id> <x_center> <y_center> <width> <height>
```

- 좌표는 0~1 사이 정규화 값

---

## 3) data.yaml 예시

`yolo_dataset/data.yaml`

```yaml
path: ./yolo_dataset
train: images/train
val: images/val

names:
  0: scratch
  1: dent
  2: stain
```

클래스 이름은 네 불량 유형에 맞게 바꾸면 됩니다.

---

## 4) 학습 명령어 (첫 실행 권장값)

```bash
yolo detect train \
  data=yolo_dataset/data.yaml \
  model=yolov8n.pt \
  epochs=50 \
  imgsz=640 \
  batch=16
```

설명:
- `yolov8n.pt`: 가벼운 입문 모델
- `epochs=50`: 처음엔 30~50 추천
- `imgsz=640`: 기본 해상도

---

## 5) 추론(테스트) 명령어

```bash
yolo detect predict \
  model=runs/detect/train/weights/best.pt \
  source=sample_test_images \
  conf=0.25
```

- 결과 이미지는 `runs/detect/predict/`에 저장

---

## 6) 성능 확인 포인트

학습이 끝나면 로그에 주요 지표가 나옵니다.

- Precision
- Recall
- mAP50
- mAP50-95

### 현업 해석 팁
- 불량 놓침이 치명적이면 recall 우선
- 오검출이 작업 효율을 망치면 precision도 함께 관리

---

## 7) 실무에서 바로 쓰는 개선 루프

1. 미검출/오검출 이미지 수집
2. 라벨 수정(누락 박스, 오라벨 수정)
3. train/val 재분리 점검
4. 재학습
5. 동일 조건으로 재평가

이 반복이 성능을 가장 많이 끌어올립니다.

---

## 8) 자주 발생하는 오류

### 오류 A: 라벨 파일이 없다고 나옴
- 이미지와 txt 파일명 동일한지 확인
- train/val 경로 오타 확인

### 오류 B: 학습은 되는데 결과가 이상함
- 라벨 좌표 형식 오류 가능성 큼
- 클래스 id와 names 매핑 확인

### 오류 C: 특정 불량을 계속 못 찾음
- 해당 클래스 데이터 부족
- 박스가 너무 작거나 라벨 품질 낮음

---

## 9) 현업용 질문(팀 대화용)

- 현재 목표는 미검출 최소화인가, 오검출 최소화인가?
- 실제 라인에서 허용 가능한 추론 시간은 몇 ms인가?
- 신규 불량 타입이 생기면 라벨링/재학습 주기는?
- 성능 리포트는 클래스별로 관리하는가?

이 질문들을 이해하고 말할 수 있으면, 노베이스여도 현업 적응 속도가 빨라집니다.
