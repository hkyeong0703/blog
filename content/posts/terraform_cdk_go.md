---
title: "Terraform용 cdk를 이용하여 Go로 AWS 인프라 구축 시작하기"
date: 2021-07-14T22:01:24+09:00
description: ""
tags: [
	"terraform",
	"cdk",
    "cdktf",
	"go",
	"hashicorp",
]
categories: []
disableComments: false
---



오늘은 Terraform용 CDK를 통해 Go언어를 사용하는 방법에대해 알아보려합니다.

우선 Terraform과 CDK가 무엇인지부터 간단하게 알아보겠습니다.



## Terraform이란?

HashiCorp사가 만든 오픈 소스 "코드형 인프라(IaC)" 툴 입니다. 인프라를 안전하고 효율적으로 구축, 변경 및 버전화 할 수 있습니다.

선언적인 코딩 툴, HCL(HashiCorp Configuration Language)라고 불리는 상위 레벨 구성 언어를 사용하거나 또는 JSON으로 어플리케이션 실행을 위해 원하는 "엔드 상태" 클라우드 또는 온프레미스 인프라를 기술하도록 합니다. 그런 다음 해당 엔드 상태에 도달하기 위한 계획을 생성하고 인프라를 프로비저닝하기 위한 계획을 실행합니다.

더 자세히 알아보고싶으시다면, [Terraform Doc](https://www.terraform.io/intro/index.html) 읽어보시면 좋을 듯 합니다.



## CDK란?

CDK(Cloud Development Kit)는 익숙한 프로그래밍 언어를 사용하여 클라우드 어플리케이션 리소스를 정의할 수 있는 오픈 소스 소프트웨어 개발 프레임워크입니다.



## Terraform용 CDK

그럼 이제 Terraform용 CDK를 알아보려합니다. 앞서 Terraform은 HCL이라는 언어를 이용해야한다고 설명했습니다. 새로운 언어를 사용하는게 좋을 수도 있지만, 좀 더 개발자들에게 익숙한 프로그래밍 언어를 사용하여 인프라를 정의한다면 더욱 편할 것입니다. 이를 가능하도록해주는 것이 Terraform용 CDK 즉, **cdktf(Cloud Development Kit for Terrafrom)**입니다. 지원되는 언어로는 [TypeScript](https://github.com/hashicorp/terraform-cdk/blob/main/docs/getting-started/typescript.md), [Python](https://github.com/hashicorp/terraform-cdk/blob/main/docs/getting-started/python.md), [Java](https://github.com/hashicorp/terraform-cdk/blob/main/docs/getting-started/java.md), [C#](https://github.com/hashicorp/terraform-cdk/blob/main/docs/getting-started/csharp.md), [Go](https://github.com/hashicorp/terraform-cdk/blob/main/docs/getting-started/go.md) 가 있습니다.

![terraform-platform.png](https://github.com/hashicorp/terraform-cdk/blob/main/docs/terraform-platform.png?raw=true)

작동 방식은 지원되는 프로그래밍 언어로 작성을하면 CDK를 통해 JSON을 생성한 다음 해당 JSON을 사용하여 표준 Terraform명령을 실행시키는 방식입니다.

`deploy` 명령어를 통해 프로그래밍 언어로 작성된 코드에서 Terraform에서 사용 할 수 있는 구조로 하위 디렉토리  `cdktf.out` 에  JSON 구성되고 인프라 프로비저닝을 진행합니다.



## cdktf 설치

설치하는 과정부터 알아봅시다.

우선, cdktf를 설치하기위해선 Terraform, Node.js, Yarn이 필요합니다.

- Terraform >= v0.12
- Node.js >= v12.16
- Yarn >= v1.21



cdktf를 설치하는 방법은 총 3가지가 있지만, 저는 MacOS를 사용하다보니 Homebrew를 이용하여 설치해보려고합니다. 다른 설치 방법은 [여기](https://learn.hashicorp.com/tutorials/terraform/cdktf-install?in=terraform/cdktf#install-cdktf) 를 참고해주세요.

아래의 명령어를 사용하면 간단히 설치가 완료됩니다.

```shell
$ brew install cdktf
```

설치가 완료되면 `cdktf` 명령어를 통해 설치 완료 여부를 확인하실 수 있습니다.

```shell
$ cdktf
cdktf [command]

Commands:
  cdktf deploy [stack] [OPTIONS]   Deploy the given stack
  cdktf destroy [stack] [OPTIONS]  Destroy the given stack
  cdktf diff [stack] [OPTIONS]     Perform a diff (terraform plan) for the given stack
  cdktf get [OPTIONS]              Generate CDK Constructs for Terraform providers and modules.
  cdktf init [OPTIONS]             Create a new cdktf project from a template.
  cdktf list [OPTIONS]             List stacks in app.
  cdktf login                      Retrieves an API token to connect to Terraform Cloud.
  cdktf synth [stack] [OPTIONS]    Synthesizes Terraform code for the given app in a directory.                    [aliases: synthesize]

Options:
  --version                   Show version number                                                                              [boolean]
  --disable-logging           Dont write log files. Supported using the env CDKTF_DISABLE_LOGGING.             [boolean] [default: true]
  --disable-plugin-cache-env  Dont set TF_PLUGIN_CACHE_DIR automatically. This is useful when the plugin cache is configured
                              differently. Supported using the env CDKTF_DISABLE_PLUGIN_CACHE_ENV.            [boolean] [default: false]
  --log-level                 Which log level should be written. Only supported via setting the env CDKTF_LOG_LEVEL             [string]
  -h, --help                  Show help                                                                                        [boolean]

Options can be specified via environment variables with the "CDKTF_" prefix (e.g. "CDKTF_OUTPUT")
```



## GO로 사용하여 AWS 인프라 구축하기

드디어 Go언어를 사용하여 AWS 인프라를 구축해보도록하겠습니다. 

시작하기위해선 Golang v1.16+ 버전과 AWS 계정, AWS Access Credentials이 필요합니다.



#### AWS 자격 증명 환경 변수 추가

AWS 인프라 구축을 위해 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 를 환경 변수로 추가합니다.

```shell
$ export AWS_ACCESS_KEY_ID=XXXXX
$ export AWS_SECRET_ACCESS_KEY=XXXXX
```



#### cdktf application 초기화

프로젝트를 구성할 디렉토리를 생성하여, 프로젝트 root 디렉토리를 이동합니다.
해당 단계에서 생성한 디렉토리를 프로젝트 root 디렉토리라고 부르겠습니다.

```shell
$ mkdir terraform/learn-cdktf-go
$ cd terraform/learn-cdktf-go
```

root 디렉토리에서 `cdktf init` 명령어를 통해 Go 템플릿으로 초기화를 시켜어야합니다.
이 과정에서 Project Name, Project Description을 입력 할 수 있는데 따로 입력하셔도되고 입력하지 않을 경우, default로 세팅됩니다.

```shell
$ cdktf init --template="go" --local
Note: By supplying '--local' option you have chosen local storage mode for storing the state of your stack.
This means that your Terraform state file will be stored locally on disk in a file 'terraform.tfstate' in the root of your project.

We will now set up the project. Please enter the details for your project.
If you want to exit, press ^C.

Project Name: (default: 'learn-cdktf-go')
Project Description: (default: 'A simple getting started project for cdktf.')
```



#### AWS provider로 추가

root 디렉토리 내 생성된 `cdktf.json` 파일을 열어 `aws` 를 provider 중 하나로 추가합니다.

```json
{
    "language": "go",
    "app": "go run main.go",
    "codeMakerOutput": "generated",
    "terraformProviders": [
        "hashicorp/aws@~> 3.42"
    ],
    "terraformModules": [],
    "context": {
        "excludeStackIdFromLogicalIds": "true",
        "allowSepCharsInLogicalIds": "true"
    }
}
```

추가한 `aws` provider를 설치하기위해 `cdktf get` 명령어를 사용해줍니다.
이 단계가 완료되는데엔 꽤 오랜 시간이 필요합니다. 저는 약 15분 가량이 걸렸던 것 같네요.

```shell
$ cdktf get
⠧ downloading and generating modules and providers...

Generated go constructs in the output directory: generated

The generated code depends on jsii-runtime-go. If you haven't yet installed it, you can run go mod tidy to automatically install it.
```

cdktf는 Go언어로 작성된 코드가  javascript로 작성된 CDK와 상호 작용할 수 있도록 `jsii` 라이브러리를 사용하는데 `go mod tidy` 명령어를 통해 `jsii` 가 설치되었는지 확인해주세요.

```shell
$ go mod tidy
go: finding module for package github.com/aws/jsii-runtime-go/runtime
go: found github.com/aws/jsii-runtime-go/runtime in github.com/aws/jsii-runtime-go v1.30.0
```



이제 Go언어를 사용하여 cdktf 사용할 준비는 모두 완료되었습니다.



#### cdktf 어플리케이션 정의

아주 간단하게 서울 리전에 미리 만들어진 AMI를 통해 인스턴스를 띄우고, 없애는 작업을 진행해보겠습니다.

서울 리전에 AMI를 이용하여 인스턴스를 띄우는 Go 언어의 코드는 아래와 같습니다.

```go
package main

import (
	"cdk.tf/go/stack/generated/hashicorp/aws"

	"github.com/aws/constructs-go/constructs/v3"
	"github.com/aws/jsii-runtime-go"
	"github.com/hashicorp/terraform-cdk-go/cdktf"
)

func NewMyStack(scope constructs.Construct, id string) cdktf.TerraformStack {
	stack := cdktf.NewTerraformStack(scope, &id)

	// AWS provider 구
	aws.NewAwsProvider(stack, jsii.String("aws"), &aws.AwsProviderConfig{
		Region: jsii.String("ap-northeast-2"),
	})

	// aws.NewInstance 클래스를 이용하여 인스턴스 정의
	instance := aws.NewInstance(stack, jsii.String("compute"), &aws.InstanceConfig{
		Ami:          jsii.String("ami-0aec549e9b79a186e"),
		InstanceType: jsii.String("t2.micro"),
		Tags: &map[string]*string{
			"Name":   jsii.String("Go-Demo"),
		},
	})

	// 결과로 생성된 인스턴스으 public ip를 반환
	cdktf.NewTerraformOutput(stack, jsii.String("public_ip"), &cdktf.TerraformOutputConfig{
		Value: instance.PublicIp(),
	})

	return stack
}

func main() {
	app := cdktf.NewApp(nil)

	NewMyStack(app, "learn-cdktf-go")

	app.Synth()
}
```



#### 배포하기 - 인프라 프로비저닝

`cdktf deploy` 를 통해 작성한 코드를 실행하여 배포합니다. 배포 작업이 완료되면 생성된 인스턴스의 public ip가 출력됩니다.

```shell
$ cdktf deploy
Deploying Stack: learn-cdktf-go
Resources
 ✔ AWS_INSTANCE         compute             aws_instance.compute

Summary: 1 created, 0 updated, 0 destroyed.

Output: public_ip = 13.124.68.3
```



AWS console EC2 대시보드에 들어가셔도 생성된 인스턴스를 확인하실 수 있습니다.

출력된 pucblic ip 또는 Name 태그를 지정했기에 'Go-Demo'로 검색하시면 확인 가능합니다.

![aws ec2 dashboard](https://hkyeong0703.github.io/posts/images/go-demo-ec2.png)



#### 인프라 정리하기

앞 단계에서 생성된 인프라를 정리해보겠습니다. 

Terraform을 이용하면, 직접 AWS Console에 접속하거나 cli를 이용하여 정리할 필요가 없습니다.

`cdktf destroy` 명령어를 통해 인스턴스를 정리할 수 있습니다.

```shell
$ cdktf destroy
Destroying Stack: learn-cdktf-go
Resources
 ✔ AWS_INSTANCE         compute             aws_instance.compute

Summary: 1 destroyed.
```

AWS console EC2 대시보드에서도 Terminated된 것을 확인하실수 있습니다.

![aws ec2 dashboard](https://hkyeong0703.github.io/posts/images/go-demo-ec2.png)



## 마치며

Terraform용 CDK를 이용하여, AWS EC2를 구축하고 정리하는 작업까지 진행해보며 사용 방법을 간단히 알아봤습니다.

CDK를 활용하면 보다 개발자가 편한 언어를 이용하여 인프라 프로비저닝을 할 수 있다는 장점은 정말 큰 것 같습니다.

하지만 Go언어를 통해 cdktf를 사용하기엔 아직은 실험적인, 베타 단계라고합니다.
cdktf 사용하시려고 계획하시거든 Go언어보단 Typesctipt 또는 Pythond 추천 드립니다.
