---
{"dg-publish":true,"permalink":"/etc//the-pains-in-terraform-collaboration/"}
---


https://itnext.io/pains-in-terraform-collaboration-249a56b4534e

- limits
    - 테라폼에서 관리하는 리소스를 명시하지 않으면, 실제 클라우드에 존재하는 리소스가 삭제되지 않는다.
    - A 리소스만 테라폼에서 관리하며 B 리소스는 클라우드에 존재하는 리소스인 경우 apply 명령 시 테라폼으로 명시한 리소스만 반영한다.


### extract resources from GCP (with gcloud cli)


```bash
# authentication
gcloud auth login

# supproted resource-type
gcloud beta resource-config list-resource-types

# bulk-export
gcloud beta resource-config bulk-export \
--project=coinone-data-dev \
--resource-format=terraform \
--path /path/to/directory
```
- bulk-export 지원하는 리소스는 두 번째 명령어로 확인할 수 있다.
- coinone-data-dev 프로젝트 리소스 리스트는 다음과 같다.
    - ArtifactRegistryRepository
    - BigQueryTable
    - ComputeFirewall
    - ComputeNetwork
    - ComputeSubnetwork
    - DNSManagedZone
    - IAMCustomRole
    - IAMServiceAccount
    - KMSKeyRing
    - PubSubTopic
    - StorageBucket


### 