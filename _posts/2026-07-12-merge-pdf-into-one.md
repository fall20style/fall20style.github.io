---
title: 현재 디렉터리의 모든 PDF 파일 순서대로 합치기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
tags:
  - python
categories:
toc: true
toc_sticky: true
toc_label: 목차
description: desc가 여기에
---


# [Python] 파이썬으로 현재 디렉터리의 모든 PDF 파일 순서대로 합치기 (pypdf 활용)

여러 개로 나누어진 PDF 파일을 하나로 합쳐야 할 때, 파이썬을 이용하면 단 몇 줄의 코드로 자동화할 수 있습니다.
특히 파일명이 날짜나 숫자로 되어 있는 경우, 순서대로 정렬하여 병합하는 스크립트입니다.


---

## 1. 개발 환경 구성

```bash
# 기존 구버전 라이브러리 삭제 및 최신 버전 설치
pip uninstall pypdf PyPDF2 -y
pip install --upgrade pypdf
```

---

## 2. 파이썬 스크립트 작성 (`merge.py`)

현재 디렉터리에 있는 `.pdf` 확장자 파일을 모두 찾아서 파일명 순으로 정렬한 뒤, `merged_result.pdf`라는 하나의 파일로 병합하는 코드입니다.

```python
import os
from pypdf import PdfWriter

def merge_pdfs_in_current_dir(output_filename="merged_result.pdf"):
    # PDF 병합을 위한 Writer 객체 생성
    merger = PdfWriter()
    current_dir = os.getcwd()
    
    # 1. 현재 디렉터리의 파일 목록 중 PDF만 골라 이름순 정렬
    # (예: 20260706_01.pdf, 20260706_02.pdf ... 순서로 정확히 정렬됨)
    pdf_files = sorted([f for f in os.listdir(current_dir) if f.lower().endswith('.pdf')])
    
    # 2. 출력 파일이 이미 존재한다면 병합 대상에서 제외 (무한 루프 방지)
    if output_filename in pdf_files:
        pdf_files.remove(output_filename)
        
    if not pdf_files:
        print("합칠 PDF 파일이 현재 디렉터리에 없습니다.")
        return

    print(f"총 {len(pdf_files)}개의 파일을 병합합니다...")
    
    # 3. 정렬된 순서대로 PDF 파일 추가
    for pdf in pdf_files:
        print(f"추가 중: {pdf}")
        merger.append(pdf)

    # 4. 하나로 합쳐진 파일 저장
    with open(output_filename, "wb") as f:
        merger.write(f)
        
    merger.close()
    print(f"✔ 병합 완료! 생성된 파일: {output_filename}")

if __name__ == "__main__":
    merge_pdfs_in_current_dir()
```

---

## 3. 코드의 주요 특징
* **자동 정렬 (`sorted`)**: 파일 이름이 숫자나 날짜 기반으로 되어 있다면, 기재된 순서 그대로 차례대로 이어 붙여집니다.
* **예외 처리**: 스크립트를 여러 번 실행하더라도 새로 생성된 결과 파일(`merged_result.pdf`)이 다시 병합 대상에 들어가지 않도록 안전 코드가 포함되어 있습니다.
* **하위 호환성 확보**: `PdfMerger`에서 간혹 발생하는 `ImportError`를 방지하기 위해 `PdfWriter` 기반으로 작성하여 버전 상관없이 안정적으로 동작합니다.

---

## 4. 실행 방법
병합하려는 PDF 파일들이 모여 있는 디렉터리로 이동한 뒤 스크립트를 실행합니다.

```bash
python merge.py
```

**실행 결과 예시:**
```text
총 12개의 파일을 병합합니다...
추가 중: 20260706_0119_02001.pdf
추가 중: 20260706_0119_02002.pdf
...
추가 중: 20260706_0119_02015.pdf
✔ 병합 완료! 생성된 파일: merged_result.pdf
```
