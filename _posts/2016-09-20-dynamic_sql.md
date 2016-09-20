---
layout: post
title: mybatis 동적 SQL
date: 2016-09-20 12:30
categories: web
---

# 동적 SQL

mybatis는 동적 SQL을 위한 엘리먼트를 제공한다.

<br/>

**동적 SQL을 작성할 때 사용하는 엘리먼트**

| 엘리먼트 예 | 설명 |
|-------------|----------|
| \<if test="조건">SQL문\</if> | `<if>`태그는 어떤 값을 상태를 검사하여 참일 경우에만 SQL문을 포함하고 싶을 때 사용한다. test속성에 지정된 조건이 참이면 `<if>`태그의 내용을 반환한다. |
| \<choose> <br/>   \<when test="조건1">SQL문\</when> <br/> \<when test="조건2">SQL문\</when> <br/> \<otherwise>SQL문 \</otherwise> \</choose> | `<choose>`태그는 검사할 조건이 여러 개일 경우에 사용한다. test속성에 지정된 조건이 참이면 `<when>`태그의 내용을 반환한다. 일치하는 조건이 없으면 `<otherwise>`의 내용을 반환한다. |
| \<where> <br> \<if test ="조건1>SQL문\</when> <br/> \<if test="조건2>SQL문\</when> <br/> \</where> | `<where>`태그는 WHERE절을 반환한다. `<where>` 안의 하위 태그를 실행하고 나서 반환값이 있으면 WHERE절을 만들어 반환하고, 없으면 WHERE절을 반환하지 않는다. |
| \<trim prefix="단어" prefixOverrides="문자열\|문자열"> <br/> \<if test="조건1">SQL문\</when> <br/> \<if test="조건2">SQL문\</when> <br/> \</trim> | `<trim>`태그는 특정 단어로 시작하는 SQL문을 반환하고 싶을 때 사용한다. prefix는 반환값 앞에 붙일 접두어를 지정한다. prefixOverrides는 반환할 값에서 제거해야 하는 접두어를 지정한다. <br/>  다시 말하면 `<trim>`의 반환값이 있다면, 그 값의 앞부분이 prefixOverride에 지정된 문자열과 일치할 경우 그 문자열을 제거한다. 그리고 그 값의 앞부분에 prefix로 지정한 접두어를 붙여 반환한다. |
| \<set> <br/> \<if test="조건1">SQL문\</when> <br/> \<if test="조건2">SQL문\</when> <br/> \<set> | `<set>`태그는 UPDATE문의 SET절을 만들 때 사용한다. 조건이 참인 `<if>`의 내용은 SET절에 포함된다. SET 절의 항목이 여러개 일 경우 자동으로 콤마(,)를 붙인다. |
| \<foreach <br/> item="항목"<br/> index="인덱스"<br/>collection="목록"<br/>open="시작문자열"<br/>close="종료문자열"<br/>separator="구분자"><br/> \</foreach> | `<foreach>`태그는 목록의 값을 가지고 SQL문을 만들 때 사용한다. 특히 IN(값, 값...)조건을 만들 때 좋다. item속성에는 항목을 가리킬 때 사용할 변수의 이름을 지정한다. index속성에는 항목의 인덱스 값을 꺼낼 때 사용할 변수 이름을 지정한다. collection속성에는 java.util.List 구현체나 배열 객체가 온다. open속성에는 최종 반환값의 접두어를 지정한다. close는 최종 반환값의 접미어를 지정한다. separatory 속성은 반복으로 생ㅇ성하는 값을 구분하기 위해 붙이는 문자열을 지정한다. |
| \<bind name="변수명" value="값"/> | `<bind>`태그는 변수를 생성할 때 사용한다. |


<br/>

**사용예**

```xml
//MySqlProjectDao.xml

  <select id="selectList" resultMap="projectResultMap">
	  select PNO, PNAME, STA_DATE, END_DATE, STATE
	  from PROJECTS
	  order by 
	  <choose> 
        <when test="orderCond == 'TITLE_ASC'">PNAME asc</when>
        <when test="orderCond == 'TITLE_DESC'">PNAME desc</when>
        <when test="orderCond == 'STARTDATE_ASC'">STA_DATE asc</when>
        <when test="orderCond == 'STARTDATE_DESC'">STA_DATE desc</when>
        <when test="orderCond == 'ENDDATE_ASC'">END_DATE asc</when>
        <when test="orderCond == 'ENDDATE_DESC'">END_DATE desc</when>
        <when test="orderCond == 'STATE_ASC'">STATE asc</when>
        <when test="orderCond == 'STATE_DESC'">STATE desc</when>
        <when test="orderCond == 'PNO_ASC'">PNO asc</when>
        <otherwise>PNO desc</otherwise>
    </choose>
  </select>
  
    <update id="update" parameterType="map">
    update PROJECTS 
 	<set>
      <if test="title != null">PNAME=#{title},</if>
      <if test="content != null">CONTENT=#{content},</if>
      <if test="startDate != null">STA_DATE=#{startDate},</if>
      <if test="endDate != null">END_DATE=#{endDate},</if>
      <if test="state != null">STATE=#{state},</if>
      <if test="tags != null">TAGS=#{tags}</if>
    </set>
    where PNO=#{no}
  </update>
  
```


<br/>


### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
