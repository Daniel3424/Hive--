# ORC
ORC(Optimized Row Columnar)는 칼럼 기반의 파일 저장방식으로, Hadoop, Hive, Pig, Spark등에 적용가능합니다. 데이터베이스의 데이터를 읽을 때 보통 칼럼단위의 데이터를 이용하는 경우가 많습니다. ORC는 칼럼단위로 데이터를 저장하기 때문에 칼럼의 데이터를 찾는 것이 빠르고, 압축효율이 좋습니다.

![ORC](https://image.slidesharecdn.com/w-1205p-230a-radhakrishnanv3-140617155040-phpapp02/95/hive-and-apache-tez-benchmarked-at-yahoo-scale-21-638.jpg?cb=1403020414)

# ORC 설정
하이브에서는 다음과 같이 STORED AS를 ORC로 선언하고, TBLPROPERTIES에 설정정보를 입력합니다.
```
CREATE TABLE table1 (
    col1 string,
    col2 string
) STORED AS ORC
TBLPROPERTIES (
    "orc.compress"="ZLIB",  
    "orc.compress.size"="262144",
    "orc.create.index"="true",
    "orc.stripe.size"="268435456",
    "orc.row.index.stride"="3000",
    "orc.bloom.filter.columns"="col1,col2"
);
```
# ORC 설정 값
 - orc.compress
   - 기본값: ZLIB
   - 압축방식 설정 (one of NONE, ZLIB, SNAPPY)
 - orc.compress.size
   - 기본값: 262,144
   - 압축을 처리할 청크 사이즈 설정(256 * 1024 = 262,144)
 - orc.create.index
   - 기본값: true
   - 인덱스 사용 여부
 - orc.row.index.stride
   - 기본값: 10,000
   - 설정 row 이상일 때 인덱스 생성 (must be >= 1000)
 - orc.stripe.size
   - 기본값: 67,108,864
   - 스트라이프를 생성할 사이즈 (64 * 1024 *1024 = 67,108,864)), 설정 사이즈마다 하나씩 생성
 - orc.bloom.filter.columns
   - 기본값: ""
   - 블룸필터1를 생성할 컬럼 정보, 콤마(,)로 구분하여 입력
 - orc.bloom.filter.fpp
   - 기본값: 0.05
   - 블룸필터의 오판 확률(fpp=false positive portability) 설정 (must >0.0 and <1.0)

