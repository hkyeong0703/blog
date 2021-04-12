---
title: "Mysql View Definer"
date: 2020-08-13T00:17:43+09:00
draft: fasle
---

```sql
https://dev.mysql.com/doc/refman/8.0/en/create-view.html

CREATE
    [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = user]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```

django unit test를 진행하는 과정에서 로컬에 세팅한 DB에 있는 view table에서 권한이 없어 select를 못하는 에러가 계속 발생.

```sql
_mysql_exceptions.OperationalError: (1449, "The user specified as a definer ('db_user'@'%') does not exist")
```

이유를 모른채 삽질을 하던 중, 생성 쿼리를 뽑아 봄.

```sql
create definer = 'db_user'@`%` view view_name as select_statement
```

definer가 지정된 상태였다.

내가 생성 할 시에 따로 설정을 해주지 않아도 기본 세팅으로 definer가 지정된 상태인 것이다.

로컬 PC에서 unit test용 DB를 생성할 때, mysqldump를 통하여 초기 생성을하는데
이 과정에서 definer로 지정된 유저가 아닌 다른 유저로 사용되고있기때문에 문제가 되는 것이다.

이를 해결하기위해 여러 해결책을 찾아보았지만 결론은 2가지였다.
1. mysqlpump —skip-definer를 이용

[https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html#option_mysqlpump_skip-definer](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html#option_mysqlpump_skip-definer)

2. mysqldump 이후, sed 명령어를 사용하여 DEFINER 제거

```bash
sed -i '' 's/DEFINER=[^*]*\*/\*/g' $sql_file # MACOS
sed -i 's/DEFINER=[^*]*\*/\*/g' $sql_file # linux
```

mysqlpump는 mysql 5.7.8 버전부터 사용 가능한 것으로 확인이 되었다. 사용하고 있는 DB 사양 중 이하 버전도 포함이 되어 있기도 하고, 데이터 백업 없이 DDL만 가져오기 때문에 2번 방법을 통해 해결하였다.

