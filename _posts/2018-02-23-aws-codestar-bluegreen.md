---
layout: post
title:  "AWS CodeStar를 이용한 Blue/Green 배포"
categories: aws
---

### 들어가며...

> 이전에 Jenkins를 이용한 CI 구성은 해본적은 있었지만, CD 구성은 처음인데다, AWS의 Code블라블라들도 처음 사용하는 터라, 많은 삽질을 한 것 같습니다. 그래도 CodeStar가 출시되어 조금은 편하게 구성할 수 있었습니다.

### AWS CodeStar 생성

> AWS CodeStar는 미리 정해진 템플릿으로 코드저장소부터 배포까지 한번에 생성하고 관리할 수 있게 만들어져 있습니다.  

1. console로 들어가서 Create a new project를 누릅니다.
1. 원하는 템플릿을 고릅니다. Web service, node.js, Amazon EC2 를 선택합니다.
1. Project name을 설정합니다. 코드저장소는 CodeCommit을 사용합니다.
1. 다음을 누르고, Edit Amazon EC2 configuration에서 EC2 사양과 VPC, subnet 설정을 할 수 있습니다.
1. Create Project를 누르고, EC2에 사용할 Key Pair를 지정합니다.
1. IDE 연동하는 화면이 나오는데, skip 하고 넘어갑니다.
1. 그럼 이제, 대시보드 화면이 나오고, 몇 분이 지나면, 샘플 코드와 함께 EC2 하나에 배포된 상태를 확인할 수 있습니다.

> CodeCommit에 새로운 Repogitory가 생겼고, CodePipeline과 CodeDeploy를 통해 새로운 EC2로 배포된 내용을 확인할 수 있습니다.  
CodeDeploy설정을 보면, In-place 배포로 설정되어 있습니다.

### ALB 생성

> Blue/Green 배포에서 사용할 ALB를 하나 생성해줍니다.

1. EC2 console의 Load Balancers로 가서 Create Load Balancer를 누릅니다.
1. HTTP ALB를 선택합니다.
1. Name을 설정하고, HTTP를 등록하고, vpc와 AZ를 선택합니다.
1. HTTPS를 등록한 경우, certificate설정이 필요합니다.
1. Security Group은 CodeStar가 생성한 SG를 선택합니다.
1. 새로운 Target Group을 설정합니다.
1. EC2 instance는 등록하지 않고 넘어갑니다.

### Auto Scaling Group 생성

> Blue/Green 배포를 위해서는 Auto Scaling Group이 필요합니다.

1. Instances 항목으로 가서 CodeStar가 만든 EC2를 선택합니다.
1. Action > Instance Settings > Attach to Auto Scaling Group을 선택합니다.
1. a new Auto Scaling Group을 생성합니다.

> Auto Scaling Groups 항목에서 생성된 Auto Scaling Group을 확인할 수 있습니다.

### IAM Role 설정

> Blue/Green 배포에 사용할 권한의 추가 지정이 필요합니다.

1. IAM의 Roles 항목으로 가서, CodeStar가 만들어놓은 CodeDeploy Role을 선택합니다.
1. Attach Policy 버튼을 누르고, AutoScalingConsoleFullAccess 권한과, AmazonEC2FullAccess 권한을 추가합니다.

### CodeDeploy 설정 변경

> CodeStar에서 만들어놓은 CodeDeploy 설정이 In-place로 되어있기 때문에, CodeDeploy 설정 변경이 필요합니다.

1. CodeStar 대시보드에서 CodeDeploy 탭으로 들어갑니다.
1. Deployment groups에 기존에 생성되어 있는 Env를 선택하고, Edit 화면으로 들어갑니다.
1. In-place deployment를 Blue/green deployment로 변경합니다.
1. Automatically copy Auto Scaling group을 선택하고, 위에서 생성한 Auto Scaling group을 선택합니다.
1. Load balancer 항목에서 위에서 생성한 ALB를 선택합니다.
1. Edit deployment settings에서 배포설정을 변경할 수 있습니다.
1. Advanced 항목에서 Rollback 정책 등을 설정할 수 있습니다.
1. 변경된 설정을 저장합니다.

### CodePipeline 에서 배포해보기

> CodeStar에서 CodePipeline으로 들어가서, Release change 버튼을 눌러서 배포를 해봅니다.  
> 에러없이 끝났다면, 새로운 Auto Scaling Group이 생겼고, ALB에도 등록되어 있는 걸 확인할 수 있습니다.

### 마치며..

> CodeStar를 이용하면, CI/CD 구성을 손쉽게 할 수 있습니다. 단, Blue/Green 배포 설정을 하다가 꽤 많은 삽질을 했었는데, 크게 아래 두가지 문제가 있었습니다.

1. Auto Scaling Group 설정을 해야해서, 별도로 Auto Scaling Group을 생성하고 했었는데, 이미 만들어진 EC2에서 새롭게 만들어야 했습니다.
1. 기본으로 설정된 In-place 보다 Blue/Green에서 필요한 권한이 더 있었습니다. IAM Role에 CodeDeploy 쪽에 추가로 권한을 넣어서 해결했습니다.

> 테스트로 생성했던 관련 Resource들을 지우려면, CodeDeploy 에서 In-place 설정으로 되돌리고, Auto Scaling Group과 ALB 삭제 후, CodeStar에서 Delete를 해주면 자동으로 생성됐던 것들이 모두 삭제됩니다.   
주의할 점은 혹시라도 CodeCommit에 올려두었던 소스가 있다면, 자동으로 생성된 CodeCommit 저장소도 같이 삭제되므로 꼭 백업을 해두셔야 합니다.