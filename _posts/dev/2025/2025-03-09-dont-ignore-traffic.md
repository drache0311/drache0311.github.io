---
title: "Don't ignore Traffic"
search: false
tags: [high-traffic, multi-thread, async]
last_modified_at: 2025-03-09T22:54:00
---
ELD 를 개발하면서 영업점의 대량 거래들을 받을 때의 속도 이슈, 거래 처리 후 BackOffice 로 전달할 때 해당 시스템의 처리한계 이슈가 발생했습니다.

어느정도의 트래픽을 고려하여 준비했었지만 예상을 뛰어넘는 예외상황에 `High-Traffic` 최적화의 필요성이 생겼습니다

하지만 이는 트래픽 자체의 문제라기보다는, 트래픽 처리가 결코 쉬운 일이 아니기 때문입니다.

이 글은 "Handling Traffic" 이라는 시리즈의 첫 번째 글로, traffic 을 처리하는 일반적인 상황 중에서 더 나은 해결 방법을 찾아가는 사례들을 다룰 예정입니다.

## What is "High-traffic"?
결국 핵심은 대규모 트래픽이 무엇인지 이해하는 것, 더 정확히 말하면 무엇이 대규모 트래픽이 아닌지 이해하는 것이라고 생각합니다. 
이를 이해하기 위해서는 해당 시스템의 아키텍처 설계, 데이터베이스 최적화, 캐싱 전략, 부하 분산 등을 파악하는 것이 가장 좋은 방법입니다.
더불어 `대규모` 란 상대적이기 때문에, 이를 특정짓기 보다는 서비스가 무엇을 하는 지 파악하고 해당 트래픽이 어느정도의 볼륨부터 서비스의 기능을 방해하는지로 생각해보는 것입니다. 

<p align="center">
    <img src="../../static/images/2025/03-09/bottleneck.png" alt="bottleneck"/><br/>
    <em>Is there a difference between these two traffics?</em>
</p>

여기 테스트를 위한 컨테이너를 준비했습니다.<br>
> Java 17, SpringBoot 4.3.2, MySQL <br>
> cpu : 2-core <br>
> mem : 2GB

이 컨테이너의 스펙을 기준으로 어느정도의 수준부터 대규모 트래픽인지, 그렇다면 어떤 방식이 효과적인지 알아가보겠습니다.

## An example
```java
    // Controller
    @PostMapping("/traffic/each")
    public void makeTrafficAtEachTransaction(@RequestBody List<SampleData> hugeDataList) {
        trafficService.handlingTrafficAtEachTransaction(hugeDataList);
    }

    // Service
    public void handlingTrafficAtEachTransaction(List<SampleData> sampleDataList) {
        sampleDataList.stream().map(SampleData::toEntity).forEach(trafficStore::createData);
    }
    
    // Store
    @Transactional
    public void createData(TrafficEntity trafficEntity) {
        entityManager.persist(trafficEntity);
    }
```

Store에는 단일 객체를 단일 트랜잭션으로 insert 하는 메소드가 있고, API 요청을 받아 데이터베이스에 10만건을 insert 하는 요구사항을 가정해보겠습니다.
이 서비스 로직 처리로써는, API 하나의 요청에 평균적으로 cpu 90% mem 300mb 가량 사용하며 30m 정도의 소요시간이 걸립니다.
이에 엔지니어는 이 서비스가 죽을수도 있는 불안정한 서비스로 생각해, 멀티-쓰레드, 캐싱, 트랜잭션 등의 방법들로 비교하여 최적의 방법을 찾아보고자 합니다.

### Each-Transaction
먼저 기존 서비스를 분석해보겠습니다.
```java
    // make 100,000 transaction, it takes 30m
    public void handlingTrafficAtEachTransaction(List<SampleData> sampleDataList) {
        sampleDataList.stream().map(SampleData::toEntity).forEach(trafficStore::createData);
    }
```
Store 에서는 단일 트랜잭션에서 단 건의 객체를 받아 insert 를 하고 있기에, jpa의 정책에 따라 트랜잭션 commit 시점마다 DB I/O 이 생겨 총 10만건의 insert 쿼리를 발생시킵니다.
이는 각각의 트랜잭션 마다 데이터베이스 커넥션이 생겨, cpu 의 부하가 생기며, 소요 시간을 증가시킵니다. 또한 처리 도중에 다른 서비스의 데이터베이스 접근 요청이 오면, 그 요청을 먼저 처리하기에
순서의 보장이 중요하거나, 데이터의 정합성이 중요하다면 위험할 수 있습니다.

### ONE-Transaction
```java
    // make 1 transaction, it takes 25s
    @Transactional
    public void handlingTrafficAtOneTransaction(List<SampleData> sampleDataList) {
        sampleDataList.stream().map(SampleData::toEntity).forEach(trafficStore::createData);
    }
```
단순하게 10만건의 쿼리 요청에 대한 트랜잭션을 서비스로 전파해 처리해보면 어떨까요?<br/>
엔티티 매니저의 모든 insert 쿼리는 쓰기지연되어 단 한 건의 데이터베이스 커넥션으로 처리되게 변경됩니다. 이에 cpu 20% mem 3mb 와 25s 로 단축되었습니다.
이전 방법의 문제였던 데이터의 순서, 정합성 등이 보완이 중요했다면 좋은 방법이 될 수 있어 보입니다.

### Parallel
```java
    // what about parallelStream?
    @Transactional
    public void handlingTrafficAtParallel(List<SampleData> sampleDataList) {
        sampleDataList.parallelStream().map(SampleData::toEntity).forEach(trafficStore::createData);
    }
```
다른 방법은 없을까요?<br/>
같은 작업을 `parallelStream`으로 병렬로 실행해 보는건 어떨까요?<br/>
제 생각은 'Maybe Not' 입니다.<br/>
parallelStream 은 ForkJoinPool 의 쓰레드로써 생성되어 병렬로 돌기에, 각 트랜잭션이 쓰레드별로 생겨 트랜잭션의 전파가 제대로 이루어지지 않습니다. 
이것은 각 쓰레드 별로 새로운 데이터베이스 커넥션이 생긴다는 말이기도 합니다. 쓰레드를 사용할 때는 데이터베이스 커넥션도 생각해야할 것입니다.
그리고 `병렬`로 돈다는 것은 `비동기`로 도는것과는 차이가 있습니다. parallelStream 은 동기적으로 돌기에 이 작업이 끝나기 전 까지는 blocking 상태가 됩니다.<br/>
시간은 45s 로 30m 보다는 많이 단축되었지만 쓰레드 별 트랜잭션에 의해 순서, 정합성 등은 이루어지지 않습니다.


### Async
```java
    // what about async(multi-thread)?
    public void handlingTrafficAtAsync(List<SampleData> sampleDataList) {
        int BATCH_SIZE = 10000;
    
        for (int i = 0; i < sampleDataList.size(); i += BATCH_SIZE) {
            int end = Math.min(i + BATCH_SIZE, sampleDataList.size());
            List<SampleData> batch = sampleDataList.subList(i, end);
            asyncService.handlingTrafficAtAsync(batch);
        }
    }

    // async-service
    @Transactional
    @Async
    public void handlingTrafficAtAsync(List<SampleData> sampleDataList) {
        sampleDataList.stream().map(SampleData::toEntity).forEach(trafficStore::createData);
    }
```
이번에는 비동기 방식을 이용한 멀티 쓰레드로 각 1만건 단위로 실행해보았습니다. 평균 시간은 10s 로 가장 짧은 시간이 소요되었습니다.
짧은 소요시간이 필요하다거나, non-blocking 이 필요하다면 좋은 방법일 것입니다. 하지만 위에서 말했던 것처럼 각 쓰레드 별 트랜잭션이 전파가 되지 않기에,
한 쓰레드에서 실패가 발생해도 모든 작업에 대해 롤백이 되지는 않습니다.<br/>
실패한 쓰레드를 별도로 추적, 관리하여 작업 이후 처리한다거나 중요성이 떨어지는 작업이라면 비동기 방법이 좋은 선택이 될 것 같습니다.
여기서 주의할 점은, 쓰레드 풀이 가득 찼을 때의 핸들링이나 쓰레드의 에러 발생 시 핸들링 같은 점을 고려해두어야 한다는 점입니다.
이것은 [Thread] 에 관한 글에서 다시 한번 다루겠습니다.


