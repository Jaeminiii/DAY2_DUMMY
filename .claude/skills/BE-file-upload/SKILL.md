---
name: BE-file-upload
description: 파일 업로드 시스템 구축. 로컬, S3, Cloudinary 등 다양한 스토리지 지원
context: fork
agent: be-agent
---

# BE 파일 업로드 가이드

## 개요
백엔드에서 파일 업로드 엔드포인트를 구현합니다. 여러 스토리지 옵션을 지원합니다.

## 지원 스토리지

### 1. 로컬 저장 (개발/테스트)
- 서버 파일시스템에 직접 저장
- 개발 환경에서만 사용 권장

### 2. AWS S3
- 프로덕션 권장
- 무제한 확장성
- CloudFront CDN 연동

### 3. Cloudinary
- 이미지 최적화 자동
- 무료 플랜 제공
- CDN 포함

### 4. Supabase Storage
- Supabase 사용 시 권장
- DB와 통합

## 파일 구조
```
backend/app/
├── routers/
│   └── upload.py         # 업로드 엔드포인트
├── services/
│   ├── s3.py            # S3 업로드 로직
│   ├── cloudinary.py    # Cloudinary 업로드 로직
│   └── local.py         # 로컬 저장 로직
└── config.py            # 환경 변수 설정
```

## 참조 문서
- [local-storage.md](references/local-storage.md) - 로컬 저장 구현
- [s3-upload.md](references/s3-upload.md) - AWS S3 업로드
- [cloudinary-upload.md](references/cloudinary-upload.md) - Cloudinary 업로드
- [validation.md](references/validation.md) - 파일 검증

## 구현 원칙
- ✅ 파일 타입 검증 (확장자, MIME type)
- ✅ 파일 크기 제한 (예: 5MB)
- ✅ 고유 파일명 생성 (UUID)
- ✅ 에러 처리 (업로드 실패 시)
- ❌ 원본 파일명 그대로 저장 금지 (보안)

## 설치 패키지

```bash
# 로컬 저장 (기본 내장)
# 추가 패키지 불필요

# S3
pip install boto3

# Cloudinary
pip install cloudinary

# Supabase
pip install supabase
```

## 기본 템플릿

### 업로드 엔드포인트
```python
from fastapi import APIRouter, UploadFile, File, HTTPException
from pathlib import Path
import uuid

router = APIRouter(prefix="/upload", tags=["upload"])

ALLOWED_EXTENSIONS = {".jpg", ".jpeg", ".png", ".webp", ".gif"}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

@router.post("/")
async def upload_file(file: UploadFile = File(...)):
    # 1. 파일 검증
    file_ext = Path(file.filename).suffix.lower()
    if file_ext not in ALLOWED_EXTENSIONS:
        raise HTTPException(400, "허용되지 않은 파일 형식입니다")

    # 2. 파일 크기 검증
    file.file.seek(0, 2)  # 파일 끝으로 이동
    file_size = file.file.tell()
    file.file.seek(0)  # 다시 처음으로

    if file_size > MAX_FILE_SIZE:
        raise HTTPException(400, "파일 크기는 5MB 이하여야 합니다")

    # 3. 고유 파일명 생성
    unique_filename = f"{uuid.uuid4()}{file_ext}"

    # 4. 스토리지에 업로드 (선택한 방식)
    url = await upload_to_storage(file, unique_filename)

    return {
        "url": url,
        "filename": unique_filename,
        "original_filename": file.filename,
        "size": file_size
    }
```

## 사용 예시

### 상품 이미지 업로드
```python
from app.models import Product
from sqlalchemy.orm import Session

@router.post("/products/{product_id}/images")
async def upload_product_image(
    product_id: int,
    file: UploadFile,
    db: Session = Depends(get_db)
):
    # 상품 존재 확인
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(404, "상품을 찾을 수 없습니다")

    # 이미지 업로드
    result = await upload_file(file)

    # DB에 이미지 URL 저장
    product.image_url = result["url"]
    db.commit()

    return result
```

## 보안 고려사항
- 파일 확장자 검증
- MIME 타입 검증
- 파일 크기 제한
- 고유 파일명 사용 (경로 탐색 공격 방지)
- 바이러스 스캔 (프로덕션에서는 ClamAV 등 활용)
