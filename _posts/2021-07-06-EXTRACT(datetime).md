---
title: SCN
categories:
- Oracle
feature_text: |
  ## EXTRACT(datetime)
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

EXTRACT 함수는 datetime 또는 interval 값에서 지정된 datetime 필드의 값을 추출하고 반환한다. TIMEZONE&#95;REGION또는 TIMEZONE&#95;ABBR를 추출할 때 반환되는 값은 적절한 표준 시간대 이름 또는 약어를 포함하는 문자열이다. 다른 값을 추출할 때 반환된 값은 그레고리력에 있다. 시간대 값이 있는 datetime에서 추출할 때 반환되는 값은 UTC다. 시간대 이름 및 해당 약어 목록을 V$TIMEZONE&#95;NAMES를 보려면 동적 성능보기를 조회해야 한다.  

```sql
SELECT EXTRACT(TIMEZONE_HOUR FROM SYSTIMESTAMP) FROM DUAL;

SELECT EXTRACT(HOUR FROM SYSTIMESTAMP) FROM DUAL;
```

TIMEZONE_&#42; 값으로 추출하는 경우 해당 세션에 설정된 TIMEZONE 값을 참조하여 추출한다. YEAR, MONTH, DAY, HOUR, MINUTE, SECOND는 UTC 기준시로 추출한다.  

```sql
SELECT EXTRACT(TIMEZONE_HOUR FROM SYSDATE) FROM DUAL;

SELECT EXTRACT(TIMEZONE_HOUR FROM TO_TIMESTAMP(SYSDATE, 'YYYY-MM-DD')) FROM DUAL;

SELECT EXTRACT(TIMEZONE_HOUR FROM TO_TIMESTAMP('2021-10-10 09:00:00', 'YYYY-MM-DD HH24:MI:SS')) FROM DUAL;
```

위의 쿼리는 오류가 발생한다. 위의 값들은 TIMEZONE 값을 추출해야 하지만 TIMEZONE 값을 가지고 있지 않기 때문에 오류를 발생시킨다.  

```sql
SELECT EXTRACT(HOUR FROM TO_TIMESTAMP('2021-10-10 09:00:00', 'YYYY-MM-DD HH24:MI:SS')) FROM DUAL;
```

위의 쿼리는 TIMEZONE 값을 참조할 필요가 없으므로 오류를 발생시키지 않는다.
