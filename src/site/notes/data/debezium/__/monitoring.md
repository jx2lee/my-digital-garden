---
{"author":"jx2lee","aliases":"CDC 파이프라인의 모니터링 환경 구성하기","created":"2024-09-02T20:58:04.000+09:00","last-updated":"2024-09-14 01:45","tags":["debezium","kafka","monitroing"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/data/debezium/__/monitoring/","dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---


> [!info] CDC 모니터링을 구성한 경험을 공유합니다.


### Debezium MySQL Connector Monitoring

Debezium MySQL 커넥터는 세 가지 유형의 메트릭을 제공합니다.

-   스냅샷 메트릭(snapshot metrics)은 스냅샷 수행 중 커넥터 작동에 대한 정보를 제공합니다.    
-   스트리밍 메트릭(streaming metrics)은 커넥터가 빈로그를 읽을 때 커넥터 작동에 대한 정보를 제공합니다.    
-   스키마 히스토리 메트릭(schema history metrics)은 커넥터의 스키마 기록 상태에 대한 정보를 제공합니다.
    

JMX를 사용하여 이러한 메트릭을 노출하는 방법에 대한 자세한 내용은 [Debezium 모니터링 문서](https://debezium.io/documentation/reference/2.4/connectors/mysql.html#mysql-monitoring "https://debezium.io/documentation/reference/2.4/connectors/mysql.html#mysql-monitoring")에서 확인할 수 있습니다. ⬆️ 제공하는 세 가지 메트릭 중 살펴볼 메트릭은 스트리밍 & 스키마 히스토리 매트릭입니다.
-   Q. 왜 snapshot metric 은 보지 않나요?
    -   준비한 debezium 의 스냅샷은 initial 기능만 제공하며 signal 을 통해 수동 스냅샷을 진행하지 않습니다. 즉, 데이터셀에서 준비한 데비지움에서 `스냅샷을 초기에만 사용한다 -> 메트릭을 보지 않겠다` 로 생각했습니다.
    -   따라서 대시보드에 snapshot metric 은 제외하였습니다.


### How to export metrics

Debezium과 Kafka 에 노출되는 메트릭은 Prometheus 와 Grafana 로 내보내고 표시할 수 있습니다. 필요한 구성에 대한 예제와 다양한 커넥터에 대한 예제 대시보드는 [Debezium 예제 저장소에](https://debezium.io/documentation/reference/2.4/operations/monitoring.html#using-prometheus-grafana "https://debezium.io/documentation/reference/2.4/operations/monitoring.html#using-prometheus-grafana")서 찾을 수 있습니다.

-   현재 데비지움 커넥터가 실행되는 커넥트 클러스터에 [jmx 옵션은 활성화](https://github.coinfra.net/coinone/data-helm/blob/master/coinone-data-warehouse-sink-connector/values.yaml#L104 "https://github.coinfra.net/coinone/data-helm/blob/master/coinone-data-warehouse-sink-connector/values.yaml#L104") 되어 있습니다. 따라서 커넥터에서 생성한 메트릭(정확히는 jmx metric 을 제공하는 커넥터)을 **프로메테우스로 전달, 프로메테우스 소스가 등록된 그라파나로 대시보드 구성이 가능**합니다.    
    -   윤성님의 [코멘트](https://coinone-jira.atlassian.net/browse/PLF-8110?focusedCommentId=164015 "https://coinone-jira.atlassian.net/browse/PLF-8110?focusedCommentId=164015")에 조금 더 자세한 내용이 포함되어 있습니다.
    -   프로메테우스로 메트릭 수집은 AMP Server 파드에 적용된 세부 설정에 의해 수행합니다. 실제 파드 안 `/etc/config/prometheus.yml` 에 이번 동작한 자동수집의 **relabel 설정**이 들어있었다고 합니다. 총 2개의 Job 이 자동수집 annotation에 의해 되었구요. 
        -   **Job name**: kubernetes-service-endpoints
        -   **Job name**: kubernetes-pods
    -   추가로, 프로메테우스로 수집한 메트릭은 AMP Workspace 라는 리소스로 저장되고, 프로메테우스 웹 혹은 그라파나 explorer 를 통해 메트릭 수집 현황 & 테스트 쿼리를 실행할 수 있다고 합니다.

-   [coinone-data-warehouse-sink-connector 의 jmx-configmap.yaml](https://github.coinfra.net/coinone/data-helm/blob/master/coinone-data-warehouse-sink-connector/templates/jmx-configmap.yaml "https://github.coinfra.net/coinone/data-helm/blob/master/coinone-data-warehouse-sink-connector/templates/jmx-configmap.yaml") 옵션을 조정, debezium 메트릭을 수집하도록 변경하였습니다.
    
    -   기존 배포한 jmx configmap 에는 debeizum 패턴이 존재하지 않아 promql 로 메트릭을 확인할 수 없었습니다.    
    -   메트릭 확인을 위해 debezium 메트릭 네이밍 패턴을 확인 → 추가하여 수집할 수 있도록 구성했습니다.
        -   참고: [커밋 해시](https://github.coinfra.net/coinone/data-helm/commit/d65b4300fe547cdb780dae867fc9e70edd07de5a "https://github.coinfra.net/coinone/data-helm/commit/d65b4300fe547cdb780dae867fc9e70edd07de5a")
    -   `- pattern: "debezium.([^:]+)<type=connector-metrics, context=([^,]+), server=([^,]+), key=([^>]+)><>RowsScanned" name: "debezium_metrics_RowsScanned" labels: plugin: "$1" name: "$3" context: "$2" table: "$4" - pattern: "debezium.([^:]+)<type=connector-metrics, server=([^,]+), task=([^,]+), context=([^,]+), database=([^>]+)>([^:]+)" name: "debezium_metrics_$6" labels: plugin: "$1" name: "$2" task: "$3" context: "$4" database: "$5" - pattern: "debezium.([^:]+)<type=connector-metrics, server=([^,]+), task=([^,]+), context=([^>]+)>([^:]+)" name: "debezium_metrics_$5" labels: plugin: "$1" name: "$2" task: "$3" context: "$4" - pattern: "debezium.([^:]+)<type=connector-metrics, context=([^,]+), server=([^>]+)>([^:]+)" name: "debezium_metrics_$4" labels: plugin: "$1" name: "$3" context: "$2" - pattern: "debezium.([^:]+)<type=connector-metrics, context=([^,]+), server=([^>]+)><>CapturedTables" name: "debezium_metrics_CapturedTables" labels: plugin: "$1" context: "$2" name: "$3"`
        
        -   **label 들은 필요에 따라 추가/삭제가 필요합니다**. 어떤 레이블들을 추가/삭제 하는것이 모니터링에 좋은지는 운영 경험이 쌓이면서 자연스레 해소될 것으로 예상해요.
            
-   [![](https://grafana.com/static/assets/img/fav32.png)Debezium MySQL Connector | Grafana Labs](https://grafana.com/grafana/dashboards/11523-mysql-connector/) 에서 제공하는 **템플릿을 활용**했습니다.
    
-   Grafana 개발환경에 데이터 조직 생성을 요청하였고, 인프라셀에서 프로메테우스 소스 등록을 도와주셨습니다.
    
    -   등록된 소스 정보는 다음과 같습니다.
        
        ![image-20240806-001034.png](blob:https://coinone-jira.atlassian.net/a1b2967c-3e51-4f56-ad88-e6c46890b9ad#media-blob-url=true&id=236580e7-ec8a-4bba-8744-8c2df93e6024&collection=contentId-759235237&contextId=759235237&mimeType=image%2Fpng&name=image-20240806-001034.png&size=422034&width=1460&height=1275&alt=image-20240806-001034.png)
        
        DEV Grafana 데이터 조직에 등록된 promethus 소스
        

## dashboard overview

**Dashboards > MSK > Debezium MySQL Connector** `Streaming Metrics`

![image-20240805-083006.png](blob:https://coinone-jira.atlassian.net/fdcdbdc8-237c-4724-8afe-92537692354f#media-blob-url=true&id=d22c1d77-cc60-41ce-96aa-19516850d36b&collection=contentId-759235237&contextId=759235237&width=1658&height=1311&alt=image-20240805-083006.png)

Streaming Metrics

**Dashboards > MSK > Debezium MySQL Connector** `Schema Metrics`

![image-20240805-083029.png](blob:https://coinone-jira.atlassian.net/e44accbb-1b99-4d23-b769-4a68a7fb8d87#media-blob-url=true&id=79604ba7-1d37-41de-befe-788b25bb0eb0&collection=contentId-759235237&contextId=759235237&mimeType=image%2Fpng&name=image-20240805-083029.png&size=349914&width=1661&height=641&alt=image-20240805-083029.png)

Snapshot Metrics

Notes

-   각 메트릭의 설명은 대시보드 내 마크다운으로 작성하였습니다. 각 메트릭에 대해 깊게 파보지는 못했고, 운영하며 고도화 할 예정입니다.
    
-   개발환경의 경우 커넥트 클러스터의 워커들이 스팟 인스턴스에 생성, 죽고 살아나기를 반복하여 메트릭 추이를 확인하는데 어려움이 있습니다. 운영 환경에 넘어가면 추이 확인이 용이할 것으로 예상합니다. (운영은 스팟 인스턴스 아니니까)
    
-   더 나아가 Sink Connector 메트릭과 조합하여 메세지가 실제 딜리버리 되는 과정에서의 지표도 확인하고 싶습니다. 단, 현재 사용중인 BigQuery Sink Connector(Legacy) 의 경우 debeizum 과 같이 메트릭을 제공하지 않습니다.
    
    -   커넥터 개발을 하는게 좋을 것 같기도 해요. Storage API 이용하면 Exactly Once 구현할 수 있고요. ([참고](https://docs.confluent.io/cloud/current/connectors/cc-gcp-bigquery-storage-sink.html#features "https://docs.confluent.io/cloud/current/connectors/cc-gcp-bigquery-storage-sink.html#features"))
        
    -   몽고 커넥터의 경우도 jmx 메트릭을 제공하네요. [![](https://www.mongodb.com/ko-kr/docs/assets/favicon.ico)Monitoring](https://www.mongodb.com/ko-kr/docs/kafka-connector/v1.12/monitoring/#monitor-the-connector)
        
