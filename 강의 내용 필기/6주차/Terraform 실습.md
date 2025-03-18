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

# 여러 리소스를 정의 및 사용 방법
```
provider "aws" {
  region = "ap-northeast-2"
  default_tags {
    tags = {
      Name  = "toby-practice2"
    }
  }
}
  
resource "aws_instance" "webserver" {
  ami = "ami-062cddb9d94dcf95d"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver_sg.id]
  
  user_data = <<-EOF
          #!/bin/bash
          yum update && yum install -y busybox
          echo "Heollo, World" > index.html
          nohup busybox httpd -f -p 8080 &
        EOF
}
  
resource "aws_security_group" "webserver_sg" {
  name = "webserver-sg-studentN"
  ingress {
      from_port = 8080
      to_port = 8080
      protocol = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
  }
}
```
=> 보안그룹에 대한 리소스를 정의하고 인스턴스에 연결

![](images/Pasted%20image%2020250318143023.png)
=> `terraform graph`를 통해 리소스 간의 의존성 확인 가능

![](images/Pasted%20image%2020250318143107.png)
=> 위 `https://dreampuf.github.io/GraphvizOnline/` 사이트를 통해 graph로 표현된 의존성을 시각화 가능
=> 시각화된 이미지를 보면 인스턴스가 보안그룹을 가리키는걸 확인할 수 있음

*위 코드에서 `user_data`에 해당하는 코드가 잘못되어서 busybox에 접속할 수가 없음(테스트 불가)*

# VPC 생성 방법

#### 생성된 VPC 리소스 맵
![결과 예상 이미지](images/Pasted%20image%2020250318164715.png)
## Subnet 생성

`cidrsubnet(prefix,newbits, netnum)`
- ﻿﻿Prefix: VPC CIDR block (Subnet으로 만들 전체 CIDR을 의미)
- ﻿﻿Newbits: Prefix를 제외한 나머지 bit 중 Subnet의 Prefix로 사용할 bit 수
- ﻿﻿Netnum: Subnet의 Prefix가 가질 값을 십진수로 지정
ex) 
cidrsubnet("10.0.0.0/24", 1, 1) => **"10.0.0.128/25"**
cidrsubnet("10.0.0.0/24", 2, 3) => **"10.0.0.192/26"**

```terraform
...
# Subnet 생성
variable "vpc_main_cidr" {
  description = "VPC main CIDR block"
  default = "10.0.0.0/23"
} 
  
resource "aws_vpc" "my_vpc" {
  cidr_block = var.vpc_main_cidr
  instance_tenancy = "defualt"
  enable_dns_hostnames = true
}
  
resource "aws_subnet" "pub_sub_1" {
  vpc_id = aws_vpc.my_vpc.id # 사용할 VPC
  cidr_block = cidrsubnet(aws_vpc.my_vpc.cidr_block, 1, 0) # cidr 블록 설정
  availability_zone = "ap-northeast-2a" # 사용할 가용영역
  map_public_ip_on_launch = true # 자동 퍼블릭 할당
}

...
```

## Internet GateWay 생성
```
# Internet GateWay 생성
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
}
```

## Route Table 생성
```
# Route Table 생성
resource "aws_route_table" "pub_rt" {
  vpc_id = aws_vpc.my_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my_igw.id
  }
}
  
resource "aws_route_table" "prv_rt1" {
  vpc_id = aws_vpc.my_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat_gw_1.id
  }
}
```


## Route Table과 Subnet을 연결
```
# Route Table과 Subnet을 연결
resource "aws_route_table_association" "pub_rt_asso" {
    subnet_id = aws_subnet.pub_sub_1.id
    route_table_id = aws_route_table.pub_rt.id
}
  
resource "aws_route_table_association" "pub_rt_asso2" {
    subnet_id = aws_subnet.pub_sub_2.id
    route_table_id = aws_route_table.pub_rt.id
}
```

## EIP 생성(NAT G/W에서 사용)
```
# EIP 생성(NAT G/W에서 사용)
resource "aws_eip" "nat_eip1" {
  domain = "vpc"
}
  
resource "aws_eip" "nat_eip2" {
  domain = "vpc"
}
```

## NAT G/W 생성
```
# NAT G/W 생성
resource "aws_nat_gateway" "nat_gw_1" {
  allocation_id = aws_eip.nat_eip1.id
  subnet_id = aws_subnet.pub_sub_1.id
  depends_on = [ aws_internet_gateway.my_igw ] # 명시적 의존성 선언(소스의 생성 순서를 지정 가능)
}
```


# 클러스터 구성 방법

**파일을 분리하여 코드 작성**
`cluster.tf` -> 클러스터 구성 관련 코드
`outputs.tf` -> 출력 변수 작성
`variables.tf` -> 입력 변수 작성
## variables.tf
```
# 입력 변수 작성
variable "server_port" {
  description = "Webserver's HTTP port"
  type = number
  default = 80
}
  
variable "my_ip" {
  description = "My public IP"
  type = string
  default = "0.0.0.0/0"
}
```

## outputs.tf
```
# 출력 변수 작성
output "alb_dns_name" {
  value = aws_lb.webserver_alb.dns_name
  description = "The domain name of the load balancer"
}
```
## cluster.tf

### 보안 그룹 생성
```
# 보안 그룹 생성
resource "aws_security_group" "webserver_sg" {
  name = "webserver-sg-studentN"
  vpc_id = aws_vpc.my_vpc.id
  ingress {
    from_port = var.server_port
    to_port = var.server_port
    protocol = "tcp"
    cidr_blocks = [ aws_subnet.pub_sub_1.cidr_block, aws_subnet.pub_sub_2.cidr_block]
  }
}
``` 

### 시작 템플릿 생성
```
# 시작 템플릿 생성
resource "aws_launch_template" "webserver_template" {
  image_id = "062cddb9d94dcf95d"
  instance_type = "t3.micro"
}
```

### Auto Scaling Group 생성

```
# Auto Scaling Group 작성
resource "aws_autoscaling_group" "webserver_asg" {
  vpc_zone_identifier = [ aws_subnet.pub_sub_1.ud, aws_subnet.pub_sub_2.id ]
  min_size = 2
  max_size = 3
  launch_template {                # 확장 시 생성될 템플릿 지정
    id = aws_launch_template.webserver_template.id
    version = "$Latest"            # 가장 최신 버전의 템플릿 사용
  }
  depends_on = [ aws_vpc.my_vpc, aws_subnet.pub_sub_1, aws_subnet.pub_sub_2 ]
}
```

> [!info] 특정 그룹에 해당하는 Subnet을 모두 넣으려면?
> 
> ![Pasted%20image%2020250318171716.png](images/Pasted%20image%2020250318171716.png)
> => 다음과 같이 데이터 소스기능(`data`)을 통해 CSP에서 제공하는 읽기 전용 정보를 API로  가져오는 것이 가능. (사진의 예제처럼 사용)

## ALB 보안그룹
```
## ALB 보안그룹
resource "aws_security_group" "alb_sg" {
  name = var.alb_security_group_name
  vpc_id = aws_vpc.my_vpc.id
  
  ingress {
    from_port = var.server_port
    to_port = var.server_port
    protocol = "tcp"
    cidr_blocks = [ var.my_ip ]
  }
  
  egress {
    from_port = 0                   # 모든 트래픽 허용
    to_port = 0                     # 모든 트래픽 허용
    protocol = "-1"                 # 모든 프로토콜 허용
    cidr_blocks = [ "0.0.0.0/0" ]
  }
}
```

## ALB 설정
```
## ALB 설정
resource "aws_lb" "webserver_alb" {
  name = var.alb_name
  load_balancer_type = "application"
  subnets = [ aws_subnet.pub_sub_1.id, aws_subnet.pub_sub_2.id ]
  security_groups = [ aws_security_group.alb_sg.id ]
}
```

## ALB 타겟 그룹
```
## ALB 타겟 그룹
resource "aws_lb_target_group" "target_asg" {
  name = var.alb_name
  port = var.server_port
  protocol = "HTTP"
  vpc_id = aws_vpc.my_vpc.id
  
  health_check {
    path = "/"
    protocol = "HTTP"
    matcher = "200"
    interval = 15
    timeout = 3
    healthy_threshold = 2
    unhealthy_threshold = 2
  }
}
```

## 리스너
```
## 리스너
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.webserver_alb.arn
  port = var.server_port
  protocol = "HTTP"
  
  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.target_asg.arn
  }
}
```

## 리스너 규칙 지정
```
## 리스너 규칙 지정
resource "aws_lb_listener_rule" "webserver_asg_rule" {
    listener_arn = aws_lb_listener.http.arn
    priority = 100
  
    condition {
        path_pattern {
          values = ["*"]
        }
    }
    action {
        type = "forward"
        target_group_arn = aws_lb_target_group.target_asg.arn
    }
}
```