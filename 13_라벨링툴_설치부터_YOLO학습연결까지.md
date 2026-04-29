# 14. 라벨링 툴 설치부터 YOLO 학습 연결까지 (디텍션/세그멘테이션)

> 목표: 라벨링 툴 실행 → 라벨 생성 → YOLO 학습/추론까지 한 번에 연결

---

## 0) 전체 흐름 요약

1. 라벨링 툴 설치/실행
2. 이미지에 라벨 작성
3. YOLO 형식 데이터셋 정리
4. 학습 실행
5. 추론 결과 확인

---

## 1) 공통 준비

```bash
python -m venv .venv
.venv\Scripts\activate
pip install --upgrade pip
pip install ultralytics opencv-python
```

작업 루트 예시:

```text
C:\vision-work
```

---

## 2) 디텍션(박스) 라벨링: labelImg

## 2-1) 설치/실행

```bash
pip install labelImg
labelImg
```

## 2-2) 기본 설정

- Open Dir: 원본 이미지 폴더
- Change Save Dir: 라벨 저장 폴더
- 라벨 포맷: **YOLO** 선택
- 클래스 목록 파일(classes.txt) 준비 (예: scratch, dent, stain)

## 2-3) 라벨링 방법

- 박스로 불량 영역 지정
- 클래스 선택
- 저장(Ctrl+S)

저장 결과: 이미지와 같은 이름의 `.txt`

---

## 3) 세그멘테이션(폴리곤) 라벨링: labelme

## 3-1) 설치/실행

```bash
pip install labelme
labelme
```

## 3-2) 라벨링 방법

- Open: 이미지 폴더
- Create Polygon: 불량 경계 점 찍기
- 클래스 이름 입력
- Save → JSON 생성

> 주의: labelme 기본 출력은 JSON이라, YOLO-seg 학습용으로 변환이 필요할 수 있음.

---

## 4) YOLO 데이터셋 구조 만들기

디텍션/세그 모두 기본 폴더 구조는 비슷합니다.

```text
dataset/
  images/
    train/
    val/
  labels/
    train/
    val/
  data.yaml
```

- `images/train/a.jpg` ↔ `labels/train/a.txt` 파일명 1:1 일치

---

## 5) data.yaml 작성

예시:

```yaml
path: ./dataset
train: images/train
val: images/val
names:
  0: scratch
  1: dent
  2: stain
```

클래스 순서(names)와 라벨의 class id가 정확히 맞아야 합니다.

---

## 6) YOLO 학습/추론 연결

## 6-1) YOLOv8 디텍션 학습

```bash
yolo detect train data=dataset/data.yaml model=yolov8n.pt epochs=50 imgsz=640
```

## 6-2) YOLOv8 디텍션 추론

```bash
yolo detect predict model=runs/detect/train/weights/best.pt source=test_images conf=0.25
```

결과 폴더:

```text
runs/detect/predict/
```

---

## 7) YOLOv8 세그멘테이션 학습/추론

## 7-1) 학습

```bash
yolo segment train data=dataset/data.yaml model=yolov8n-seg.pt epochs=50 imgsz=640
```

## 7-2) 추론

```bash
yolo segment predict model=runs/segment/train/weights/best.pt source=test_images conf=0.25
```

결과 폴더:

```text
runs/segment/predict/
```

---

## 8) YOLOv5를 계속 쓰고 싶다면 (네 이전 환경 호환)

`C:\jsy.dev\python\yolov5` 기준:

### 디텍션 학습
```bash
python train.py --img 1280 --batch 16 --epochs 100 --data ./dataset/dataset.yaml --weights yolov5s.pt --name mask
```

### 디텍션 추론
```bash
python detect.py --weights runs/train/mask/weights/best.pt --source ./test_images --conf 0.25
```

결과 폴더:

```text
runs/detect/exp*/
```

---

## 9) 자주 막히는 지점 (체크리스트)

- [ ] 가상환경 활성화했는가?
- [ ] `yolo` 또는 `python train.py` 명령이 현재 폴더 기준으로 맞는가?
- [ ] 이미지-라벨 파일명이 정확히 같은가?
- [ ] class id와 data.yaml의 names 순서가 같은가?
- [ ] train/val 경로 오타가 없는가?

---

## 10) 추천 진행 순서 (노베이스)

1. **디텍션부터 시작** (labelImg + YOLO)
2. 작은 데이터셋으로 1회 완주
3. 오검출/미검출 분석
4. 필요 시 세그멘테이션으로 확장

처음부터 세그멘테이션 올인보다, 디텍션으로 파이프라인 익히는 게 훨씬 빠릅니다.
