---
layout: post
title: Parquet Write 작업시 주의할 점
date: 2020-07-28 13:10
categories: BigData
---
# Parquet rewriter만들기 작업

회사에서 parquet로 적재된 데이터를 특정row의 특정컬럼 데이터를 Null로 update하는 작업을 수행하던 중 발견한 이슈이다.
> impala로 insert overwirte하면, 불필요한 file I/O가 발생하여 직접 parquet rewriter를 만들어 rowGroup단위로 필요한 부분을 update하는 엔진을 만들었다.
> 여기서 최적화를 한번더하면, rowGroup보다 더 작은 단위인 page단위로 update를 하는 처리를 할 수 있는데, rowGroup단위로 해도 충분히 성능이 나와 stop했음
> 해당 엔진을 만들 때 [참고한 소스](https://github.com/Factual/parquet-rewriter))

Java를 이용해서 Hadoop Parquet read/write를 하기 위해서 많이 사용되는 것이 avro-parquet라이브러리 일 것이다.
하지만 이 라이브러리에서는 `INT96` type (Decimal, timestamp)를 지원하지 않는다. (parquet-format은 INT96타입을 지원하지 않음 [참고](https://github.com/apache/parquet-format/blob/master/src/main/java/org/apache/parquet/format/LogicalTypes.java))

> Decimal타입을 사용하지 않으면, 굳이 해당 내용을 생각할 필요가 없다.

```java
    PrimitiveType asPrimitive = parquetType.asPrimitiveType();
    PrimitiveTypeName parquetPrimitiveTypeName = asPrimitive.getPrimitiveTypeName();
    final OriginalType annotation = parquetType.getOriginalType();
    Schema schema = (Schema)parquetPrimitiveTypeName.convert(new PrimitiveTypeNameConverter<Schema, RuntimeException>() {
    public Schema convertBOOLEAN(PrimitiveTypeName primitiveTypeName) {
        return Schema.create(Type.BOOLEAN);
    }

    public Schema convertINT32(PrimitiveTypeName primitiveTypeName) {
        return Schema.create(Type.INT);
    }

    public Schema convertINT64(PrimitiveTypeName primitiveTypeName) {
        return Schema.create(Type.LONG);
    }

    public Schema convertINT96(PrimitiveTypeName primitiveTypeName) {
        throw new IllegalArgumentException("INT96 not yet implemented.");
    }

    public Schema convertFLOAT(PrimitiveTypeName primitiveTypeName) {
        return Schema.create(Type.FLOAT);
    }

    public Schema convertDOUBLE(PrimitiveTypeName primitiveTypeName) {
        return Schema.create(Type.DOUBLE);
    }

    public Schema convertFIXED_LEN_BYTE_ARRAY(PrimitiveTypeName primitiveTypeName) {
        int size = parquetType.asPrimitiveType().getTypeLength();
        return Schema.createFixed(parquetType.getName(), (String)null, (String)null, size);
    }

    public Schema convertBINARY(PrimitiveTypeName primitiveTypeName) {
        return annotation != OriginalType.UTF8 && annotation != OriginalType.ENUM ? Schema.create(Type.BYTES) : Schema.create(Type.STRING);
    }
```

위 코드는 avro-parquet 내부소스인데, `INT96` type에 대해서 컨버터를 만들어 주지 않아. 

```
java.lang.IllegalArgumentException: INT96 not yet implemented
```

Exception을 표시한다.


**Parquet-writer를 만들때 이슈를 정리하면 다음과 같다.**

## 배경지식
- Parquet에서 INT96을 지원하지 않는다.
- INT96은 우리가 일반적으로 아는 Decimal, timestamp type이 저장되는 데이터저장공간이다.
- Kudu에서는 decimal type이 지원함

## 이슈
- kudu talbe을 impala을 이용해 parquet table만들 때, decimal type을 INT96으로 저장하고 읽을때 메타스토어를 refer하여 정상적으로 읽음.(HUE, Impala사용자는 문제없음)
- spark에서는 Parquet read시 INT96을 무조건 timestamp로 치환해 읽음(Spark사용자들은 decimal type값을 정상적으로 읽을 수 없음)
- parquet-rewriter작업만들때 INT96타입을 처리하는 별도 로직이 필요하고, 구성원들의 합의가 필요함.

## 결정이 필요한 내용
- parquet-rewriter작업은 INT96타입을 어떻게 처리하면 좋을까요? 

### Reference

#### parquet공식
- https://issues.apache.org/jira/browse/PARQUET-323

#### parquet소스
- [https://github.com/apache/parquet-mr/blob/master/parquet-avro/src/main/java/org/apache/parquet/avro/AvroSchemaConverter.java#L279](https://github.com/apache/parquet-mr/blob/master/parquet-avro/src/main/java/org/apache/parquet/avro/AvroSchemaConverter.java#L279)

#### Spark소스
- [https://github.com/apache/spark/blob/a988aaf3fa38e89a05ab644ce79ddb14af56859d/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/parquet/ParquetSchemaConverter.scala#L154](https://github.com/apache/spark/blob/a988aaf3fa38e89a05ab644ce79ddb14af56859d/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/parquet/ParquetSchemaConverter.scala#L154)

```
Sets which Parquet timestamp type to use when Spark writes data to Parquet files.
INT96 is a non-standard but commonly used timestamp type in Parquet.
TIMESTAMP_MICROS is a standard timestamp type in Parquet, which stores number of microseconds from the Unix epoch.
TIMESTAMP_MILLIS is also standard, but with millisecond precision, which means Spark has to truncate the microsecond portion of its timestamp value.
```


#### converter관련 소스
- [https://stackoverflow.com/questions/54657496/how-to-write-timestamp-logical-type-int96-to-parquet-using-parquetwriter](https://stackoverflow.com/questions/54657496/how-to-write-timestamp-logical-type-int96-to-parquet-using-parquetwriter)