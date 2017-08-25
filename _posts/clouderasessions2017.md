# Cloudera Sessions 2017
 
## 1. 환영사 - 강형준 지사장
 
하둡이 10년 넘게 준비 해서 전문적이다.
600명 넘게 왔음..
cloudera는 si가 아니고 빅데이터를 통해 목적을 달성하게 도와주는 역할을 함!
올해 상장해서 public업체가 되었음

## 2. clouder 애코시스템

올해 상장해서 이제 사업 많이 할꺼야!
파트너참가를 쉽게 했음
cloudera solutions gallery를 만들었음 
파트너되면 교육도 해줌
cludera관련 소프트웨어를 개발하면 gallery에 올려서 공유하자

## 3. 머신러닝 AI 빅데이터분석의 미래 (chief marketing officer)

#### 올해 뉴욕증권거래소에 상장됨 cloudera가 무슨 일을 하는가? 

- why -> data can make what is impossible today, possible tomorrow
- how -> we empower people to transform omplex data into clear and actionable insights
- what -> we deliver the modern platform for machine learning and advanced analytics

cloudera는 잘나감 -> 데이터는 폭증하고 오픈소스를 사용해서 밴더의 종속적이지 않음 그리고 머신러닝을 지원함

예) 
- bt통신사 cloudera를 사용해서 이용자의 패턴을 파악하여 고객이탈을 방지하고 있음
- nveistar회사는 차량의 센서를 장착하여 데이터를 분석해서 유지보수 비용감소
- mastercard 고객정보를 보호함

데이터회사의 5가이 패턴

1. Build a data-driven culture
2. Develop the right team and skils
3. Be agile/lean in development
4. Leverage DevOps for production
5. Right-size data governance

Data-driven business 순서 visibilytiy ->ddd

전세계 잘난 회사들은 clouera 고객임

신제품 나옴 -> Cloudera Altus : 머신러닝 쉽게해주는 툴임

특징

- Elastic
- Portable
- Enterprise-grade

지금 26개의 오픈소스를 받아드림

1000개 cloudera의 고객사중 650개 회사가 spark를 사용하는데, 이는 머신러닝을 위해 사용된다.

cloudera 사용목적은 크게 3개 

1. 데이터 통찰력
2. product 비용 절감
3. 데이터 보안


대부분의 회사가 데이터를 활용을 잘 못하고 데이터 권한을 잘못 부여하고 대부분 데이터 가공에 시간을 보냄 근데 클라우데라가 이를 잘하게 도와줌


## 4. Data Science and Machine Learning in the Enterpirse (cloudera CTO)

현재는 자동화의 제 6의 물결을 지나고 있음

- 제 1물결 - 언어로 내용을 전달하는 것 10만년전
- 제 2물결 - 농업의 혁명 (수렵을 안해도 되고 먹는거 생산의 자동화) 1만년
- 제 3물결 - 수학과 과학의 원칙을 발견 (방법의 자동화) 3천년전에 생김
- 제 4물결 - 산업혁명 (제조의 자동화) 200년전
- 제 5물결 - it의 물결 (프로세스의 자동화) 80년전
- 제 6물결 - 의사결정의 자동화 물결 (예 의료 진단 (패턴을 매칭해서 어떤 병의 걸렸는지 의사결정 이런것을 머신러닝을 통해 자동화를 한다)10년전

머신러닝으로 잘하는 것

1. pattern recognition
2. prediction
3. detection (비정상적인 것을 찾음 예-고객이탈,오류검출)

cloudera는 다양한 것을 할 수 있다.

spark가 급속도로 성장하고 있다. 의사결정의 자동화를 도와준다. 즉 머신러닝에 도움이 된다.

상위 1000개의 기업중에서 71%기업이 데이터 사이언스를 잘하기 위해서 spark 사용한다.


cloudera data scienteis workbanch -> local container를 지원해서 알고리즘 개발이 가능함

cloudera가 가장 중요시하는 것은 사이버 보안이다. dell usecase가 있음 - 비정상적인 패턴을 찾아낸다.

## 5. ok cashbag

- **보안성 측면** : 개인정보를 삭제하기 위해서는 하둡에 적제할 때 암호화하여 적재하고 hive 복호화udf를 사용하여 찾음
- **정보 품질 및 신뢰성** : hive에 Data Quality Management System을 적용함 full dump를 하여 데이터를 적재함(데이터 정합성을 위해) 테이블 복잡도가 높이지면 udf를 사용하면 성능향상에 도움됨
- **정보 접근성 및 생산성**

#6 Cloudera Data Science Workbranch

데이터사이언스가 사용하는 툴

- Data analyst - 데이터 정재 및 sql 짜기
- Data scientist - 해비코더
- Data Engineer - 가치있는 데이터를 더 가치있게 만듬 모델링이 완료 된 데이터를 packaging
- 인프라담당자(IT)

전통적인 workflow
it -> da -> ds -> de

cloudera Data Science Workbranch(CDSW)는 인프라 담장자와 데이터 분석하는 그룹 간에 갈등을 해소한다.

docker 위에서 동작 하고 admin에서 리소스관리를 할 수 있다. 권한 제한도 가능하다. 

-> 커스텀 타겟팅을 하게 된다면 새로운 타겟팅을 개발 하기위한 실험실 같은 역할을 하면 좋을 듯 아직 판매는 안하고 곧 출시할 예정이다. 


## 6. Hadoop 3.0 (제길 동시통역 지원안함..ㅠㅠ)

하둡 3.0개발동기
- java 7 지원 중지 java 8skdha
- HDFS erasure coding
- classpath isolation
- Other micellaneous incompatible bugfix

17년 4분기에 릴리즈할 계획임

기존 하둡은 3카피라서 200% overhead가 있다.

3.0에서는 2partiy bolck으로 만들어서 67% overhead로 줄였다.

[http://kysepark.blogspot.kr/2016/10/hadoop-30_5.html](http://kysepark.blogspot.kr/2016/10/hadoop-30_5.html)

yarn에 Timeline Service v2에서 job의 리소스관리 로그를 볼수 있음
-> 현재 sparkstremming 등 spark job에 대해서 history관리가 안되고 있는데, 이 기능을 사용하면 관리가 쉬울꺼 같음..

yarn resource manager UI 완전 개선 nodejs, emberjs를 사용해서 비동기로 실시간으로 job동작을 업뎃을 하도록 개선 했음!

Resource Type을 제공해서 job을 submmit할때 memory와 CPU관리 더 잘 할 수 있게함

YARN Federation을 적용 부서별로 Resource Manager를 가져 갈 수 있게 함
-> 전사적으로 하둡 클러스터를 통합할 때 이런 방식으로 가져가면 효율적으로 리소스 관리를 할 수 있을 것 같음

#### Other yarn improvemnets

- Long Running Services 
	- Docker suppert (머신러닝 환경 만들기 좋음)
- Scheduler improvements
	- Capacity scheduler
		- online scheduling
		- Queue management
	- Fair scheduler
		- Performance and preemption improvements
- High availabilty improvements
- MappReduce Native Collerctor (성능 30% up)


하둡2 -> 하둡3가는데 많이 도와줄꺼임 이미 일본 야후에서 하둡3를 사용하고 있음

## 7. spark bootcamp

[https://www.slideshare.net/sangbaelim](https://www.slideshare.net/sangbaelim)
 
		
## 8. Mllib 

1. 홈페이지 들어가면 두개 페키지가 있음 ( mllib(RDD기반 알고리즘이 더많음 3.0에서 삭제 예정) vs ml(Dataframe기반 RDD보다 빠름) spark로 머신러닝을 도입한다면 ml페키지로 사용해야함!!

n-gram을 사용하면 문서에서 불필요한 문자 제거 가능(mllib라이브러이에 있음) 문맥타겟팅에 도움될듯~~^^


modelDB -  spark piepeline를 사용하여 하이퍼파라미터, 모델, 리콜,프리시전 같은 것을 디비로 관리
autoML - 하이퍼파라미터를 자동으로 튜닝해준다. 아직까지는 spark에서 제공은 안함

텐서플로onSpark 모델들을 분산처리해서 계산할 수 있게 해주는 프레임웍

DL4J

## 8. spark 2.2 (7월 12일 릴리즈)

cost-based optimizer 기능이 추가 되었음 근뎅 아직 불안정함..
Schema in a DDL -formating
Multiline supperot

2.1까지는 dataset보다 dataframe이 더 빠름 (실제 돌아가는 코드는 dataset에서는 boxing됨)

spark2.0은 hive dependency를 제거했다.

spakr2.0이상 부터는 hive에 비해 쓰기는 편하지만 읽는데 오래걸림 (bucketing방식의차이 (hive에서는 bucketing당 파일하나)






### 발표자료 자료
[https://drive.google.com/drive/folders/0B6vz9vaqSFqAczllS3JHMGl2S0k](https://drive.google.com/drive/folders/0B6vz9vaqSFqAczllS3JHMGl2S0k)
