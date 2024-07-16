---
{"dg-publish":true,"permalink":"/data/datahub/__/actions-framework-overview/","dgShowInlineTitle":true,"noteIcon":"","created":"2024-06-30T00:39:32.592+09:00"}
---

#datahub #action #test 

### background
column description 에 특정 키워드를 포함하면 tag 를 자동으로 붙이는 기능이 필요하다.
이를 **[DataHub Action framework](https://datahubproject.io/docs/actions)** 로 구현할 수 있는 실험을 하고자 한다.
- 확인사항
	- Action Framework 로 변경된 메타데이터를 확인할 수 있는지
	- 변경된 메타데이터를 확인하고 이를 커스텀하게 개발할 수 있는지

### experiment
- env: local (`docker/dev-without-neo4j.sh`)
- ingestion: mssql
- action yaml: `actions-sample.yml`
```yaml
name: "hello_world"
source:
  type: "kafka"
  config:
    connection:
      bootstrap: broker:29092
      schema_registry_url: http://schema-registry:8081
filter:
  event_type: "MetadataChangeLogEvent_v1"
  event:
    entityType: "dataset"
    changeType: ["UPSERT", "DELETE"]
    aspectName: "schemaMetadata"
action:
  type: "hello_world"
```
- 실험 순서
	1. reference 의 테스트 json 을 ingestion 한다.
	2. 테스트 json 내 idx 칼럼 description 을 변경한다.
		- `testtestttttttt -> after...`
	3. 변경한 json 을 재 ingestion 한다.
	4. action 에 찍힌 이벤트 로그를 확인한다.

### result
- 변경된 description || 새로 추가한 entity 의 description 은 MCL(Metadata Change Log) 에서 확인할 수 있다.
	- 메타데이터가 변경된 경우에는 aspect & previousAspectValue 값으로 변경 전 후 데이터를 확인할 수 있다.
	- **단, 변경이 일어나지 않은 데이터 주입 시 이벤트에서 description 을 확인할 수 없다.**
	- 이벤트 로그
		- 첫 수집시 발생하는 로그
			```json
			{
			    "event_type": "MetadataChangeLogEvent_v1",
			    "event": {
			        "entityType": "dataset",
			        "entityUrn": "urn:li:dataset:(urn:li:dataPlatform:mssql,RT_CoinoneDB.dbo.ACCOUNT,DEV)",
			        "changeType": "UPSERT",
			        "aspectName": "schemaMetadata",
			        "aspect": {
			            "value": "{\"platformSchema\":{\"com.linkedin.schema.MySqlDDL\":{\"tableSchema\":\"\"}},\"created\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"lastModified\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"fields\":[{\"nullable\":false,\"fieldPath\":\"idx\",\"description\":\"testtestttttttt\",\"isPartOfKey\":true,\"type\":{\"type\":{\"com.linkedin.schema.NumberType\":{}}},\"recursive\":false,\"nativeDataType\":\"INTEGER()\"}],\"schemaName\":\"RT_CoinoneDB.dbo.ACCOUNT\",\"version\":0,\"hash\":\"\",\"platform\":\"urn:li:dataPlatform:mssql\"}",
			            "contentType": "application/json"
			        },
			        "systemMetadata": {
			            "lastObserved": 1677202881255,
			            "runId": "file-2023_02_24-10_41_21"
			        },
			        "created": {
			            "time": 1677202881441,
			            "actor": "urn:li:corpuser:__datahub_system"
			        }
			    },
			    "meta": {
			        "kafka": {
			            "topic": "MetadataChangeLog_Versioned_v1",
			            "offset": 1060,
			            "partition": 0
			        }
			    }
			}
			```
		- description 변경 이후 재수집 시 발생하는 로그
			```json
			{
			    "event_type": "MetadataChangeLogEvent_v1",
			    "event": {
			        "entityType": "dataset",
			        "entityUrn": "urn:li:dataset:(urn:li:dataPlatform:mssql,RT_CoinoneDB.dbo.ACCOUNT,DEV)",
			        "changeType": "UPSERT",
			        "aspectName": "schemaMetadata",
			        "aspect": {
			            "value": "{\"platformSchema\":{\"com.linkedin.schema.MySqlDDL\":{\"tableSchema\":\"\"}},\"created\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"lastModified\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"fields\":[{\"nullable\":false,\"fieldPath\":\"idx\",\"description\":\"after...\",\"isPartOfKey\":true,\"type\":{\"type\":{\"com.linkedin.schema.NumberType\":{}}},\"recursive\":false,\"nativeDataType\":\"INTEGER()\"}],\"schemaName\":\"RT_CoinoneDB.dbo.ACCOUNT\",\"version\":0,\"hash\":\"\",\"platform\":\"urn:li:dataPlatform:mssql\"}",
			            "contentType": "application/json"
			        },
			        "systemMetadata": {
			            "lastObserved": 1677213158911,
			            "runId": "file-2023_02_24-13_32_38"
			        },
			        "previousAspectValue": {
			            "value": "{\"created\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"platformSchema\":{\"com.linkedin.schema.MySqlDDL\":{\"tableSchema\":\"\"}},\"lastModified\":{\"actor\":\"urn:li:corpuser:unknown\",\"time\":0},\"schemaName\":\"RT_CoinoneDB.dbo.ACCOUNT\",\"fields\":[{\"nullable\":false,\"fieldPath\":\"idx\",\"description\":\"testtestttttttt\",\"isPartOfKey\":true,\"type\":{\"type\":{\"com.linkedin.schema.NumberType\":{}}},\"nativeDataType\":\"INTEGER()\",\"recursive\":false}],\"version\":0,\"hash\":\"\",\"platform\":\"urn:li:dataPlatform:mssql\"}",
			            "contentType": "application/json"
			        },
			        "previousSystemMetadata": {
			            "lastObserved": 1677202974099,
			            "runId": "file-2023_02_24-10_41_21"
			        },
			        "created": {
			            "time": 1677213159020,
			            "actor": "urn:li:corpuser:__datahub_system"
			        }
			    },
			    "meta": {
			        "kafka": {
			            "topic": "MetadataChangeLog_Versioned_v1",
			            "offset": 1071,
			            "partition": 0
			        }
			    }
			}
			```
- 참고
	- [kafka topic](https://datahubproject.io/docs/how/kafka-config/#topic-configuration): MetadataChangeLog_Versioned_v1
	- [event_type](https://datahubproject.io/docs/what/mxe): [MetadataChangeLogEvent_v1](https://datahubproject.io/docs/advanced/mcp-mcl/)

### conclusion
앞서 확인할 두 가지 사항에 대한 결론은 다음과 같다.
- ~~Action Framework 로 변경된 메타데이터를 확인할 수 있는지~~
	- **가능하다.**
	- 단, 변경되지 않은 메타데이터 재수집 시 이벤트는 발생하지 않기 때문에
		- DEV/PRD 모두 메타데이터를 삭제한 후 진행하고 재수집 해야한다.
- ~~변경된 메타데이터를 확인하고 이를 커스텀하게 개발할 수 있는지~~
	- **가능하다.**
	- [Developing Action](https://datahubproject.io/docs/actions/guides/developing-an-action) 과 같이 환경에 맞게 개발이 가능하다.
- **TO-DO**
	- action management
		- 현재 action pod 에 접근하여 프로세스를 실행해야한다.
			- `datahub actions run -c {yaml}`
			- 관리가 어렵다.
		- 쉽게 관리할 수 있는 방안이 필요하다. 커뮤니티에 문의해봐도 좋을 듯 싶다.
	- [develop action](https://datahubproject.io/docs/actions/guides/developing-an-action)

### reference
- 실험에 사용한 ingestion file 예시

<details>
```json
[  
  { 
	"entityType": "container",  
	"entityUrn": "urn:li:container:bc0dd858aef120cc872958302fa28bfe",  
	"changeType": "UPSERT",  
	"aspectName": "containerProperties",  
	"aspect": {  
	  "value": "{\"customProperties\": {\"platform\": \"mssql\", \"instance\": \"DEV\", \"database\": \"rt_coinonedb\"}, \"name\": \"rt_coinonedb\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938739,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:bc0dd858aef120cc872958302fa28bfe",  
	"changeType": "UPSERT",  
	"aspectName": "status",  
	"aspect": {  
	  "value": "{\"removed\": false}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938740,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:bc0dd858aef120cc872958302fa28bfe",  
	"changeType": "UPSERT",  
	"aspectName": "dataPlatformInstance",  
	"aspect": {  
	  "value": "{\"platform\": \"urn:li:dataPlatform:mssql\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938740,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:bc0dd858aef120cc872958302fa28bfe",  
	"changeType": "UPSERT",  
	"aspectName": "subTypes",  
	"aspect": {  
	  "value": "{\"typeNames\": [\"Database\"]}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938741,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:751bf4bff992efefb2445d387ef31d75",  
	"changeType": "UPSERT",  
	"aspectName": "containerProperties",  
	"aspect": {  
	  "value": "{\"customProperties\": {\"platform\": \"mssql\", \"instance\": \"DEV\", \"database\": \"rt_coinonedb\", \"schema\": \"dbo\"}, \"name\": \"dbo\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938770,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:751bf4bff992efefb2445d387ef31d75",  
	"changeType": "UPSERT",  
	"aspectName": "status",  
	"aspect": {  
	  "value": "{\"removed\": false}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938771,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:751bf4bff992efefb2445d387ef31d75",  
	"changeType": "UPSERT",  
	"aspectName": "dataPlatformInstance",  
	"aspect": {  
	  "value": "{\"platform\": \"urn:li:dataPlatform:mssql\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938771,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:751bf4bff992efefb2445d387ef31d75",  
	"changeType": "UPSERT",  
	"aspectName": "subTypes",  
	"aspect": {  
	  "value": "{\"typeNames\": [\"Schema\"]}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938771,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "container",  
	"entityUrn": "urn:li:container:751bf4bff992efefb2445d387ef31d75",  
	"changeType": "UPSERT",  
	"aspectName": "container",  
	"aspect": {  
	  "value": "{\"container\": \"urn:li:container:bc0dd858aef120cc872958302fa28bfe\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858938772,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "dataset",  
	"entityUrn": "urn:li:dataset:(urn:li:dataPlatform:mssql,RT_CoinoneDB.dbo.ACCOUNT,DEV)",  
	"changeType": "UPSERT",  
	"aspectName": "container",  
	"aspect": {  
	  "value": "{\"container\": \"urn:li:container:751bf4bff992efefb2445d387ef31d75\"}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858939422,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"proposedSnapshot": {  
	  "com.linkedin.pegasus2avro.metadata.snapshot.DatasetSnapshot": {  
		"urn": "urn:li:dataset:(urn:li:dataPlatform:mssql,RT_CoinoneDB.dbo.ACCOUNT,DEV)",  
		"aspects": [  
		  {  
			"com.linkedin.pegasus2avro.common.Status": {  
			  "removed": false  
			}  
		  },  
		  {  
			"com.linkedin.pegasus2avro.dataset.DatasetProperties": {  
			  "customProperties": {},  
			  "name": "ACCOUNT",  
			  "tags": []  
			}  
		  },  
		  {  
			"com.linkedin.pegasus2avro.schema.SchemaMetadata": {  
			  "schemaName": "RT_CoinoneDB.dbo.ACCOUNT",  
			  "platform": "urn:li:dataPlatform:mssql",  
			  "version": 0,  
			  "created": {  
				"time": 0,  
				"actor": "urn:li:corpuser:unknown"  
			  },  
			  "lastModified": {  
				"time": 0,  
				"actor": "urn:li:corpuser:unknown"  
			  },  
			  "hash": "",  
			  "platformSchema": {  
				"com.linkedin.pegasus2avro.schema.MySqlDDL": {  
				  "tableSchema": ""  
				}  
			  },  
			  "fields": [  
				{  
				  "fieldPath": "idx",  
				  "description": "testtestttttttt",  
				  "nullable": false,  
				  "type": {  
					"type": {  
					  "com.linkedin.pegasus2avro.schema.NumberType": {}  
					}  
				  },  
				  "nativeDataType": "INTEGER()",  
				  "recursive": false,  
				  "isPartOfKey": true  
				}  
			  ]  
			}  
		  }  
		]  
	  }  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858939423,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  },  
  {  
	"entityType": "dataset",  
	"entityUrn": "urn:li:dataset:(urn:li:dataPlatform:mssql,RT_CoinoneDB.dbo.ACCOUNT,DEV)",  
	"changeType": "UPSERT",  
	"aspectName": "subTypes",  
	"aspect": {  
	  "value": "{\"typeNames\": [\"table\"]}",  
	  "contentType": "application/json"  
	},  
	"systemMetadata": {  
	  "lastObserved": 1676858939424,  
	  "runId": "mssql-2023_02_20-11_08_56"  
	}  
  }  
]
</details>
