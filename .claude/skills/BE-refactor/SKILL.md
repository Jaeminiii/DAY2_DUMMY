---
name: BE-refactor
description: FastAPI 백엔드 코드를 리팩토링합니다. 코드 정리, 네이밍 개선, 중복 제거, 구조 개선 요청 시 사용합니다.
context: fork
agent: be-agent
---

# BE Refactor 가이드

## 개요
기존 FastAPI 코드를 더 읽기 쉽고 유지보수하기 좋게 개선합니다.

## 리팩토링 체크리스트
1. **네이밍**: 변수, 함수, 클래스명이 명확한가?
2. **중복 제거**: 반복되는 코드가 있는가?
3. **함수 분리**: 하나의 함수가 너무 많은 일을 하는가?
4. **타입 힌트**: 타입이 명시되어 있는가?
5. **에러 처리**: 예외 처리가 적절한가?

## 참조 문서
- [naming.md](references/naming.md) - 네이밍 규칙
- [structure.md](references/structure.md) - 구조 개선
- [patterns.md](references/patterns.md) - FastAPI 패턴

## 리팩토링 원칙
- ✅ 동작을 변경하지 않고 구조만 개선
- ✅ 작은 단위로 점진적 개선
- ✅ 테스트 통과 확인 후 진행
- ❌ 과도한 추상화 금지
- ❌ 불필요한 파일 분리 금지