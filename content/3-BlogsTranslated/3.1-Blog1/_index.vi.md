---
title: "Blog 1"
date: 2026-04-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Containers và cơ sở hạ tầng dưới dạng mã — như bơ đậu phộng và mứt

> by Clare Liguori ┃ on 18 OCT 2019 ┃ in [Amazon EC2 Container Registry](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-registry/), [Amazon EC2 Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-service/), [Amazon Elastic Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-elastic-container-service/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-kubernetes-service/), [AWS Cloud Development Kit](https://aws.amazon.com/blogs/containers/category/developer-tools/aws-cloud-development-kit/), [AWS CloudFormation](https://aws.amazon.com/blogs/containers/category/management-tools/aws-cloudformation/), [AWS Fargate](https://aws.amazon.com/blogs/containers/category/compute/aws-fargate/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/)

Các công cụ cơ sở hạ tầng dưới dạng mã như AWS CloudFormation và HashiCorp Terraform cho phép các nhóm mô tả và tự động hoá việc cấp phát các tài nguyên hạ tầng đám mây, bao gồm các tài nguyên liên quan đến container như dịch vụ Amazon ECS và cụm Amazon EKS. Trong bài viết này, tôi trình bày lý do tôi tin rằng cơ sở hạ tầng dưới dạng mã đặc biệt quan trọng đối với các ứng dụng container hóa, cách chúng tôi sử dụng cơ sở hạ tầng dưới dạng mã với containers tại Amazon, hướng phát triển của các công cụ này, và các công cụ chúng tôi đang xây dựng cho Amazon ECS để giúp bạn áp dụng được điều đó.

## Cơ sở hạ tầng dưới dạng mã và containers: tốt hơn khi kết hợp

Cơ sở hạ tầng dưới dạng mã và containers kết hợp rất tốt với nhau (như bơ đậu phộng và mứt!) khi phát triển ứng dụng triển khai trên đám mây. Thoạt nhìn, một ảnh container dường như là một ứng dụng tự chứa hoàn chỉnh: nó có toàn bộ mã và các phụ thuộc phần mềm cần thiết để chạy ứng dụng cả trên máy tính cá nhân và trên đám mây. Tôi có thể chia sẻ ảnh đó để người khác cũng dễ dàng chạy ứng dụng của tôi. Tuy nhiên, khi tôi triển khai và vận hành ảnh đó trên đám mây, tôi cần nhiều cấu hình hơn để mở rộng, làm cho nó đáng tin cậy và dễ quan sát. Tôi có thể cần cấu hình một bộ điều phối container như ECS hoặc Kubernetes để giữ một số bản sao nhất định của ảnh đang chạy, và có thể cần các hạ tầng khác như load balancer, mục nhập DNS, chứng chỉ TLS, bảng điều khiển (dashboards), cảnh báo và logging. Ứng dụng container hóa của tôi trên đám mây có thể trông giống như sơ đồ này, trong đó ảnh container chỉ là một phần của toàn bộ ứng dụng:

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/complex_cloud_infrastructure.png)

Khi đã triển khai lên đám mây, ảnh container một mình không còn mô tả đầy đủ ứng dụng nữa. Việc chia sẻ ảnh với người khác không có nghĩa là người khác có thể dễ dàng chạy ứng dụng của tôi trên đám mây, vì họ sẽ phải tái tạo toàn bộ hạ tầng xung quanh ảnh đó. Ứng dụng hoàn chỉnh thực sự được mô tả tốt nhất bằng sự kết hợp giữa ảnh container và một mẫu cơ sở hạ tầng dưới dạng mã chứa toàn bộ cấu hình này. Một số “trình quản lý gói” cho ứng dụng container như [Helm](https://helm.sh/) trở nên phổ biến vì chúng giúp bạn chia sẻ ứng dụng container hóa với người khác bằng cách đóng gói cả ảnh lẫn mẫu cấu hình.

Cơ sở hạ tầng dưới dạng mã không chỉ quan trọng để chia sẻ ứng dụng container hóa, mà còn để phát hành (release) chúng. Lời hứa ban đầu của containers là rằng ảnh container gói mọi thứ cần để chạy ứng dụng, vì vậy ứng dụng sẽ hành xử giống nhau ở nhiều môi trường khác nhau, ví dụ môi trường “Dev” và “Prod”. Ảnh gói các phụ thuộc của ứng dụng như thư viện hệ thống, nên nó lẽ ra chạy giống hệt trong Dev và Prod, đúng không? Không còn phải đoán hay hy vọng ứng dụng sẽ hành xử giống nhau khi triển khai lên một tập hợp instance khác trong Prod! Nhưng khi thêm toàn bộ hạ tầng như đã mô tả phía trên, thì việc có cùng ảnh chạy ở nhiều môi trường không còn đủ: quan trọng là phải có cùng cấu hình hạ tầng chính xác giữa các môi trường để ứng dụng hành xử giống nhau.

Sự khác biệt nhỏ về hạ tầng giữa các môi trường có thể ảnh hưởng đáng kể tới hành vi và độ tin cậy của ứng dụng container hóa. Hình dưới cho thấy ví dụ khi tôi cấu hình đường dẫn kiểm tra sức khỏe (health check) khác nhau trên load balancer ở Dev và Prod — điều dễ bỏ sót nếu tôi cấu hình load balancer thủ công. Khi tôi triển khai ảnh lên Dev, ứng dụng hoạt động đúng và các bài kiểm thử end-to-end thành công ở đó. Nhưng khi triển khai cùng ảnh lên Prod, load balancer có thể gửi traffic đến các container không khỏe mạnh vì nó dùng đường dẫn health check sai, gây lỗi cho ứng dụng. Tôi có thể tránh vấn đề này bằng cách sử dụng cơ sở hạ tầng dưới dạng mã. Cơ sở hạ tầng dưới dạng mã giúp các thay đổi hạ tầng trở nên lặp lại và dự đoán được trên nhiều giai đoạn trong quy trình phát hành, như Dev và Prod. Nó cũng giúp tái tạo môi trường staging càng giống production càng tốt, để tôi có thể tự tin triển khai cả thay đổi hạ tầng và thay đổi mã lên Prod sau khi đã kiểm thử ở Dev.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/diff_between_dev_and_prod.png)

## Cơ sở hạ tầng dưới dạng mã với containers tại Amazon

Tự động hóa cấp phát hạ tầng cho microservices rất quan trọng cho năng suất của nhà phát triển tại Amazon. Cả kiến trúc microservices và cơ sở hạ tầng dưới dạng mã đều là các thực hành tốt (best practices) cho những gì chúng tôi gọi là "[ứng dụng hiện đại](https://aws.amazon.com/modern-apps/)", và chúng tôi đã lần lượt áp dụng cả hai tại Amazon. Khi trước đây chúng tôi có một monolith lớn phải được dựng lên và cấu hình thủ công, việc tự động hóa provisioning trở nên cần thiết khi chúng tôi chuyển sang kiến trúc microservices. Việc dựng hạ tầng mới thủ công cho hàng chục, hàng trăm hay hàng ngàn microservices trong một ứng dụng hiện đại trở nên phức tạp hơn, dễ lỗi hơn và mất thời gian hơn. Tự động hoá cấu hình và provisioning qua các mẫu (templates) nghĩa là các nhà phát triển của chúng tôi có thể dành nhiều thời gian hơn để xây tính năng mới, thay vì làm cấu hình thủ công. Hơn nữa, khi chúng tôi bắt đầu triển khai các mẫu cơ sở hạ tầng dưới dạng mã qua pipeline CI/CD, các nhà phát triển không còn phải chạy triển khai mẫu thủ công; tất cả những gì họ phải làm giờ chỉ là “git push.” CTO của Amazon, Werner Vogels, kể thêm câu chuyện hành trình áp dụng các thực hành ứng dụng hiện đại tại Amazon [trên blog của ông.](https://www.allthingsdistributed.com/2019/08/modern-applications-at-aws.html)

Đối với các ứng dụng container hóa nội bộ, thực hành tốt của chúng tôi là triển khai cả thay đổi mã và thay đổi hạ tầng microservice thông qua cùng một mẫu cơ sở hạ tầng dưới dạng mã và cùng pipeline CI/CD. Ví dụ đơn giản của quy trình phát hành của chúng tôi được minh họa dưới đây. Mẫu cơ sở hạ tầng chứa cả cấu hình liên quan đến container (ví dụ: định nghĩa task ECS và service ECS) và hạ tầng của microservice (ví dụ: load balancer). Ở giai đoạn “build” của pipeline, ảnh container được build và push, và ID duy nhất cho ảnh mới được chèn vào mẫu cơ sở hạ tầng dưới dạng mã. Mỗi giai đoạn của pipeline như “Dev” và “Prod” sau đó triển khai cùng một mẫu đó. Thực hành này mang lại cho chúng tôi sự tự tin rằng việc triển khai toàn bộ ứng dụng là có thể lặp lại và kiểm thử được, và chúng tôi có tầm nhìn đầy đủ trong pipeline về cả mã ứng dụng chính xác và mã hạ tầng chính xác đang được triển khai ở môi trường production.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/Infra_as_Code_and_Containers_blog_post_diagrams.png)

## Giai đoạn tiếp theo: kiến trúc dưới dạng mã

Tôi tin rằng chúng ta sẽ ngày càng thấy các công cụ cơ sở hạ tầng dưới dạng mã giúp các nhà phát triển nghĩ vượt ra ngoài việc mô hình hóa và cấp phát các tài nguyên hạ tầng đơn lẻ, thay vào đó nghĩ theo hướng mô hình hóa và cấp phát kiến trúc đám mây. Là nhà phát triển, khi thiết kế ứng dụng đám mây, chúng ta thường không nghĩ theo các tài nguyên hạ tầng cụ thể. Chúng ta nghĩ theo kiến trúc: vẽ các hộp và mũi tên ở mức cao trên bảng trắng để mô tả microservices và tương tác giữa các dịch vụ, chứ không phải các chi tiết như load balancer hay security group. Hình dưới là một sơ đồ kiến trúc mà tôi đã vẽ gần đây cho một ứng dụng nhỏ tôi đang xây để tổng hợp lượt bầu chọn. Nó giống nhiều thiết kế trên bảng trắng: ở mức cao và không đi sâu vào hạ tầng bên trong mỗi hộp microservice.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/whiteboard_drawing.png)

Nhiều hộp microservice trong các sơ đồ trên bảng trắng thường theo các mẫu “loại” hoặc “kiểu” giống nhau: một dịch vụ API đằng sau load balancer, một dịch vụ kéo công việc từ queue, một công việc theo lịch chạy mỗi ngày, v.v. Hạ tầng cho một loại microservice, như một “API service,” thường trông giống nhau giữa nhiều API service. Thông thường nếu so sánh các mẫu cơ sở hạ tầng dưới dạng mã cho tất cả API service của chúng tôi, chỉ có một vài thuộc tính thay đổi giữa các mẫu, ví dụ ảnh container và quy mô dịch vụ.

Khi chúng ta phát hiện các mẫu trong hạ tầng đại diện cho các loại dịch vụ khác nhau, những mẫu này có thể được đóng gói thành các thành phần tái sử dụng (reusable components) và chia sẻ giữa nhiều microservice. Khi đó các nhà phát triển có thể mô tả kiến trúc mức cao của họ trong một mẫu: họ chỉ cần chỉ định “loại” microservice họ đang xây và cách nó liên kết với các microservice khác trong kiến trúc, thay vì phải chỉ định từng tài nguyên hạ tầng lặp đi lặp lại trong một mẫu mới cho mỗi microservice. Khi cần thay đổi cấu hình cho một loại microservice, ví dụ thay đổi đường dẫn health check cho một “API service,” thay đổi đó có thể được thực hiện trong thành phần tái sử dụng và ngay lập tức triển khai cho mỗi microservice sử dụng mẫu đó, mà không cần chỉnh sửa từng mẫu microservice.

Tôi tin rằng chúng ta sẽ thấy nhiều công cụ “kiến trúc dưới dạng mã” xuất hiện hơn, cho phép các nhà phát triển mô tả microservices của họ theo các mẫu hạ tầng mức cao. Công cụ sẽ lo liệu phần kết hợp các thành phần mẫu hạ tầng với mô tả microservice của nhà phát triển để tự động cấp phát các tài nguyên hạ tầng cần thiết, như cấu hình bộ điều phối container, load balancers, security groups và IAM roles. Ngày nay, đã có nhiều lựa chọn để tạo và phân phối các mẫu hạ tầng giữa các microservice và các đội để thực hành “kiến trúc dưới dạng mã” trong tổ chức của bạn, như [CloudFormation macros](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-macros.html), [AWS Cloud Development Kit constructs](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_author), [Terraform modules](https://developer.hashicorp.com/terraform/language/modules), và [Kubernetes custom resource definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions).

## Kiến trúc dưới dạng mã với Amazon ECS và AWS CDK

Trong đội AWS Containers năm ngoái, chúng tôi bắt đầu khám phá các kiểu microservice thường chạy trên Amazon ECS và những mẫu đó trông như thế nào về mặt tài nguyên hạ tầng. Chúng tôi muốn giúp các đội dễ dàng áp dụng mô hình kiến trúc dưới dạng mã. Trong mô hình này, các nhà phát triển chỉ cần chỉ định loại dịch vụ container họ muốn chạy và Dockerfile của họ, và tất cả tài nguyên hạ tầng bao gồm VPC, subnet, cấu hình ECS, registry ECR, load balancer, v.v. sẽ được cấp phát tự động. Họ sẽ không cần biết chính xác những tài nguyên nào cần cấp phát hay cách kết nối chúng với nhau. Chúng tôi xác định ba mẫu microservice phổ biến ban đầu cho ứng dụng container hóa mà muốn xây dựng công cụ kiến trúc dưới dạng mã xung quanh: API Service, Queue Processor, và Scheduled Job (tất cả đều thấy trong sơ đồ ứng dụng voting ở trên).

Chúng tôi chọn phân phối tập các mẫu kiến trúc ECS ban đầu của mình trong [AWS Cloud Development Kit (AWS CDK)](https://aws.amazon.com/cdk/). AWS CDK là một framework mã nguồn mở để mô hình hóa tài nguyên ứng dụng đám mây bằng các ngôn ngữ lập trình quen thuộc như TypeScript, Python, Java và C#, rồi cấp phát chúng bằng CloudFormation. Trong CDK, chúng tôi xây dựng module [“ECS Patterns”](https://docs.aws.amazon.com/cdk/api/v2/), kết hợp các tài nguyên thấp hơn thành các kiểu tài nguyên trừu tượng mức cao đại diện cho một microservice container hóa hoàn chỉnh. Chúng tôi bắt đầu bằng việc phát hành ba mẫu cho ECS: Load Balanced Service, Queue Processing Service, và Scheduled Tasks.

Với những mẫu này, tôi có thể mô hình hóa và cấp phát sơ đồ kiến trúc ứng dụng voting ở trên, mà không cần đi sâu vào chi tiết từng cấu hình tài nguyên hạ tầng. Tôi có thể mô tả toàn bộ dịch vụ Vote API chỉ với 36 dòng mã TypeScript, thay vì 775 dòng template CloudFormation thô! Lớp “ApplicationLoadBalancedFargateService” dưới đây là một trong các mẫu ECS trong CDK. Với đoạn mã này, tôi có được một dịch vụ ECS chạy trên Fargate, một Application Load Balancer, một cụm ECS, một repository ECR chứa ảnh của tôi, một VPC, một mục DNS, và một chứng chỉ TLS, tất cả đã được liên kết sẵn.

```
import { ContainerImage } from '@aws-cdk/aws-ecs';
import { ApplicationLoadBalancedFargateService } from '@aws-cdk/aws-ecs-patterns';
import { ApplicationProtocol } from '@aws-cdk/aws-elasticloadbalancingv2';
import { HostedZone } from '@aws-cdk/aws-route53';
import cdk = require('@aws-cdk/core');
import path = require('path');

// Stack with a Fargate service + load balancer serving https://api.my-vote-app.com
class VoteApiStack extends cdk.Stack {
	constructor(parent: cdk.App, name: string, props: cdk.StackProps) {
		super(parent, name, props);

		// Domain name info
		const domainName = 'api.my-vote-app.com';
		const domainZone = HostedZone.fromLookup(this, 'Zone', {
			domainName: 'my-vote-app.com'
		});

		new ApplicationLoadBalancedFargateService(this, 'Service', {
			protocol: ApplicationProtocol.HTTPS,
			domainName,
			domainZone,
			taskImageOptions: {
				// build and push image using Dockerfile located in vote-api directory
				image: ContainerImage.fromAsset(path.resolve(__dirname, '../vote-api/')),
			},
			desiredCount: 3
		});
	}
}

const app = new cdk.App();
new VoteApiStack(app, 'VoteApiService', {
	env: { account: process.env['CDK_DEFAULT_ACCOUNT'], region: process.env['CDK_DEFAULT_REGION'] }
});
app.synth();
```

## Kết luận

Bài viết này đã trình bày lợi ích của cơ sở hạ tầng dưới dạng mã cho các ứng dụng container hóa, các thực hành tốt của Amazon trong việc triển khai containers bằng cơ sở hạ tầng dưới dạng mã, khái niệm “kiến trúc dưới dạng mã”, và một ví dụ về kiến trúc dưới dạng mã với Amazon ECS và AWS CDK. Để thử áp dụng kiến trúc dưới dạng mã với ứng dụng container hóa trên AWS, hãy xem [các mẫu ECS của AWS CDK](https://docs.aws.amazon.com/cdk/api/v2/).

Chúng tôi tiếp tục khám phá cách giúp khách hàng AWS Containers sử dụng cơ sở hạ tầng dưới dạng mã và kiến trúc dưới dạng mã thông qua AWS CDK và các công cụ khác. Gần đây chúng tôi đã đưa ra một [đề xuất công khai](https://github.com/aws/containers-roadmap/issues/513) với kế hoạch cho công cụ CLI ECS mới được thiết kế quanh mô hình kiến trúc dưới dạng mã (rất hoan nghênh phản hồi!). Chúng tôi rất muốn nghe phản hồi của bạn về các mẫu CDK ECS [trên GitHub](https://github.com/aws/aws-cdk/issues/new/choose). Hãy cho chúng tôi biết những mẫu còn thiếu, ví dụ những kiểu kiến trúc và mẫu microservice nào bạn thấy trong ứng dụng container hóa của bạn mà có thể được đóng gói thành các thành phần cơ sở hạ tầng tái sử dụng. Và cho chúng tôi biết bạn muốn thấy công cụ hoặc tích hợp cơ sở hạ tầng dưới dạng mã nào khác từ đội Containers trên [Bản đồ lộ trình AWS Containers](https://github.com/aws/containers-roadmap).

> THẺ: [Amazon ECS](https://aws.amazon.com/blogs/containers/tag/amazon-ecs/), [Amazon EKS](https://aws.amazon.com/blogs/containers/tag/amazon-eks/), [cơ sở hạ tầng dưới dạng mã](https://aws.amazon.com/blogs/containers/tag/infrastructure-as-code/)

