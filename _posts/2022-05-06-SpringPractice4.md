---
title: 회원 도메인 개발
description:
categories:
 - baedalmate
tags:
---

# 회원 도메인 개발

**구현 기능**

- 회원 등록

- 회원 목록 조회

**순서**

- 회원 엔티티 코드 다시 보기

- 회원 리포지토리 개발

- 회원 서비스 개발

- 회원 기능 테스트

## 회원 리포지토리 개발

**MemberRepository**

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Member;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.PersistenceUnit;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class MemberRepository {
    private final EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }
    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {

        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}
```

**기술 설명**

- `@Repository`: 스프링 빈으로 등록, JPA 예외를 스프링 기반 예외로 예외 변환

- `@PersistenceContext`: 엔티티 매니저(`EntityManager`) 주입

- `@PersistenceUnit`: 엔티티 매니저 팩토리(`EntityManagerFactory`) 주입

- `@RequiredArgsConstructor`: final 변수를 자동 등록하여 생성자 생성

## 회원 서비스 개발

**MemberService**

```
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    // 회원 가입
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member);    // 중복 회원 검증
        memberRepository.save(member);
        return member.getId();
    }

    public void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다");
        }
    }

    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }


    // 회원 전체 조회

}

```

**기술 설명**

- `@Service`: 스프링 빈으로 등록

- `@Transactional`: 트랜잭션, 영속성 컨텍스트
  
  - `readOnly=true`: 데이터의 변경이 없는 읽기 전용 메서드에 사용. 영속성 컨텍스트를 플러시 하지 않으므로 약간의 성능 향상
  
  - 데이터베이스 드라이버가 지원하면 DB에서 성능 향상

- `@RequiredArgsConstructor`: final 변수를 자동 등록하여 생성자 생성

> **참고**: 실무에서는 검증 로직이 있어도 멀티 쓰레드 상황을 고려해서 회원 테이블의 회원명 컬럼에 유니크 제약 조건을 추가하는 것이 안전하다.

> 참고: 스프링 필드 주입 대신에 생성자 주입을 사용하자.

## 회원 기능 테스트
