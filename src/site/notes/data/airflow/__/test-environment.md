---
{"author":"jx2lee","aliases":"신뢰성 있는 Airfow dag Repository 를 위한 여정","created":"2025-07-13T18:04:45.000+09:00","last-updated":"2025-07-13 18:04","tags":["airflow","test","ci","internal"],"project":{"include":false,"status":"doing","company":"Bithumb","duration":null},"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/data/airflow/__/test-environment/","dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---


### background

### objective

### howto
**Component Diagram**

```mermaid
flowchart BT
local[Local]
ci[CI]

changed_files[변경된 파일 확인]
unittest[Unittest 실행]

finish[end]

subgraph integration["Integration"]
    direction BT
    check_schema{non-prod 테스트 스키마 확인}
    create_schema[스키마 생성]
    
    subgraph ods["Source2Redshift"]
        direction BT
        check_only_ods{ods dag changed}
        run_glue_redshift_task(Source2Redshift)
        ods_end[end]

        check_only_ods --> |yes| run_glue_redshift_task --> ods_end[end]
        check_only_ods --> |no| ods_end
    end

    subgraph transform["Transformation"]
        direction BT
        check_only_mart{mart dag changed}
        run_query(Transformation)
        transform_end[end]

        check_only_mart --> |yes| run_query --> transform_end
        check_only_mart --> |no| transform_end
    end

    check_schema --> | exists | ods
    check_schema --> | not exists | create_schema --> ods
    ods --> transform
end

local --> |pre-commit| changed_files
ci --> |feature2prod MR| changed_files

changed_files --> |dag| integration --> finish
changed_files --> |utils| unittest --> finish
```

### result

### keytakeaway

### more