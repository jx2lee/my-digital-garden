---
{"author":"jx2lee","aliases":["Airbyte 차트 구성하기 (oauth2 & 커넥터 관리)"],"created":"2024-06-30T00:39:32.589+09:00","last-updated":"2023-08-12 18:04","tags":["airbyte","helm","oauth2-proxy"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":true,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":true,"dg-pass-frontmatter":true,"permalink":"/data/airbyte/__/airbyte-custom-chart/","dgHomeLink":true,"dgPassFrontmatter":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgShowInlineTitle":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":""}
---



> [!tldr]
> Airbyte 접근제어를 위해 oauth2-proxy 를 사용했고 chart 를 변경하여 default connector 를 제어했다. 접근제어는 이메일 또는 source/dest 커넥터를 차트에 추가하고 argo sync (auto sync 는 off 인 경우) 를 실행한다.

---

### Why I did it


- Airbyte 0.44.2 기준 접근 제한 기능을 제공하지 않았습니다. ([참고](https://github.com/airbytehq/airbyte/issues/768)) 
  - 접근을 허가받은 사용자에게 source/destination 커넥터 관리 기능/커넥션을 관리할 수 있는 기능을 제공해야하는 요구사항이 있습니다. (보안 조직에서의 오구사항)
- 접근 제어 기능을 제공하지 않아 다음 선택지가 있었습니다.
  1. 접근 제어 기능을 개발하고 OSS 프로젝트에 기여한다.
  2. oauth 기능을 대체하는 프록시를 함께 제공한다.
- **조직 상황 / 플랫폼 제공 기한 여부로 인해 `2`번 방식으로 작업을 진행하였습니다.**
  - Oauth2 프록시를 제공하는 oauth2-proxy 프로젝트를 발견하였고 이를 기존 Airbyte 차트에 녹여 배포하기로 결정하였습니다.
  - **Airbyte 공식 차트 + oauth2-proxy 차트를 subchart 로 추가하여 배포**


### background


Airbyte
- application version: **0.44.2**
- [chart](https://github.com/airbytehq/airbyte-platform/tree/v0.45.17-helm) version: **0.45.17**


oauth2-proxy
- application version: **7.4.0**
- [chart](https://github.com/bitnami/charts/tree/main/bitnami/oauth2-proxy) version: **3.6.3**
- (참고) oauth2-proxy 아키텍처
![|500](https://i.imgur.com/QdNwSJN.png)

### with Oauth2


- Oauth 기능을 추가한 후 기대한 작업 과정은 다음과 같습니다.
    - 요청자
		- 티켓을 생성하고 슬랙으로 공유합니다.
		- 작업이 완료되길 대기한 후 완료된 시점에 Airbyte 접근을 시도합니다.
	- 작업자
		- 요청자 email 을 차트에 기입합니다. 작성되 email 사용자에게만 인증을 허용합니다.
			- `oauth2-proxy.configuration.authenticatedEmailsFile`
		- 차트 repo 반영 > ArgoCD sync 작업으로 수정사항을 반영합니다. autoSync 활성화로 refresh > sync 하지 않아도 좋지만, 현재 데이터 조직에서 authSync 를 비활성화 하여 운영중입니다.

values 파일은 다음과 같이 구성하였습니다.
- webapp 서비스를 upstream 으로 설정하여 oauth 인증완료된 사용자만이 airbyte 에 접근할 수 있도록 구성하였습니다.
- authenticatedEmailsFile 에 접근 가능한 사용자 이메일을 작성합니다. 존재하는 사용자만이 접근이 가능하고, 기입되지 않은 사용자는 Airbyte 에 접근이 불가합니다.

```yaml
# airbyte root values yaml
...
oauth2-proxy:
  enabled: true
  image:
    repository: {xxxx}/bitnami/oauth2-proxy
    tag: latest

  nodeSelector:
    {key: value}

  service:
    type: NodePort
    port: 80

  configuration:
    clientID: xxx
    clientSecret: xxx
    redirectUrl: xxx

    content: |-
      skip_provider_button = true
      cookie_secure = false
      session_store_type = "redis"
      upstreams = [ "http://airbyte-airbyte-webapp-svc.data.svc.cluster.local:80" ]
      redis_connection_url = [ "redis://airbyte-redis-master.data.svc.cluster.local:6379/1" ]

    authenticatedEmailsFile:
      enabled: true
      content: |-
        jj.lee@xxx.com
        ...
        ...
        sh.lim@xxx.com
        yoori@xxx.com
```

### default connector 제어


- why i did it 부분 요구사항 중 보안성 검토 결과에 따라 제공하면 안되는 source/desitnation 커넥터들이 존재합니다. 때문에, Airbyte 서비스가 실행될 때 커넥터를 안보이도록 수정이 필요합니다.
- bootloader 차트가 커넥터를 관리합니다. 따라서 bootloader 차트를 수정하였습니다.
    - `manageCatalog` custom 값으로 source/destination 을 관리합니다.
- 커넥터를 추가하는 과정은 다음과 같습니다.
	- 커넥터를 airbyte-bootloader.manageCatalog 의 source 혹은 destination 에 추가합니다.
	- 차트 repo 반영 > argo sync 합니다. 싱크 시 bootloader pod 가 실행됩니다.

```yaml
# airbyte-bootloader.manageCatalog
  manageCatalog:
    enabled: true
    sources:
      - "Coin API"
      - "CoinGecko Coins"
      - "CoinMarketCap"
      - "ElasticSearch"
    destinations:
      - "BigQuery"
      - "BigQuery (denormalized typed struct)"

```

```yaml
# _helpers.tpl
{{- define "bootloader.connectionList" -}}
{{- if .Values.manageCatalog.enabled }}
    {{- $combinedSources := join "', '" .Values.manageCatalog.sources }}
    {{- $combinedDestinations := join "', '" .Values.manageCatalog.destinations }}
    {{- printf "('%s', '%s')" $combinedSources $combinedDestinations }}
{{- else}}
    {{- printf "('Coin API', 'CoinGecko Coins', 'CoinMarketCap', 'BigQuery', 'BigQuery (denormalized typed struct)')"}}
{{- end }}
```

####  modify airbyte-bootloader
- 다음과 같이 동작하도록 수정하였습니다.
    - 작성한 source / destination 을 읽어 bootloader initContainer 이후 기입한 커넥터를 제외한 나머지를 삭제하는 컨테이너가 동작하도록 변경하였습니다.
	- 원래 metadb(postgresql) 의 모든 default connector 를 삽입하는 컨테이너는 initContainer 가 아닙니다. 따라서  **inint 후 커넥터를 삭제하는 작업을 추가하기 위해 container → initContainer 로 변경**하였스빈다.
		- 기존: bootloader container(이하 A라고 표현합니다)
		- 변경: A -> initContainer 로 변경합니다. 필요한 커넥터만 남기고 삭제하는 작업을 담당하는 container 를 추가합니다.
	- 위 내용을 실행하기 위해서는 이미지가 필요합니다. 조직에서 사용하는 ECR 에 psql-client 이미지를 업로드하고, client 에서 쿼리를 실행하는 방법으로 구현하였습니다.
- source/destination 의 목록을 쿼리로 변경하기 위해 helm 변수를 활용하였습니다. `manageCatalog.sources/destinations`
- 변경한 bootloader deployment 템플릿은 다음과 같습니다.

```yml
# additional container
  containers:
    - name: airbyte-bootloader-clean
      image: {{ printf "%s:latest" .Values.image.clean.repository }}
      imagePullPolicy: "{{ .Values.image.pullPolicy }}"
      command:
        - "psql"
        - "-c"
        - "delete from actor_definition where name not in {{ include "bootloader.connectionList" . }};"
        - "dbname=db-airbyte user=$(DATABASE_USER) host=$(DATABASE_HOST)"
      env:
        {{- if eq .Values.global.deploymentMode "oss"  }}
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: {{ .Values.global.configMapName | default (printf "%s-airbyte-env" .Release.Name) }}
              key: DATABASE_HOST
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Values.global.database.secretName | default (printf "%s-airbyte-secrets" .Release.Name ) }}
              key: {{ .Values.global.database.secretValue | default "DATABASE_PASSWORD" }}
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: {{ .Values.global.secretName | default (printf "%s-airbyte-secrets" .Release.Name) }}
              key: DATABASE_USER
        {{- end }}
```

### conlcusion


- 무엇을 했나요
    - 요구사항을 만족하도록 서로 다른 두 차트를 혼합하여 새로운 차트를 구성하였습니다. (oauth 기능)
    - 요구사항을 만족하도록 기존 템플릿을 변경하였습니다. (bootloader 수정을 통한 source/destination 커넥터 관리)
- 어떤걸 고민하였나요
    - 공식 차트를 최소한으로 수정하고 싶었습니다. 결과적으로는 bootloader 템플릿을 많이 수정하였는데요. `initContainer` 를 추가하여 필요한 src/dest 커넥터만 남기도록 쿼리를 실행하도록 했어요. 이는 기존 로직을 유지한 채 최소한의 수정만 발생했어요.
    - 요구사항을 만족하는, 누구에게나 공개할 수 있도록 가독성 높고 이쁜 차트를 구성하고 싶었습니다. 현재 Airbyte 차트에서는 oauth 기능을 제공하고 있어요(keyclock). 작업 당시 상황을 고려하여 두 차트를 효율적으로 혼합하여 유지보수 가능한 차트를 만들었어요.
- 무엇을 배웠나요
    - 접근제어가 불가능한 도구로 oauth2-proxy 를 활용할 수 있어 좋았어요. 특히 어렵지 않게 설정이 가능하고 추후 다양한 도구에서 사용할 수 있을 것이라 기대해요.
    - helm 차트를 구성하고 어떤 컴포넌트로 생성하는지, 클라우드 네이티브한 어플리케이션을 어떻게 구성하는지 간접적으로 확인할 수 있었습니다. 이후 차트작업은 눈감고 할 정도로 많은 걸 배웠어요.
