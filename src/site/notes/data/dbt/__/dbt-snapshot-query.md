---
{"dg-publish":true,"permalink":"/data/dbt/__/dbt-snapshot-query/","noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---


# 생성과정
snapshot 모델을 생성하는 과정은 다음 두 가지다.
1. 처음 생성했을 때
2. 처음 생성되고 이후 모델이 실행될 때

# 1. 모델을 처음 생성했을 때
**tl;dr**
- 하나의 쿼리만 실행한다.

쿼리를 살펴보면 다음과 같다.
```sql
    create or replace table `coinone-data-dev`.`temp_jj`.`snapshot_preregister_user`  
      
      
    OPTIONS()  
    as (  
        
    select *,  
        to_hex(md5(concat(coalesce(cast(id as string), ''), '|',coalesce(cast(updated_at as string), '')))) as dbt_scd_id,  
        updated_at as dbt_updated_at,  
        updated_at as dbt_valid_from,  
        nullif(updated_at, updated_at) as dbt_valid_to  
    from (  
          
  
  
select * from `coinone-data-dev`.`kakaobank_in_advance`.`preregister_user`  
  
    ) sbq  
  
  
  
    );
```

스냅샷 대상 테이블의 모든 컬럼과 hash 값인 dbt_scd_id, 검증을 위한 컬럼(dbt_valid_from, dbt_valid_to) 과 strategy 로 설정한 key(updated_at) 값들을 추가한다. DDL 은 직관적으로 이해하기 쉽다. 각 추가된 테이블의 자세한 내용은 [snapshot meta-fields](https://docs.getdbt.com/docs/build/snapshots#snapshot-meta-fields) 를 살펴보자.

# 2. 처음 생성되고 이후 모델이 실행될 때

## #1
```sql
create  
or replace table `coinone-data`.`dbt_metric`.`snapshot_preregister_user__dbt_tmp`  
  
  
    OPTIONS(  
      expiration_timestamp=TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 12 hour)  
    )  
    as (  
      with snapshot_query as (  
  
  
  
  
  
select * from `coinone-data`.`kakaobank_in_advance`.`preregister_user`  
  
  
    ),  
  
    snapshotted_data as (  
  
        select *,  
            id as dbt_unique_key  
  
        from `coinone-data`.`dbt_metric`.`snapshot_preregister_user`  
        where dbt_valid_to is null  
  
    ),  
  
    insertions_source_data as (  
  
        select  
            *,  
            id as dbt_unique_key,  
            updated_at as dbt_updated_at,  
            updated_at as dbt_valid_from,  
            nullif(updated_at, updated_at) as dbt_valid_to,  
            to_hex(md5(concat(coalesce(cast(id as string), ''), '|',coalesce(cast(updated_at as string), '')))) as dbt_scd_id  
  
        from snapshot_query  
    ),  
  
    updates_source_data as (  
  
        select  
            *,  
            id as dbt_unique_key,  
            updated_at as dbt_updated_at,  
            updated_at as dbt_valid_from,  
            updated_at as dbt_valid_to  
  
        from snapshot_query  
    ),  
  
    insertions as (  
  
        select  
            'insert' as dbt_change_type,  
            source_data.*  
  
        from insertions_source_data as source_data  
        left outer join snapshotted_data on snapshotted_data.dbt_unique_key = source_data.dbt_unique_key  
        where snapshotted_data.dbt_unique_key is null  
           or (  
                snapshotted_data.dbt_unique_key is not null  
            and (  
                (snapshotted_data.dbt_valid_from < source_data.updated_at)  
            )  
        )  
  
    ),  
  
    updates as (  
  
        select  
            'update' as dbt_change_type,  
            source_data.*,  
            snapshotted_data.dbt_scd_id  
  
        from updates_source_data as source_data  
        join snapshotted_data on snapshotted_data.dbt_unique_key = source_data.dbt_unique_key  
        where (  
            (snapshotted_data.dbt_valid_from < source_data.updated_at)  
        )  
    )  
  
    select * from insertions  
    union all  
    select * from updates  
  
    );
```

## #2
```sql
merge into `coinone-data`.`dbt_metric`.`snapshot_preregister_user` as DBT_INTERNAL_DEST  
    using `coinone-data`.`dbt_metric`.`snapshot_preregister_user__dbt_tmp` as DBT_INTERNAL_SOURCE  
    on DBT_INTERNAL_SOURCE.dbt_scd_id = DBT_INTERNAL_DEST.dbt_scd_id  
    when matched     and DBT_INTERNAL_DEST.dbt_valid_to is null     and DBT_INTERNAL_SOURCE.dbt_change_type in ('update', 'delete')        then  
update set dbt_valid_to = DBT_INTERNAL_SOURCE.dbt_valid_to    when not matched     and DBT_INTERNAL_SOURCE.dbt_change_type = 'insert'        then  
insert  
(  
`id`  
,  
`account_id`  
,  
`status`  
,  
`message_send_count`  
,  
`updated_at`  
,  
`created_at`  
,  
`dbt_updated_at`  
,  
`dbt_valid_from`  
,  
`dbt_valid_to`  
,  
`dbt_scd_id`  
)  
        values (`id`, `account_id`, `status`, `message_send_count`, `updated_at`, `created_at`, `dbt_updated_at`, `dbt_valid_from`, `dbt_valid_to`, `dbt_scd_id`)  
  
-- 이후 __dbt_tmp 테이블 삭제 쿼리 실행
```

# reference
- [Add snaphosts to your DAG](https://docs.getdbt.com/docs/build/snapshots)
