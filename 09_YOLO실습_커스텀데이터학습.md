# 9. YOLO 실습: 커스텀 데이터 학습

## 1) 폴더 구조 예시
```
dataset/
  images/
    train/
    val/
  labels/
    train/
    val/
  data.yaml
```

## 2) 학습 실행 예시
```bash
yolo detect train data=dataset/data.yaml model=yolov8n.pt epochs=50 imgsz=640
```

## 3) 추론 실행 예시
```bash
yolo detect predict model=runs/detect/train/weights/best.pt source=sample.jpg
```

## 4) 결과 해석
- mAP50, mAP50-95
- Precision / Recall
- 실제 누락 케이스(오검출/미검출) 확인

## 5) 개선 루프
오류 케이스 수집 → 라벨 수정/추가 → 재학습 반복
