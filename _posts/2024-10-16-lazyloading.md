---
layout : single
title : "[Project] 임시저장 후 제출 구현 시 문제 :HibernateException"
categories: Project
tag : [project3, practice]
author_profile: true
---

📌 개인적인 공간으로 공부를 기록하고 복습하기 위해 사용하는 블로그입니다. <br>
정확하지 않은 정보가 있을 수 있으니 참고바랍니다 :😸 <br>
[틀린 내용은 댓글로 남겨주시면 복받으실거에요]  
{: .notice--primary}

## 전자결재 임시 저장 > 다시 작성  > 제출 과정 구현 중 오류 발생

프론트에서 임시저장도 db에 저장되어있어야 해서 db에 저장하는 것으로 변경하였고

다시 작성 버튼을 누른 다음 제출 할 때 patch 요청을 하는 것으로 변경하다가 예외가 발생했다.

### HibernateException 발생

1. 콘솔의 오류메세지
    
    <img src = "https://github.com/user-attachments/assets/3e694237-5a13-4530-b96e-9118ebe08c28" width=500/>
    
    <img src = "https://github.com/user-attachments/assets/bded1c55-bf70-4414-a2c9-44f5b8680697" width=500/>
    
2. 원인
    - 에러메세지를 읽어보면 Workflow 엔티티의 approvals 가 더 이상 소유 엔티티에서 참조되지 않아 HibernateException이 발생했다는 내용이다.
    - 나는 apporval을 orphanRemoval을 true로 설정해 두었는데 HibernateException은 cascade=all, orphanRemoval이 설정이 된 컬렉션이 소유 엔티티에서 분리되었을 때 발생한다고 한다.
        
        <img src = "https://github.com/user-attachments/assets/6a337a08-4bf2-4d60-ab8d-7bcebda456d9" width=500/>
        
    - 원래 `Hibernate`에서는 `cascade = CascadeType.ALL, orphanRemoval = true`으로 설정된 컬렉션에 대해, 소유 엔티티에서 해당 컬렉션을 참조하지 않게 되면 그 컬렉션의 요소들을 **고아 객체**로 인식하고 삭제하는데 Hibernate는 **컬렉션 자체가 완전히 교체**되면, 이전 컬렉션과의 참조가 끊어졌다고 판단하여 `HibernateException`을 발생시킨다.
    - 아무래도 임시저장때 사용된 결재선의 승인자를 교체하면서 발생된 것 같다. > 임시저장 때 설정했던 결재선을 다시 작성 때 수정할 수 있다 보니 …고아 객체가 발생되었다.

### 문제 해결 - 기존 approvals 업데이트

오류를 해결하기 위해 `Workflow` 엔티티의 `approvals` 직접 교체하는 대신, 기존 `approvals` 을 업데이트하는 방식으로 수정하였다.
다른 방법을 하기에는 수정 범위가 커질 수 도 있고 (cascade, orphanRemoval 설정을 변경 시), 남은 시간이 별로 없어서 여기에서만 빠르게 처리하기 위해서 였다.

```java
public Document updateDocument(Document document) {

    Document findDocument = findVerifiedDocument(document.getId());

    // Workflow 업데이트
    Optional.ofNullable(document.getWorkflow())
            .ifPresent(newWorkflow -> {
                Workflow existingWorkflow = findDocument.getWorkflow();
                if (existingWorkflow != null) {
                    // 기존 approvals 컬렉션 업데이트
                    List<Approval> existingApprovals = existingWorkflow.getApprovals();

                    // 새로운 Approval들의 ID 수집
                    Set<Long> newApprovalIds = newWorkflow.getApprovals().stream()
                            .filter(approval -> approval.getId() != null)
                            .map(Approval::getId)
                            .collect(Collectors.toSet());

                    // 기존 Approval 중에서 새로운 Approval에 없는 것들을 제거
                    existingApprovals.removeIf(approval -> !newApprovalIds.contains(approval.getId()));

                    // 새로운 Approval 추가 또는 기존 Approval 업데이트
                    for (Approval approval : newWorkflow.getApprovals()) {
                        if (approval.getId() == null) {
                            // 새로운 Approval인 경우
                            approval.setWorkflow(existingWorkflow);
                            existingApprovals.add(approval);
                        } else {
                            // 기존 Approval인 경우 필요한 필드 업데이트
                            Approval existingApproval = existingApprovals.stream()
                                    .filter(a -> a.getId().equals(approval.getId()))
                                    .findFirst()
                                    .orElseThrow(() -> new BusinessLogicException(ExceptionCode.APPROVAL_NOT_FOUND));
                            // 필요한 필드 업데이트
                        }
                    }

                    // currentStep 등 다른 속성 업데이트
                    existingWorkflow.setCurrentStep(newWorkflow.getCurrentStep());

                } else {
                    // 새로운 Workflow 설정
                    Workflow findWorkflow = workflowRepository.findById(document.getWorkflow().getId())
                            .orElseThrow(() -> new BusinessLogicException(ExceptionCode.WORKFLOW_NOT_FOUND));

                    EmployeeData employee = verifiedEmployee(document.getEmployeeId());

                    // 결재라인에 담당자 추가
                    Workflow addApproval = insertAuthor(findWorkflow, employee.getEmployeeId());
                    findDocument.setWorkflow(addApproval);
                }
                
                ...
                
                
            });
```

### 추가 문제 발생 : approvals 로딩 안됨

 postman으로 테스트 하니 기존문제는 해결되었으나 원래 나오던 approval은 나오지 않았다.
 
 <img src = "https://github.com/user-attachments/assets/7f13b1e9-e809-47dd-968f-d1ecd81c1d81" width=500/>

기존에는 잘 나오다가 patch 메서드로 변경하고 나서 로딩이 안되는데 지연로딩 문제라고 생각 되었다.

1. 지연로딩 문제
    - @OneToMany  는 기본적으로 지연 로딩이 적용되는데 Workflow에서 approvals가 @OneToMany 로 Setting 되어있기 때문이다.
2. 문제 해결 : 
    - approval이 함께 조회되지 않는 문제를 해결하기 위해서 EntityGraph 또는 JPQL의 Fetch Join을 사용하여 연관된 엔티티를 함께 로드하는 방법에 대해 생각했다.
    - 찾아보니 두 방법 모두 동일한 방식으로 작동하기 때문에(유사한 쿼리 생성) 성능 차이는 없다고 생각되어 좀 더 쓰기 간단한 @EntityGraph를 사용하기로 했다.(하지만 많이 사용하면 조인 테이블 수가 많아져서 성능이 저하 될 수 있음!@)
    
    > @EntityGraph
    Spring Data JPA에서 엔티티의 연관된 엔티티를 함께 로드하기 위해 사용
    `attributePaths`에 명시된 연관 관계를 **Eager Fetch** 방식으로 가져온다.
    내부적으로는 JPA가 제공하는 엔티티 그래프 기능을 사용하여 쿼리를 생성
    > 

---

### 알고보니 Transaction 문제일 수 있다…!

추후에 더 찾아본 후 알게 된 사실은 지연 로딩으로 인해 트랜잭션 범위를 벗어나 approvals 컬렉션이 초기화되지 않았기 때문이라고 한다.

> 지연 로딩된 연관 엔티티를 실제로 접근하려면, 해당 엔티티가 로드될 때 세션이 열려있어야 한다. 보통 트랜잭션이 열려있는 동안 세션이 유지되므로, 트랜잭션 범위 내에서 접근하면 지연 로딩된 데이터를 정상적으로 로드할 수 있지만 트랜잭션이 종료된 후에 지연 로딩된 데이터를 접근하려고 하면 **LazyInitializationException**이 발생하게 된다고 한다.
> 

근데 문제는 해당 서비스에서 애초에 @Transactional  설정이 되어있지 않았다…😥

결국 @Transactional 에 대한 문제 일 수 있다.

1. 트랜잭션 범위
    - **트랜잭션**은 데이터베이스의 상태를 일관되게 유지하기 위한 단위 작업
    - **트랜잭션 범위**는 특정 작업이 수행되는 동안 열려있는 트랜잭션의 기간을 의미
2. @Transcational을 사용하지 않아서 생긴 문제점
    - 지연 로딩(Lazy Loading)은 실제로 연관된 엔티티가 필요할 때 데이터를 로드하는 방식인데 이때 Hibernate 세션이 열려 있어야만 연관된 데이터를 로드할 수 있다.
    - 트랜잭션은 Hibernate 세션의 범위를 정의하며, 트랜잭션이 종료되면 세션도 종료된다.
    - 따라서 트랜잭션 범위 내에서만 지연 로딩이 제대로 동작하게 되는 것이다.
    - 또한 데이터 일관성 문제가 생길 수 있다! (여러 데이터베이스 작업이 하나의 트랜잭션으로 묶이지 않아 중간에 오류가 발생하면 데이터 일관성이 깨질 수 있다.)

무튼 아래와 같이 2가지 문제점을 해결했는데 조금 몰랐던 빈틈이 생겨서 다시 공부할 수 있는 시간이 되었다.,

부족함을 다시 알게 되었고 지연로딩이랑 즉시로딩 등 jpa에서 발생했던 에러가 매번 프로젝트마다 생기는데 끝나고 더 깊이 공부할 수 있는 시간을 가지고 싶다.,,,

- workflow 수정 가능하도록 고아객체 문제 해결
- 지연 로딩으로 인한 문제(approvals 나오지 않는 현상) [@entitygraph](https://github.com/entitygraph) 사용하여 개선

[https://github.com/pingpong-works/core-api/issues/55](https://github.com/pingpong-works/core-api/issues/55)