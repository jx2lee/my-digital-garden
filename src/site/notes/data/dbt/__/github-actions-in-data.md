---
{"dg-publish":true,"permalink":"/data/dbt/__/github-actions-in-data/","tags":["dbt","cicd"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgShowTags":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---



> [!tldr;]
> Atlassian Bamboo 에 구성한 dbt pipeline 을 Github Actions 으로 마이그레이션한 과정을 소개한다.


### diagrams


![|500](https://i.imgur.com/sDFjU9T.png)


![|500](https://i.imgur.com/VbXvLeK.png)


### Github Actions


- [변성윤님 포스트](https://zzsza.github.io/development/2020/06/06/github-action/)
- 현재 코인원 [Github Actions 구조](https://tech.kakao.com/2022/05/06/github-actions/)는 카카오 엔터프라이즈와 유사
- Actions 작성 시 아래 링크를 적극 활용
	- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions): Actions Syntax 에 대한 포스트. 자동완성 기능이 제공되지 않은 환경에서 작업 시 참고하면 좋다.
	- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows): github 이벤트에 따른 트리거를 설정할 수 있는데, 각 이벤트에 대해 설명하고 이를 활용할 수 있는 예시들이 있어 yml 작성 시 참고하면 좋다.
	- [Variables](https://docs.github.com/en/actions/learn-github-actions/variables): GitHub Actions 워크플로 실행에 대한 기본 변수를 설정함. `run` step 으로 사용하지 않고 default 로 제공하는 변수를 사용하면 yml 을 간편하게 작성할 수 있다.


### shallow dive


변경 혹은 추가된 모델만 build 하는 slim-build Action 아래와 같고 라인별로 내용을 살펴본다.
> 아래 테스트한 내용은 추후 변경될 수 있음

```yaml
name: dbt Slim ci
on:
  push:
    branches:
      - dev
      - main
    paths:
      - models/**
      - seeds/**
      - macros/**
      - docs/**
      - snapshots/**
jobs:
  build_model:
    runs-on: [self-hosted, linux, build]
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3

      - name: Configure GCP credential for develop environment
        if: ${{ github.ref == 'refs/heads/dev' }}
        run: |
          echo "${{ secrets.DEV_DBT_KEYFILE }}" | base64 -d > ${{ vars.KEYFILE_PATH }}
          sudo mv ${{ vars.KEYFILE_PATH }} /var/run/secrets

      - name: Configure GCP credential for production environment
        if: ${{ github.ref == 'refs/heads/main' }}
        run: |
          echo "${{ secrets.PRD_DBT_KEYFILE }}" | base64 -d > ${{ vars.KEYFILE_PATH }}
          sudo mv ${{ vars.KEYFILE_PATH }} /var/run/secrets

      - name: Configure AWS credentials
        uses: actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ECR }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY_ECR }}
          aws-region: ap-northeast-2

      - name: Login to ECR
        id: login-ecr
        uses: actions/amazon-ecr-login@v1

      - name: Set env to run container
        id: set-envs
        run: |
          echo "latest-author=$(git log -1 --pretty=format:%ae)" >> "$GITHUB_ENV"
          echo "repo-name=$(basename -s .git ${{ github.repository }})" >> "$GITHUB_ENV"

      - name: dbt slim ci
        id: build-image
        env:
          ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}
        run: |
          docker run --name dbt_slim_build \
            -v /var/run/secrets/keyfile:/app/keyfile \
            -v ${{ github.workspace	}}:/app \
            -v ${AWS_WEB_IDENTITY_TOKEN_FILE}:${AWS_WEB_IDENTITY_TOKEN_FILE} \
            -e AWS_WEB_IDENTITY_TOKEN_FILE=${AWS_WEB_IDENTITY_TOKEN_FILE} \
            -e AWS_ROLE_ARN=${AWS_ROLE_ARN} \
            -e DBT_TARGET=${GITHUB_REF_NAME} \
            -e JIRA_USERNAME=jj.lee \
            -e JIRA_TOKEN=${{ secrets.JIRA_TOKEN }} \
            -e JIRA_ASSIGNEE=${{ env.latest-author }} \
            ${{ env.ECR_REGISTRY }}/${{ env.repo-name }}:latest slim
```


#### name


- workflow identifier
- UI 에서 확인할 수 있다.
	- ![|300](https://i.imgur.com/RZPpmTZ.png)


#### on


- workflow 를 자동으로 트리거하기 위해 사용
- 위 예시는 [`push`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onpushbranchestagsbranches-ignoretags-ignore) 이벤트가 발생하면 trigger
	- dev/main 에 push 이벤트가 발생하는 경우 변경된 모델들만 빌드하기 위함
		- paths: 설정한 path 의 변경사항이 있는 경우에만 트리거
		- branches/branches-ignore/tags/tags-ignore 는 [glob patterns](https://en.wikipedia.org/wiki/Glob_(programming)) 을 따름
	- 경영지표 제공을 위해 매일 스케쥴로 빌드해야하는 경우 [`schedule`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onschedule) 을 사용할 수 있음
	- 만약 작성중인 workflow 를 재사용하고 싶다면 [`workflow_call`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call)
	- 이 외에도 다양한 이벤트 기반으로 트리거할 수 있는 기능들이 제공할 수 있기 때문에 [링크](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on) 참고

#### jobs


- 워크플로 실행은 기본적으로 병렬로 실행되는 하나 이상의 job 으로 구성
- 위 예시는 하나의 build_model job 이 존재하고,
	- 7 step 으로 구성됨
	- 각 step 은 id 를 설정할 수 있어 build-image step 에서의 ECR_REGISTRY 변수와 같이 step output 을 재사용 할 수 있음
		- `ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}`
		- https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsteps
	- github 의 [secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets?tool=webui), [Variables](https://docs.github.com/en/actions/learn-github-actions/variables) 을 사용할 수 있음
		- ![|500](https://i.imgur.com/ylNE4pq.png)
			- `${{ secrets.DEV_DBT_KEYFILE }}`
			- `${{ secrets.PRD_DBT_KEYFILE }}`
			- `${{ secrets.JIRA_TOKEN }}`
		- ![|500](https://i.imgur.com/zsptDSG.png)
			- `${{ vars.KEYFILE_PATH }}`
- 서비스 클러스터와 ci/cd 클러스터는 분리되어 있기 때문에 서비스의 s3 로 접근이 필요했다.
	- runs-on 에 **build** label 을 추가한다.
	- build 레이블을 추가한 runner 는 service 클러스터의 s3 로 접근이 가능하기 때문에 꼭 추가해야한다.


#### steps


- job 은 step 의 모임인데, 명령을 실행하거나/설정 작업을 실행하거나/리포지토리 & 공용 리포지토리 또는 Docker 레지스트리에 push 하는 작업을 실행할 수 있음
- [condition (if)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-only-run-job-for-specific-repository)
	- 특정 조건을 만족하는 경우에만 step 을 실행
	- 예시에는 환경에 따라 GCP keyfile 을 선택하게끔 설정
- docker run step 에서의 volume 과 환경변수
	- `AWS_WEB_IDENTITY_TOKEN_FILE`, `AWS_ROLE_ARN`
		- `-v ${AWS_WEB_IDENTITY_TOKEN_FILE}:${AWS_WEB_IDENTITY_TOKEN_FILE}`
		- `-e AWS_ROLE_ARN=${AWS_ROLE_ARN}`
	- 위 변수들과 볼륨은 s3 접근을 위해 IAM Role 토큰이 필요함. 필수적으로 mount & 선언해야하고 없다면 s3 접근이 되지 않을 것이다.
	- [참고](https://github.com/actions/actions-runner-controller/issues/246)


### 그 외 추가로 생성한 워크플로우


#### reusable workflow


[Reusing workflows](https://docs.github.com/ko/actions/using-workflows/reusing-workflows)

- 기존 워크플로를 재사용하여 중복을 피하는 방법 중 하나
- DRY 를 따르고자 reusable workflow 를 이용하고 싶었지만 불가능한 환경
	- job 은 병렬로 실행하며
	- 지정한 노드에 pod 형태로 배포됨
		- `runs-on` 라벨링된 노드는 2개 노드이므로
		- ECR/GCP 인증을 다른 Job 으로 실행하면, 실제 인증이 필요한 Job 에서 인증이 안된채로 실행함
- 타셀 제공 저장소에는 reusable workflow 를 사용
    - dbt-build 하는 workflow 를 재사용하게 만들어두고,
    - daily / by pushing / manual 워크플로우에서 호출하는 구조로 생성


#### PR auto labeler


- 오픈소스 기여하면서 PR 생성 시 자동으로 Label 을 달아주는 기능을 보았다.
- 프로젝트에 녹여내면 재밌겠다고 생각한다.
- e.g
	- https://github.com/datahub-project/datahub/pull/7637
	- docs 폴더 내 파일을 수정하고 PR 을 생성하니,
		- github-actions 봇이 label 을 자동으로 할당한다.
		- ![|500](https://i.imgur.com/8Iv1HnX.png)
		- [ref](https://github.com/datahub-project/datahub/blob/master/.github/workflows/pr-labeler.yml)
- [labeler](https://github.com/actions/labeler) 액션으로 구성 **완료**했다.


#### container job


[Running jobs in a container](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)
- dbt build 하는 step 가독성이 많이 떨어짐
- env, volume 등을 깔끔하게 건네줄 수 있지만, reusable workflow 을 사용하지 못하는 이유와 동일함
	- job 을 새로 설정하면
	- ecr/gcp 인증이 풀린 상태에서 컨테이너를 실행할 수 있음
- 대안은 없나요?
	- 네. 동일한 노드에서 진행할 수 없으면 사용이 불가함


### SQL linter


- 보푸라기를 제거하는 린트 롤러(Lint roller)처럼 코드의 오류나 버그, 스타일 따위를 점검하는 것을 [린트(Lint) 혹은 린터(Linter)](https://en.wikipedia.org/wiki/Lint_(software))라고 부름
- SQL linter 로 많이 사용되는 sqlfluff 활용
- 오픈된 액션이 존재하여 조금 수정해서 사용

> [!todo]
> docker container 형태로 실행되는 구조라 이미지를 빌드함. 워크플로우 실행시간이 길어 javascript 로 변환해서 사용하면 효율적


### 팁


- IDE 플러그인을 적극 활용해보자
	- JetBrains 에서는 자동완성 기능 및 설명을 보여주는 기능을 제공함
	- ![|500](https://i.imgur.com/xP1Jdxh.png)
	- ![|500](https://i.imgur.com/5jhy69j.png)
- 공식 문서를 잘 활용하자
	- bamboo 보다는 보기도 편하고 예시도 많음
	- 또한, usecase 도 많아 (오픈소스에서 사용하는 actions 들) 레퍼런스가 많으니 검색을 해보고 적용해보자.


### reference


- https://tech.kakaoenterprise.com/180
- https://tech.kakao.com/2022/05/06/github-actions/
