# K8s CRD & Operator 학습 프로젝트

Kubernetes의 **CRD(Custom Resource Definition)** 와 **Operator 패턴**을 배우기 위한 예제 프로젝트.

## 기본 결정

- 언어/도구: **Go + Kubebuilder** (공식 방식, 자료가 가장 많음)
- 로컬 클러스터: **kind**
- 예제 도메인: **`Website` CRD**
  - `spec.image`, `spec.replicas`를 선언하면 Operator가 Deployment와 Service를 만들어 준다.
  - 작지만 reconcile 패턴을 모두 보여준다.

## 학습 단계

| 단계 | 목표 | 배우는 것 |
|---|---|---|
| 0. 환경 | Go, Docker, kind, kubectl, kubebuilder 설치 후 `kind create cluster` | |
| 1. CRD만 직접 | Operator 없이 YAML로 CRD를 만들고 `kubectl apply`로 CR 생성 | CRD, CR, OpenAPI 스키마, `kubectl get <kind>` |
| 2. 스캐폴딩 | `kubebuilder init`, `kubebuilder create api --kind Website` | 프로젝트 구조, `api/`, `controllers/`, Makefile |
| 3. 타입 정의 | `WebsiteSpec`(image, replicas)과 `WebsiteStatus`(availableReplicas) 정의, `make manifests`로 CRD 생성 | marker 주석, spec/status 분리 |
| 4. Reconcile 기본 | CR을 읽어서 Deployment 생성 | Reconcile 루프, 선언적 모델, 멱등성 |
| 5. 소유권 | `SetControllerReference`와 `Owns(&Deployment{})` 추가, Service도 생성 | OwnerReference, 가비지 컬렉션, watch |
| 6. Status 갱신 | Deployment 상태를 CR의 `.status`에 반영, Condition 추가 | Status subresource, Conditions 규약 |
| 7. Finalizer | 삭제 시 정리 로직(외부 리소스 흉내) 추가 | Finalizer, 삭제 흐름 |
| 8. 검증 | CEL 검증 마커 또는 Validating Webhook으로 `replicas >= 0` 검증 | Admission, Webhook |
| 9. 테스트 | envtest로 controller 테스트 1~2개 작성 | envtest, 통합 테스트 |
| 10. 배포 | `make docker-build` → `kind load` → `make deploy` | RBAC, Deployment로 실행되는 Operator |

## 예상 디렉터리 구조 (Kubebuilder 생성)

```
operator-example/
├── api/v1/website_types.go                    # CRD 타입
├── internal/controller/website_controller.go  # Reconcile 로직
├── config/{crd,rbac,samples}/                 # 생성된 매니페스트
├── docs/                                      # 단계별 학습 메모
└── Makefile
```

## 진행 방식

- 단계마다 짧은 커밋 + `docs/NN-*.md` 메모 한 장 (이해한 개념, 실행 명령).
- 먼저 **1~4단계**를 진행한다. 4단계를 마치면 CR 생성 시 Deployment가 만들어지는 것을 확인할 수 있다.

## 범위 밖 (필요하면 나중에)

Operator SDK, Helm/Ansible 기반 operator, OLM 패키징, 멀티 클러스터.
