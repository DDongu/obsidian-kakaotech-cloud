---
sticker: emoji//1f5a5-fe0f
tags:
  - terraform
  - 실습
---
# Terraform 간단 실습

1. 예제 1 
```tf
# main.tf
provider "aws" {
    region = "ap-northeast-2"
}

resource "aws_instance" "practice1" {
  ami = "ami-062cddb9d94dcf95d"
  instance_type = "t2.micro"
}
```

![](images/Pasted%20image%2020250318132012.png)
![](images/Pasted%20image%2020250318132048.png)

2. 예제 2
```terraform
provider "aws" {
    region = "ap-northeast-2"
}

resource "aws_instance" "practice1" {
  ami = "ami-062cddb9d94dcf95d"
  instance_type = "t2.micro"

  tags = {                         # 태그 추가
    Name = "001205"
  }
}
```
![](images/Pasted%20image%2020250318132354.png)
=> `terraform plan`을 통해 제대로 반영이될지 미리 체크

![](images/Pasted%20image%2020250318132520.png)
![](images/Pasted%20image%2020250318132549.png)
=> apply 후 태그가 정상적으로 들어간 모습
(위 명령어에서 `-auto-approve`는 재확인 절차없이 바로 apply를 하는 명령어)


> [!info] TAG 사용
> default_tags 를 사용하면 프로젝트 내의 모든 리소스에 대한 기본 태그를 지정할 수 있음
> ❗이때 그냥 tags와 중복되면 안됨


3. destroy 삭제 명령어
생성한 리소스를 모두 삭제
```sh
terraform destroy
```
