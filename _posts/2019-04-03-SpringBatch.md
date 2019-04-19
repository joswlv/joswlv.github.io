---
layout: post
title: SpringBatch 정리
date: 2019-04-03 15:10
categories: Spring
---

# SpringBatch

## Batch특징

- 특징
	- 대용량 데이터 처리
	- 사람의 조작없이 자동으로 처리
	- 주기적 실행
- 배치실행의 예
	- 신용카드 청구서 발행
	- 마켓팅 이메일 발송

-> 배치 사용하기 전에 배치를 꼭 사용하는가를 먼저 생각하기

## Spring Batch

- 많은 오픈소스가 웹기반 프레임워크에 집중
- 재사용 가능한 표준 배치 아키텍처가 없었음
- Accenture의 일괄 처리 아키텍처와 스피링 프로그래밍 모델을 통합하여 배치 프레임워크 개발
- 개발자는 비지니스 로직에 집중하고 일괄 처리 인프라는 프레임워크에서 관리

## Domain Lang귀지

### Job

- Setp container
	- step구성
	- 재시작 여부
- Job instance
	- Job + Job parameter
- Job execution	
	- Job instance의 실행을 기술
	- 실행결과, 시간등의 데이터
	- 재시작등으로 1:n 관계가능

### Step

- 독립적인 실행 단계
- 배치처리를 정의하고 제어하는데 필요한 모든 정보를 포함
- Job은 하나 이상의 Step으로 구성
- ItemReader
- ItemProcessor
- ItemWriter

개발자가 실제로 개발하는 부분
Tip은 복잡한 step을 만들때 간단한.step을 여러개 만들어서 만드는게 낫다 
배치를 만들때 어떻게 잘 나눌까?를 고민하기

### ExecutionContext

- Key-Value Store
- SpringBatch 프레임워크에서 자동으로 영속성 관리
- JobScope VS StepScope

저장소 개념으로 사용

### Persistent MetaData

- JobRepository
	- JobInstance, JobExecution, StepExcution 등의 CRUD
- JobExplorer
	- 읽기전용 
- JobOperator= JobRepository + JobExplorer

Job의 영속성관리 할때 사용 DAO라고 보면 됨


## Job

### JobRepository

- DB이용
	- Table Prefix 설정가능
	- 지원하는 DB타입이 정해져있음
	- 지원하지 않는 dB타입을 사용하는 경우 (CUBRID)
		- JobRepositoryFactoryBean 이용
		- 호환되는 적당한 타입을 강제로 설정
- DB를 이용하지 않는 경우
	- SimpleJobRepository 안의 Dao들을 구현해서 사용
	- MapJobRepository (jenkins사용하는 경우)
		- 인메모리 저장소 = 휘발성
		- ResourcelessTransactionmanager 사용해야함

Job meta데이터를 어디에 저장할 것인가를 결정하는 인터페이스임

### Job Launcher

- job을 실행하는 인터페이스
- 기본제약사항
	- jobExcution이 생성되었으면 Job실행결과와 상관없이 Execution을 리턴함
	- 이전에 정지된 Execution이 있었으면 새로 만들지 않고 그걸 실행
	- Exception은 Job실행 시작과정에서 에러가 발생한 경우만 발생
		- jobInstance가 이미 완료되었거나, 실행중인 경우
		- Parameer가 Null이거나
		- 파라미터 validation이 실패한 경우
	- 그외는 Execution을 리턴하고 에러발생 여부는 상태를 보고 판단
- 기본구현체 SimplejobLauncher
- Async 실행 - ThreadPoolTaskExecutor주입	

### JobRegistry

- Job 객체에 편하게 접근할 수 있다.
	- SpringBatch의 Jobinstance 개념이 아닌 Java객체
	- Application context에서 bean받아 올 수도 있다. (beanName)
	- job registry는 job name기반
	- mapJobRegistry사용 (이거 bean등록)
- JobRegistryBeanPostProcessor로 자동 등록 (이거 bean등록)

job이름을 가져올때 사용 

### JobOperator

- 모니터링에 유용한 기능들을 제공
	- 실행중 job목록 가져오기
	- 실행, 다음실행, 재실행, 정지, 실행요약, job이름목록
- 기본구현체 SimpleJobOperator
	- jobRepository, Jobexplorer, job.. bean등록 되어 있어야지 사용가능


## Step

### Step Restart
- 같은 job instance에서, 한번 성공한 Step은 skip
- allow-start-if-complete
	- 성공여부 상관없이 항상 실행
- start-limit
	- 스텝을 싱핼 수 있는 횟수 제한

### Chunk-Oriented

- 하나의 데이터 -> item
- item을 모아서 chunks
- 쓰기는 chunks 단위로
- Transaction boundary
- chunk크기는 Step에설정
	- commit-interval
	- .chunk()

### Skip Exception

- Exception발생시 실패로 만들지 않고 해당 item만 skip
	- Step skip과는 다름

demo5를 보면 batch의 트렌젝션을 볼 수 있음

주의할점 wirte에서 error나면 chunk가 1로 변경되기 때문에 느려질수도 잇으니 적절하게 chunck크기를 조절하자 모니터링잘하구

### TaskletStep

- read-process-wirte 형태가 아닌 작업인경우
	- 디비프로시저 실행
	- 스크립트 실행
	- 단순 sql업뎃
- RepeatStatus
	- 반환값이 FINISIHED일때 까지 반복 실행됨
	- 각 반복마다 트랜잭션 생성	

	
	
## Partition

- partition을 나누면 read도 멀티쓰레드도 가능