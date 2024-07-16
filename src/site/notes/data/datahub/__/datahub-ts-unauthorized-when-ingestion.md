---
{"dg-publish":true,"permalink":"/data/datahub/__/datahub-ts-unauthorized-when-ingestion/","noteIcon":"","created":"2024-06-30T00:39:32.594+09:00"}
---

#datahub #troubleshooting 

### tl;dr
- datahub helm 프로젝트의 삭제/생성을 반복하다 사용하는 secret 이 변경되었다.
	- 해당 시크릿은 존재하지 않는 경우 임의의 값을 생성하였다.
- 이를 방지하고자 helm chart 내 default 값을 기입하여 시크릿이 변경되는 문제를 방지하였다.
- **원인과 해결은 다음과 같다.**
	- 원인: helm 프로젝트 삭제/배포로 인한 시크릿 변경으로 GMS 통신 불가
	- 해결: 외부 서비스가 아니기 때문에 default 값을 helm chart 에 기입하여 변경되는 문제 방지

### background
- datahub CLI 로 dbt 소스 ingestion 하던 중
- token 은 expired 하지 않은 설정으로 만들었는데 401 unauthorized 오류가 발생함
```
[2023-01-04, 18:39:12 ] {pod_manager.py:197} INFO - HTTPError: 401 Client Error: Unauthorized for url: http://datahub-datahub-gms.data.svc.cluster.local:8080/aspects/urn%3Ali%3Adataset%3A%28urn%3Ali%3AdataPlatform%3Adbt%2Ccoinone-data-dev.dbt_metric.date_spine%2CDEV%29?aspect=globalTags&version=0
```

### assumption

#### 정상 토큰으로 요청 시 WARN 메시지가 발생하지 않는다.
- 1DAY expired 토큰 생성 후 정상동작하지 않는 토큰과 비교하여 warning 메시지가 발생하는지 확인한다.
	```
	`01:43:07.861 [qtp522764626-31] WARN  c.d.a.a.AuthenticatorChain:70 - Authentication chain failed to resolve a valid authentication. Errors: [(com.datahub.authentication.authenticator.DataHubSystemAuthenticator,Failed to authenticate inbound request: Authorization header is missing 'Basic' prefix.), (com.datahub.authentication.authenticator.DataHubTokenAuthenticator,Failed to authenticate inbound request: Unable to verify the provided token.)]`
	```
- 결과
	- 새로 생성한 토큰 & 문제가 발생하는 토큰 모두 WARN 메시지는 발생하였다.
	- 메세지로 확인할 수 있는 부분은 없다.

#### datahub-auth-secrets 시크릿을 재성성할 때마다 jwt key 가 변경된다.
- 시크릿을 삭제한 내역이 있는데 이 영향으로 401 오류가 발생했다.
- 삭제 전 datahub-auth-secrets 의 `token_service_signing_key`, `token_service_salt` 값은 다음과 같다.
	- token_service_signing_key: MSRxxxx...xxxx
	- token_service_salt: Iwexxxx...xxxx
- 삭제 후 재 생성 & GMS 삭제 후 토큰값
	- token_service_signing_key: MSRxxxx...xxxx
	- token_service_salt: Iwexxxx...xxxx
- 결과
	- **삭제 전 후 token 값 변경은 없다.**

#### datahub-auth-secrets 템플릿이 렌더링 할 때마다 변경된다.

```
{{- $secret := lookup "v1" "Secret" .Release.Namespace "datahub-auth-secrets" -}}  
{{- $data := $secret.data | default dict -}}  
{{- if .Values.global.datahub.metadata_service_authentication.provisionSecrets -}}  
apiVersion: v1  
kind: Secret  
metadata:  
  name: "datahub-auth-secrets"  
type: Opaque  
data:  
  system_client_secret: {{ index $data "system_client_secret" | default (randAlphaNum 32 | b64enc | quote) }}  
  token_service_signing_key: {{ index $data "token_service_signing_key" | default (randAlphaNum 32 | b64enc | quote) }}  
  token_service_salt: {{ index $data "token_service_salt" | default (randAlphaNum 32 | b64enc | quote) }}  
{{- end -}}
```

- token_service_signing_key 값을 randAlphaNum 32 로 임의로 생성함
- 이는 어플리케이션을 삭제 후 재설치 한다면 siginig key 값이 변경됨!
- 애플리케이션을 삭제하고 재배포하면 signing key 가 변경될 것이다.
- result
	- **helm 차트를 변경할 일이 있어 새로운 브랜치로 재 생성해보니 signing_key & salt 값이 변경됨**
	- token 생성에 사용하는 키 변경으로 인해 ingestions 이 실패했다.
	- gms pod 에서 확인한 key 값
		- token_service_signing_key: HrFxxxx...xxxx
		- token_service_salt: S6axxxx...xxxx
- **secret template 문법을 해석해보면**
	- lookup 함수로 datahub-auth-secrets 시크릿이 존재한다면, secret 변수로 할당한다.
	- secret 변수가 존재한다면 시크릿 데이터를 data 변수로 담고, 없다면 빈 dictionary 를 생성한다.
	- 각 시크릿 value 에는 data 값의 key 가 존재하면 value 를, 없다면 영문자+숫자 랜덤값을 이용한다. (randAlphaNum)
- [참고문서](https://itnext.io/manage-auto-generated-secrets-in-your-helm-charts-5aee48ba6918)

### solution
- token 생성에 필요한 key 값들을 미리 default 로 설정한다.
- 사용되는 키값들은 다음과 같다.
	- DATAHUB_SYSTEM_CLIENT_SECRET
	- DATAHUB_TOKEN_SERVICE_SIGNING_KEY
	- DATAHUB_TOKEN_SERVICE_SALT
	- SECRET_SERVICE_ENCRYPTION_KEY
- helm chart 시크릿 템플릿을 변경하였다.
	- `encryption_key_secret: {{ index $data "encryption_key_secret" | default ("{default_값}" | b64enc | quote) }}`
- 우려사항
	- 코드 레포가 유출된다면
		- 키값을 레포에 raw 하게 작성하지 않고 할 수 있는 방법은 없을까?
