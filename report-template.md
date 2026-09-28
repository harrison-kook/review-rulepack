# 코드 리뷰 리포트

> 이 템플릿은 `renderer-markdown` / `renderer-pr-comment`가 Findings JSON을 사람이 읽는 리포트로 렌더링할 때 사용하는 구조를 정의한다. `{{ }}`는 치환 변수를 뜻한다.

## 요약

- 대상: `{{repo}}` / `{{ref}}` (`{{mode}}` 모드)
- 적용 프로파일: `{{profiles}}`
- 전체 지적: `{{total_count}}`건 (HIGH `{{high_count}}` · MEDIUM `{{medium_count}}` · LOW `{{low_count}}` · INFO `{{info_count}}`)
- Gate 결과: `{{gate_status}}` (`fail_on: {{fail_on}}`)

## 규칙별 위반 집계

| 규칙 ID | 심각도 | 건수 |
|---|---|---|
| `{{ruleId}}` | `{{severity}}` | `{{count}}` |

## 상세 지적

각 Finding마다 아래 블록을 반복한다.

### [{{severity}}] {{ruleId}} — {{file}}:{{line}}

{{message}}

```{{language}}
{{evidence}}
```

**제안**: {{suggestion}}

---

## 테스트 결과 (2단계 이후)

- 생성된 테스트: `{{generated_test_count}}`건 (통과 `{{passed}}` / 실패 `{{failed}}`)
- 커버리지 증분: `{{coverage_delta}}`
- 뮤테이션 스코어: `{{mutation_score}}`

## 참고

- 근거(`evidence`) 없는 지적은 이미 필터링되어 이 리포트에 포함되지 않는다.
- 중복 지적은 `fingerprint` 기준으로 제거되었다.
