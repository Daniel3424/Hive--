# CBO
하이브 0.14 버전부터 사용자의 쿼리를 최적화하는 CBO를 지원합니다. CBO가 적용되면 사용자의 쿼리를 분석해서 쿼리를 최적화 합니다. 예를 들어 테이블 A, B, 의 조인을 처리할 때 조인 데이터를 비교하여 셔플 단계의 데이터를 줄일 수 있는 방향으로 쿼리를 수정하여 줍니다. 다음과 같은 쿼리에서 t1, t2를 모두 읽어서 조인하지 않고, t2를 읽을 때 먼저 필터링을 처리한뒤 조인하는 방향으로 쿼리를 최적화 하여 줍니다. t2의 데이터가 많을 경우 네트워크를 통해 이동하는 데이터가 줄어들어서 처리 비용이 감소합니다.
```
SELECT sum(v)
  FROM (
    SELECT t1.id,
           t1.value AS v
      FROM t1 JOIN t2
     WHERE t1.id = t2.id
       AND t2.id > 50000) inner
  ) outer
  ```
  
  ![CBO](https://wikidocs.net/images/page/33619/CBO.png)
  
  
  # CBO Option
CBO를 적용하기 위해서는 다음의 옵션을 설정해야 합니다. 기본적으로 true 상태입니다.
```
-- CBO 적용
set hive.cbo.enable=true;
-- 새로 생성되는 테이블과 INSERT 처리를 할 때 자동으로 통계정보 수집 
set hive.stats.autogather=true;
set hive.stats.fetch.column.stats=true;
set hive.stats.fetch.partition.stats=true;
```

통계정보를 자동으로 수집하지 않으면 `ANALYZE`명령을 이용해서 수동으로 정보를 수집해야 합니다. 통계정보 수집도 맵리듀스 작업이기 때문에 데이터가 많으면 시간이 오래 걸릴 수 있습니다.
```
ANALYZE TABLE sample_table PARTITION(yymmdd='20180201') COMPUTE STATISTICS for columns;
ANALYZE TABLE sample_table PARTITION(yymmdd='20180201') COMPUTE STATISTICS;  
```
# explain 쿼리 확인
`explain` 명령으로 CBO 적용 여부를 확인할 수 있습니다. CBO가 작용되면 다음과 같이 "Plan optimized by CBO." 라는 메시지가 출력됩니다.
```
hive> explain INSERT OVERWRITE DIRECTORY 'hdfs:///user/data/location'
    > select name, count(1)
    >   from sample_table
    >  where yymmdd=20180201
    >  group by name
    >    
    > ;
OK
Plan optimized by CBO.
```

# CBO 적용 불가
CBO가 적용 불가능한 상황이 몇가지 있습니다. CBO 적용이 불가능한 경우 로그에 불가능한 원인을 다음과 같이 출력합니다. 따라서 "Plan not optimized by CBO." 메시지가 출력되면 하이브 로그를 확인하여 적용 불가 원인을 확인하고 수정하면 됩니다.
```
2019-04-05T08:08:12,490 INFO  [main([])]: parse.BaseSemanticAnalyzer (:()) - Not invoking CBO because the statement has sort by
```

# 적용 불가 상황
CBO 적용이 불가능한 몇가지 상황은 다음과 같습니다.

 - 트랜스폼(transform)은 사용 불가
 - 인라인 Lateral View Join 만 가능
 - UNIQUE 조인은 불가
 - 서브쿼리 사용 불가
 - Having 절에 select의 alias 가 들어있으면 사용 불가
 - Sort By 사용 불가
  
  
  
  
  
  
  
  
  
  
  
