# VPC & Route53
## 아티클
### VPC
-  사전 배경 지식으로 aws가 서비스를 제공하는 단위들을 배웠다. IDC 여러 개가 모이면 AZ(Availability Zone)가 된다. 그리고  AZ(Availability Zone) 여러 개가 모이면  Region이 된다. 이 단위들을 알고 아래 토폴로지를 보면 이해하기 쉽다.

<img width="555" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/2c3a1c78-d0b3-4bc5-8a80-5a07cfa91091">

```
토폴로지 설명
1. 내가 VPC를 만들면 내가 속한 Region 안에서 구축 된다.
2. 내가 커스터마이징해서 내 VPC 안에 서브넷을 이것 저것 구축하면 내 Region 안에 있는 AZ 안에 구축이 된다.
3. 내가 만든 VPC가 인터넷과 연결이 되려면 Ingernet Gateway를 통해 나가야 한다.
```

- 기본 지식을 배웠으니 나도 VPC를 만들고, 서브넷을 나눠서 각각의 서브넷이 인터넷에 접근할 수 있게 구축을 할 수 있게 해보자. 내가 만들 vpc 구조는 아래와 같다.
<img width="460" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/02c3dd97-bec1-42f9-9f62-05760849c9a8">
- 먼저, vpc를 이름은 my-second-vpc-vpc, ip는 10.10.0.0/16(IPv4의 CIDR식 표기)로 할당해서 만든다.
<img width="626" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/c0b8b0c9-14c8-47d8-8d26-cd5e210a8f9c">

- 이제 VPC 안에 Public 서브넷,Private 서브넷을 각각 만들어 보자. 먼저 서브넷을 이름만 다르게 2개를 만들자.하나는 my-second-public-subnet으로 만들고, 10.10.240.0/20을 할당했다. 다른 하나는 my-second-private-subnet을 10.10.0.0/20을 할당했다. <img width="851" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/119d5b44-af01-4fa0-aa07-42a1173f2c21">

    - 여기서 세션에서도 중요하게 다룬 aws에서 제공하는 서브넷 리소스의 특징이 나온다. 바로 처음 서브넷 리소스를 만들 때는 public과 private 서브넷 차이를 구분해서 만들지 않는다는 점이다. aws에서 서브넷을 public 또는 private으로 만들려면 서브넷에 연결된 라우팅 테이블에 Internet Gateway로 연결 여부 규칙을 설정해야 한다.
-  이제 내가 아까 만든 이름만 서로 다르고 상태는 똑같이 만든 my-second-public-subnet, my-second-private-subnet을 각각 public, private으로 만들자.

    - 우선 서브넷을 인터넷과 연결할 문인 Internate Gateway를 my-second-vpc-igw로 만들었다. 
    <img width="628" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/45681fca-c859-4110-9f3e-c1420477050d">

    - 그리고 아까 만든 서브넷들을 연결해서 사용할 라우팅 테이블을 각각 Public 용, Private 용으로 2개 만들었다. 라우팅 테이블을 만들 때에도 따로 이름만 다르게 my-second-public-routing-table, my-second-private-routing-table 으로 2개 만든다. 그리고 각각을 public, private 서브넷에 연결한다.
    <img width="628" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/d22db67c-0196-486d-ab6d-15aed5e6ca0d">

    - 그 다음은 라우팅 테이블의 연결 규칙을 편집한다(드디어!). 기본적으로 라우팅 테이블들은 연결된 서브넷이 소속된 vpc 대역대들에 local 로 연결된 연결 규칙이 존재한다. 즉, 기본적으로 같은 vpc 내부에 있는 ip들은 서로 ip를 알고 있다는 규칙이다. 

    - Private 서브넷용 라우팅 테이블이라면 위 기본 규칙을 유지하면 된다. 그 이유는 Private 서브넷의 경우 Public 서브넷을 거쳐서 인터넷에 연결되기 때문에, 일단 Public 서브넷과의 연결 규칙만 존재하면 된다.
    <img width="625" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/2f54b924-6593-4fb4-8380-5b62b05d3f1f">

    - Public 서브넷 연결용 라우팅 테이블에는 기본 연결 규칙에 새 연결 규칙을 추가하는 것이 필요하다. 나의 목적은 Public 서브넷의 경우 어디서든 인터넷을 통해 서브넷에 접속할 수 있게 하는 것이다. 그래서, 연결규칙의 대상을 어디서 오는 트래픽이든 연결이 가능하게끔 0.0.0.0/0으로 선택한다. 그리고 내가 연결하고자하는 인터넷에 해당하는 우리 vpc 에 연결된 Intenet Gate Way를 선택한다. 
    <img width="633" alt="image" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/97fb3dc7-6fb1-4d51-93b4-065511affef5">

- 위와 같이 라우팅 테이블 연결 규칙 편집까지 마치면 아래 vpc 리소스맵을 완성할 수 있다.

    <img width="880" alt="스크린샷 2023-11-04 오후 2 32 47" src="https://github.com/HyeJung-Hwang/asc-ewha-article/assets/79091824/7d50c7a8-a43e-4b26-ab12-04bc7540b9c0">

### Route53
- Route53은 AWS의 DNS 서비스이다. 단순히 도메인 네임 등록기관의 역할만 하는 것이 아니라, DNS 쿼리 서비스도 같이 제공한다. AWS가 늘상 강조하는 고가용성과 확작성을 가지는 완전 관리형 DNS이다. 그리고 100% SLA라고 한다. Rouet53이 제공하는 리소스는 크게 DNS 레코드, 호스팅 영역, 헬스 체크 3가지다.

- DNS 레코드는  특정 도메인에 대해 설정된 DNS 레코드로 도메인에 대한 쿼리응답에 해당한다. 호스팅 영역은 레코드들의 집합으로 Public과 Private으로 나뉜다. Public 호스팅 영역은 인터넷 상에서 라우트를 제공할 레코드를 모아놓는 곳이다. Privated Hosted Zones은 VPC 내부의 트래픽을 라우팅하는 레코드를 모아놓는 곳이다.
