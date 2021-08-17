---
title: ASSM
categories:
- Oracle
feature_text: |
  ## TBL$OR$IDX$PART$NUM 함수
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

파티션 번호를 반환한다.  

tbl$or$idx$part$num(
  <partitioned_table_name>           IN VARCHAR2,
  <index_identifier>                 IN NUMBER,
  <number_of_column_in_partition_key IN NUMBER,
  p#                                 IN BINARY_INTEGER,
  <partition_value>)                 IN ROWID)
RETURN <UNKNOWN>;
