# databasecheck
Check Oracle datamodel

Check if tables have a primary key.

```
select table_name
from user_tables
minus
select table_name
from user_constraints
where constraint_type='P'
```

Check if foreign keys have an index.

```
select
case
   when b.table_name is null then
      'unindexed'
   else
      'indexed'
 end as status,
   a.table_name      as table_name,
   a.constraint_name as fk_name,
  a.fk_columns      as fk_columns,
  b.index_name      as index_name,
  b.index_columns   as index_columns
from
(
   select 
    a.table_name,
   a.constraint_name,
   listagg(a.column_name, ',') within
group (order by a.position) fk_columns
from
   user_cons_columns a,
   user_constraints b
where
   a.constraint_name = b.constraint_name
and 
   b.constraint_type = 'R'
and 
   a.owner = b.owner
group by 
   a.table_name, 
   a.constraint_name
) a
,(
select 
   table_name,
   index_name,
   listagg(c.column_name, ',') within
 group (order by c.column_position) index_columns
from
   user_ind_columns c
group by
   table_name, 
   index_name
) b
where
   a.table_name = b.table_name(+)
and 
   b.index_columns(+) like a.fk_columns || '%'
order by 
   1 desc, 2
```
