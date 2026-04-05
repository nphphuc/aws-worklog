---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Xây dựng và triển khai ứng dụng Spring Boot lên AWS App Runner với CI/CD pipeline sử dụng Terraform

> by Irshad Buchh and Yang Xiao ┃ on 05 NOV 2021 ┃ in [Amazon EC2 Container Registry](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-registry/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/), [Technical How-to](https://aws.amazon.com/blogs/containers/category/post-types/technical-how-to/)

## Giới thiệu

[Spring Boot](https://spring.io/projects/spring-boot) là một framework mã nguồn mở hàng đầu để xây dựng các ứng dụng web dựa trên Java. Nó được thiết kế để giúp bạn khởi chạy và chạy nhanh nhất có thể với cấu hình tối thiểu. Quan điểm “có sẵn cho production” của Spring Boot làm cho việc áp dụng các thực hành tốt hiện đại trở nên trực quan và dễ dàng.

[AWS App Runner](https://aws.amazon.com/apprunner/) là dịch vụ ứng dụng container được quản lý hoàn toàn, giúp khách hàng không có kinh nghiệm về container hoặc hạ tầng dễ dàng xây dựng, triển khai và chạy các ứng dụng web và API container hóa. Khách hàng chỉ cần cung cấp mã nguồn hoặc ảnh container, App Runner tự động build và triển khai ứng dụng web, cân bằng tải và tự động scale theo nhu cầu.

Trong bài đăng này, chúng tôi sẽ hướng dẫn cách chạy một ứng dụng Spring Boot ở quy mô trên AWS App Runner và thiết lập pipeline để tự động build và triển khai. Ứng dụng mẫu sử dụng trong bài là Spring PetClinic. Ứng dụng PetClinic được chọn để minh họa cách dùng Spring Boot, Spring MVC và Spring Data để xây dựng một ứng dụng hướng cơ sở dữ liệu đơn giản nhưng mạnh mẽ. Ứng dụng PetClinic lưu dữ liệu trong [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds/). Hạ tầng được provision và quản lý bằng [Terraform](https://developer.hashicorp.com/terraform).

## Kiến trúc

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner1.png)

## Yêu cầu trước

Trước khi bạn xây dựng toàn bộ hạ tầng, bao gồm pipeline CI/CD, bạn cần đảm bảo các yêu cầu sau.

### 1. Tài khoản AWS

Đảm bảo bạn có quyền truy cập vào một tài khoản AWS và một bộ credential có quyền administrator. Lưu ý: Trong môi trường production, chúng tôi khuyến nghị hạn chế quyền xuống mức tối thiểu cần thiết để vận hành pipeline.

### 2. Tạo môi trường AWS Cloud9

Đăng nhập vào [AWS Management Console](https://aws.amazon.com/console/) và tìm dịch vụ [AWS Cloud9](https://console.aws.amazon.com/cloud9/home?region=us-east-1) trong thanh tìm kiếm. Chọn AWS Cloud9 và tạo một môi trường AWS Cloud9 trong vùng `us-east-1` dựa trên Amazon Linux 2.

## Giải pháp

### Cấu hình môi trường AWS Cloud9

Khởi chạy IDE AWS Cloud9. Đóng tab `Welcome` và mở tab `Terminal` mới.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner2.png)

### Tạo và gắn IAM role cho phiên AWS Cloud9 của bạn

Theo mặc định, AWS Cloud9 quản lý credential IAM tạm thời cho bạn. Không may là credential tạm thời này không tương thích với Terraform. Để khắc phục, bạn cần tắt credential tạm thời của Cloud9 và tạo rồi gắn một IAM role cho instance Cloud9 của bạn.

  1. Làm theo [liên kết sâu này để tạo một IAM role với quyền Administrator.](https://console.aws.amazon.com/iam/home#/roles$new?step=review&commonUseCase=EC2%2BEC2&selectedUseCase=EC2&policies=arn:aws:iam::aws:policy%2FAdministratorAccess)
  2. Xác nhận rằng AWS service và EC2 được chọn, rồi chọn **Next** để xem permissions.
  3. Xác nhận AdministratorAccess được đánh dấu, rồi chọn **Next: Tags** để gán tags.
  4. Giữ mặc định, rồi chọn **Next: Review** để xem lại.
  5. Nhập tên `workshop-admin` cho Role, và chọn **Create role**.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner3.png)

  6. Làm theo [liên kết sâu này để tìm instance EC2 của AWS Cloud9.](https://console.aws.amazon.com/ec2/v2/home#Instances:tag:Name=aws-cloud9-;sort=desc:launchTime)
  7. Chọn instance, rồi chọn **Actions / Instance settings / Modify IAM role**. Lưu ý: Nếu bạn không thấy tùy chọn này, hãy tìm dưới **Actions / Security / Modify IAM role**

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner4.png)

  8. Chọn `workshop-admin` từ dropdown IAM role, và chọn **Apply**.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner5.png)

  9. Trở lại workspace và mở **Preferences** (biểu tượng bánh răng, góc phải trên) hoặc mở tab mới và chọn **Open preferences**.
  10. Chọn **AWS settings**.
  11. Tắt AWS managed temporary credentials.
  12. Đóng tab Preferences.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner6.png)

  13. Trong pane terminal của AWS Cloud9, chạy lệnh:

  ```
  rm -vf ${HOME}/.aws/credentials
  ```

  14. Kiểm tra cuối cùng bằng cách dùng lệnh CLI `GetCallerIdentity` để xác nhận rằng IDE Cloud9 đang sử dụng IAM role đúng:

  ```
  aws sts get-caller-identity --query Arn | grep workshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
  ```

### Nâng cấp awscli

Đảm bảo bạn đang chạy phiên bản mới nhất của AWS CLI:
```
aws ——version
pip install awscli —upgrade —user
```

Chạy `aws configure` để cấu hình region. Để trống các trường còn lại. Bạn sẽ có dạng:
```
admin:~/environment $ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: us-east-1
Default output format [None]:
```

### Cài Terraform
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
```

Xác nhận bạn có thể chạy Terraform:
```
terraform version
```

### Cài Apache Maven
```
cd /tmp
sudo wget https://www-eu.apache.org/dist/maven/maven-3/3.8.1/binaries/apache-maven-3.8.1-bin.tar.gz
sudo tar xf /tmp/apache-maven-*.tar.gz -C /opt
sudo ln -s /opt/apache-maven-3.8.1 /opt/maven
```

### Thiết lập Apache Maven
```
sudo nano ~/.bashrc
```

Chèn các dòng sau vào cuối file:
```
export M2_HOME=/opt/maven
export MAVEN_HOME=/opt/maven
export PATH=${M2_HOME}/bin:${PATH}
```

Xác nhận cài đặt Maven:
```
source ~/.bashrc
mvn ——version
```

### Clone repository workshop
Clone mã nguồn:
```
cd ~/environment
git clone https://github.com/aws-samples/aws-apprunner-terraform.git
```

## Package ứng dụng bằng Apache Maven
```
cd ~/environment/aws-apprunner-terraform/petclinic
mvn package -Dmaven.test.skip=true
```

Lần đầu chạy lệnh này, Maven sẽ cần tải plugin và dependency phục vụ cho lệnh. Trên một cài đặt Maven mới, điều này có thể mất thời gian (ví dụ: gần năm phút). Nếu chạy lại lệnh, Maven sẽ dùng cache và chạy nhanh hơn.

Các lớp Java biên dịch sẽ đặt tại `spring-petclinic/target/classes`, là convention chuẩn của Maven. Bằng cách tuân thủ convention chuẩn, POM giữ nhỏ gọn và bạn không cần chỉ rõ nơi nguồn hay nơi output.

## Build và tag ảnh Docker của Petclinic
Từ thư mục petclinic:
```
docker build -t petclinic .
```

## Chạy ứng dụng Petclinic cục bộ
Trong terminal AWS Cloud9, chạy:
```
docker run -it --rm -p 8080:80 --name petclinic petclinic
```

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner7.png)

Lệnh trên chạy ứng dụng dùng cổng container 80 và map ra cổng host 8080. Chọn **Preview** từ menu trên cùng rồi **Preview Running Application** để mở trình duyệt hiển thị ứng dụng Spring Petclinic.

## Push ảnh petclinic lên Amazon ECR
Trong terminal AWS Cloud9, chạy:
```
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' ——output text)
AWS_REGION=$(aws configure get region)

export REPOSITORY_NAME=petclinic
export IMAGE_NAME=petclinic

aws ecr create-repository \
--repository-name $REPOSITORY_NAME \
--image-scanning-configuration scanOnPush=true \
--region $AWS_REGION

aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

docker tag $IMAGE_NAME $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_NAME
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_NAME
```

## Xây dựng hạ tầng và pipeline
Chúng ta sẽ dùng Terraform để xây dựng kiến trúc ở trên, bao gồm cả [AWS CodePipeline](https://aws.amazon.com/codepipeline/).

Lưu ý: Workshop này sẽ tạo các tài nguyên có thể bị tính phí trong tài khoản của bạn. Khi hoàn tất, hãy đảm bảo dọn dẹp tài nguyên như hướng dẫn ở phần cuối.

### Thiết lập tham số SSM cho mật khẩu DB
```
aws ssm put-parameter --name /database/password --value mysqlpassword —type SecureString
```

### Chỉnh các biến Terraform
```
cd ~/environment/aws-apprunner-terraform/terraform 
```

### Build
Khởi tạo Terraform:
```
terraform init
```

Build hạ tầng và pipeline bằng Terraform:
```
terraform apply
```

Terraform sẽ hiển thị action plan. Khi được hỏi có muốn tiếp tục, nhập `yes`.
Chờ Terraform hoàn tất việc tạo — sẽ mất vài phút cho `terraform apply`.

### Khám phá stack bạn đã tạo
Khi build xong, bạn có thể duyệt môi trường qua AWS console:

- Xem RDS qua [Amazon RDS console](https://console.aws.amazon.com/rds).
- Xem repo ECR qua [Amazon ECR console](https://console.aws.amazon.com/ecr).
- Xem repo CodeCommit qua [AWS CodeCommit console](https://console.aws.amazon.com/codecommit).
- Xem project CodeBuild qua [AWS CodeBuild console](https://console.aws.amazon.com/codebuild).
- Xem pipeline qua [AWS CodePipeline console](https://console.aws.amazon.com/codepipeline).

Lưu ý pipeline có thể bắt đầu ở trạng thái failed vì repo CodeCommit chưa có mã! Ở bước tiếp theo, bạn sẽ push app petclinic vào repo để kích hoạt pipeline.

### Khám phá App Runner service
Chi tiết service App Runner nằm trong `~/environment/aws-apprunner-terraform/terraform/services.tf`:
```
image_repository {
      image_configuration {
        port = var.container_port
        runtime_environment_variables = {
          "spring.datasource.username" : "${var.db_user}",
          "spring.datasource.password" : "${data.aws_ssm_parameter.dbpassword.value}",
          "spring.datasource.initialization-mode" : var.db_initialize_mode,
          "spring.profiles.active" : var.db_profile,
          "spring.datasource.url" : "jdbc:mysql://${aws_db_instance.db.address}/${var.db_name}"
        }
      }
      image_identifier      = "${data.aws_ecr_repository.image_repo.repository_url}:latest"
      image_repository_type = "ECR"
    }
```

## Triển khai ứng dụng petclinic bằng pipeline
Giờ bạn sẽ dùng git để push petclinic qua pipeline.

### Thiết lập git repo local cho petclinic
Chuyển vào thư mục `petclinic`:
```
cd ~/environment/aws-apprunner-terraform/petclinic
```

Cấu hình tên và email git của bạn:
```
git config --global user.name "Your Name"
git config --global user.email you@example.com
```

Tạo git repo local:
```
git init
git add .
git commit -m "Baseline commit"
```

### Thiết lập remote CodeCommit
Một repo AWS CodeCommit đã được tạo trong quá trình build. Giờ bạn cấu hình nó làm remote cho repo local petclinic.

Để xác thực, bạn có thể dùng IAM git credential helper để sinh credential dựa trên IAM role. Chạy:
```
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
```

Từ output của Terraform, lưu giá trị `source_repo_clone_url_http`:
```
cd ~/environment/aws-apprunner-terraform/terraform 
export tf_source_repo_clone_url_http=$(terraform output -raw source_repo_clone_url_http)
```

Thiết lập remote:
```
cd ~/environment/aws-apprunner-terraform/petclinic 
git remote add origin $tf_source_repo_clone_url_http
git remote -v
```

Bạn sẽ thấy dạng:
```
origin  https://git-codecommit.eu-west-2.amazonaws.com/v1/repos/petclinic (fetch)
origin  https://git-codecommit.eu-west-2.amazonaws.com/v1/repos/petclinic (push)
```

### Kích hoạt pipeline
Để kích hoạt pipeline, push branch master:
```
git push -u origin master
```

Pipeline sẽ kéo code, build ảnh docker, push lên ECR và deploy lên Amazon ECS cluster. Việc này mất vài phút. Bạn có thể theo dõi pipeline trên [AWS CodePipeline console](https://console.aws.amazon.com/codepipeline).

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner8.png)

### Kiểm thử ứng dụng
Từ output của Terraform, lưu giá trị `apprunner_service_url`:
```
cd ~/environment/aws-apprunner-terraform/terraform
export apprunner_service_url=$(terraform output -raw apprunner_service_url)
echo "https://$apprunner_service_url"
```

Dùng URL này trong trình duyệt để truy cập ứng dụng.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner9.png)

## Đẩy thay đổi qua pipeline và kiểm thử lại
Pipeline giờ có thể dùng để deploy bất kỳ thay đổi nào. Bạn có thể thử chỉnh thông điệp chào như sau:
```
cd ~/environment/aws-apprunner-terraform/petclinic 
vi src/main/resources/messages/messages.properties
```

Thay giá trị welcome, ví dụ thành “Hello.” Commit thay đổi:
```
git add .
git commit -m "Changed welcome string"
```

Push thay đổi để kích hoạt pipeline:
```
git push origin master
```

Như trước, bạn có thể quan sát tiến trình trong console. Khi hoàn tất, xác nhận ứng dụng hoạt động với thông điệp welcome đã thay đổi.

## Dọn dẹp
### Gỡ stack
Hãy nhớ teardown stack khi xong để tránh phí. Dọn tài nguyên bằng:
```
cd cd ~/environment/aws-apprunner-terraform/terraform
terraform destroy
```

Khi được hỏi, nhập `yes` để cho phép xóa stack. Sau khi xong, bạn sẽ cần thủ công empty và delete S3 bucket dùng cho pipeline.

### Xóa repository Amazon ECR
```
aws ecr delete-repository \
    --repository-name $REPOSITORY_NAME \
    --region $AWS_REGION \
    --force
```

## Kết luận
AWS App Runner giúp chạy ứng dụng web an toàn ở quy mô lớn mà không cần kinh nghiệm về hạ tầng như cấu hình server, networking, load balancing, autoscaling hay deployment an toàn. Các tài nguyên và thành phần hạ tầng được provision cho ứng dụng của bạn được AWS quản lý toàn diện và hưởng các thực hành bảo mật và vận hành tốt nhất, cho phép bạn tập trung hoàn toàn vào ứng dụng.

Dự án ví dụ trong bài minh họa cách thiết lập chạy ứng dụng Spring Boot trên App Runner, đồng thời là bước khởi đầu tốt để phát triển các ứng dụng web dựa trên Spring. Tham khảo [tài liệu AWS App Runner](https://docs.aws.amazon.com/apprunner/) để biết thêm cách dùng App Runner, và [AWS App Runner developer guide](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html) để xem hướng dẫn bắt đầu, best practices và khắc phục sự cố. Bạn cũng có thể xem roadmap và gợi ý tính năng trên [AWS App Runner Roadmap](https://github.com/aws/apprunner-roadmap) trên GitHub.

> THẺ: [AWS App Runner](https://aws.amazon.com/blogs/containers/tag/app-runner/), [Terraform](https://aws.amazon.com/blogs/containers/tag/terraform/)

