# TEZ
테즈(TEZ)는 YARN 기반의 비동기 사이클 그래프 프레임워크입니다. 하이브에서 맵리듀스 대신 실행엔진으로 사용할 수 있습니다. 맵리듀스는 맵단계에서 데이터를 읽어서 처리하고, 리듀스 단계에서 처리 결과를 저장합니다. 하나의 작업이 여러 단계의 맵리듀스를 거치게 되면 중간 작업 결과를 HDFS에 쓰고, 다시 맵단계에서 파일을 읽어서 처리하게 됩니다. 작업 중간 임시 데이터도 디스크에 쓰게 되어 IO 작업으로 인한 오버헤드가 많았습니다.

테즈는 맵단계 처리 결과를 메모리에 저장하고, 이를 리듀스 단계로 바로 전달합니다. 리듀스 작업의 결과를 맵단계를 거치지 않고 리듀스 단계로 전달하여 IO 오버헤드를 줄여서 속도를 높일수 있습니다.

![tez](https://wikidocs.net/images/page/23569/tez.png)

# MapReduce vs. TEZ
[AWS EMR을 이용한 테스트](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/tez-using.html)에서 동일한 작업에서 맵리듀스는 약 50초, 테즈는 30초로 40% 정도 성능이 향상된것을 확인할 수 있습니다.

## MapReduce time: 49.699 sec
```
Starting Job = job_1464200677872_0002, Tracking URL = http://ec2-host:20888/proxy/application_1464200677872_0002/
Kill Command = /usr/lib/hadoop/bin/hadoop job  -kill job_1464200677872_0002
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2016-05-27 04:53:11,258 Stage-1 map = 0%,  reduce = 0%
2016-05-27 04:53:25,820 Stage-1 map = 13%,  reduce = 0%, Cumulative CPU 10.45 sec
2016-05-27 04:53:32,034 Stage-1 map = 33%,  reduce = 0%, Cumulative CPU 16.06 sec
2016-05-27 04:53:35,139 Stage-1 map = 40%,  reduce = 0%, Cumulative CPU 18.9 sec
2016-05-27 04:53:37,211 Stage-1 map = 53%,  reduce = 0%, Cumulative CPU 21.6 sec
2016-05-27 04:53:41,371 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 25.08 sec
2016-05-27 04:53:49,675 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 29.93 sec
MapReduce Total cumulative CPU time: 29 seconds 930 msec
Ended Job = job_1464200677872_0002
Moving data to: s3://myBucket/mr-test/os_requests
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 29.93 sec   HDFS Read: 599 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 29 seconds 930 msec
OK
Time taken: 49.699 seconds
```

## TEZ time: 30.711 sec

```
Time taken: 0.517 seconds
Query ID = hadoop_20160527050505_dcdc075f-8338-4041-adc3-d2ffe69dfcdd
Total jobs = 1
Launching Job 1 out of 1


Status: Running (Executing on YARN cluster with App id application_1464200677872_0003)

--------------------------------------------------------------------------------
        VERTICES      STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED
--------------------------------------------------------------------------------
Map 1 ..........   SUCCEEDED      1          1        0        0       0       0
Reducer 2 ......   SUCCEEDED      1          1        0        0       0       0
--------------------------------------------------------------------------------
VERTICES: 02/02  [==========================>>] 100%  ELAPSED TIME: 27.61 s    
--------------------------------------------------------------------------------
Moving data to: s3://myBucket/tez-test/os_requests
OK
Time taken: 30.711 seconds
```
# 실행엔진 설정
하이브에서 테즈 엔진을 사용하는 방법은 다음과 같습니다.
```
-- 테즈 엔진 설정 
set hive.execution.engine=tez;
set tez.queue.name=tez_queue_name;

-- 맵리듀스 엔진 설정 
set hive.execution.engine=mr;
set mapred.job.queue.name=mr_queue_name;
```
