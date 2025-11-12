### Lời mở đầu: Tuyên bố của Chuyên gia

Chào mừng bạn đến với hướng dẫn toàn diện về HashiCorp Terraform. Tài liệu này không chỉ là một bản tóm tắt cú pháp; nó là một tài liệu có tính định hướng (opinionated), được đúc kết từ nhiều năm kinh nghiệm vận hành các hệ thống cơ sở hạ tầng quy mô lớn, phức tạp trong môi trường doanh nghiệp.

Trọng tâm của chúng tôi sẽ là tính an toàn (safety), khả năng tái sử dụng (reusability), và quản trị (governance).

Triết lý cốt lõi của Terraform là **khai báo (declarative)**. Bạn không ra lệnh cho Terraform _làm thế nào_ để tạo ra một máy chủ (đó là tư duy _imperative_ của shell script). Thay vào đó, bạn _khai báo_ **"trạng thái mong muốn" (desired state)** của mình—ví dụ: "Tôi muốn có một máy chủ t3.micro với port 80 mở". Terraform sẽ tự mình phân tích "trạng thái hiện tại" (current state) (ban đầu là "không có gì"), so sánh nó với "trạng thái mong muốn", và tạo ra một "kế hoạch" (plan) để thu hẹp khoảng cách đó.

Kỹ năng quan trọng nhất bạn sẽ học là **đọc, hiểu, và tin tưởng vào `plan`**. Trong một doanh nghiệp, một `plan` sai lầm có thể xóa sổ cơ sở dữ liệu production. Tài liệu này được thiết kế để giúp bạn không bao giờ mắc phải sai lầm đó. Chúng ta sẽ đi từ việc tạo một tài nguyên duy nhất đến việc điều phối các hệ thống đa đám mây (multi-cloud) được quản trị bằng chính sách (policy-driven).

---

## I. CƠ BẢN (Beginner): Nền tảng và Tài nguyên đầu tiên

Mục tiêu của cấp độ này là đưa bạn từ "zero" đến việc có thể chạy `terraform apply` một cách an toàn và hiểu được điều gì đang xảy ra. Chúng ta tập trung vào vòng đời cơ bản và các khối xây dựng cốt lõi.

### 1. Mục tiêu học tập

- Hiểu Infrastructure as Code (IaC) là gì và tại sao nó vượt trội so với "click-ops" (nhấp chuột trên giao diện web).
    
- Hiểu vai trò của Terraform: Declarative (khai báo), State-driven (điều khiển bằng state), và Provider-based (dựa trên nhà cung cấp).
    
- Cài đặt và cấu hình Terraform CLI, xác thực với một nhà cung cấp cloud (ví dụ: AWS).
    
- Thực hành thành thạo vòng đời 3 bước: `terraform init`, `terraform plan`, `terraform apply`.
    
- Hiểu sự nguy hiểm và tính chất không thể đảo ngược của `terraform destroy`.
    

### 2. Lý thuyết ngắn gọn

- **Infrastructure as Code (IaC):** Là việc quản lý và cấp phát hạ tầng (mạng, máy chủ, tường lửa, cơ sở dữ liệu) thông qua các tệp tin cấu hình (code) thay vì thao tác thủ công. Lợi ích: tự động hóa, lặp lại (repeatability), kiểm soát phiên bản (version control) qua Git, và review (Pull Requests).
    
- **Declarative vs. Imperative:** Đây là một thay đổi tư duy cơ bản.
    
    - _Imperative_ (Như Shell/Ansible): "Hãy làm A. Sau đó, hãy làm B. Sau đó, kiểm tra C." Bạn ra lệnh các bước.
        
    - _Declarative_ (Như Terraform/Kubernetes): "Tôi muốn kết quả cuối cùng là D." Bạn định nghĩa trạng thái mong muốn, và Terraform tự tìm ra các bước (A, B, C) để đạt được D.
        
- **Terraform Core & Providers:** Terraform Core là bộ não, chịu trách nhiệm đọc code HCL, quản lý state, và xây dựng đồ thị phụ thuộc. Nó không biết gì về AWS hay Azure. Để nói chuyện với các API (như AWS), nó sử dụng **Providers**. Provider là các plugin (viết bằng Go) thực hiện việc dịch code HCL của bạn thành các lệnh gọi API cụ thể.
    
- **Các khối (Blocks) cơ bản:**
    
    - `terraform {}`: Khối siêu dữ liệu, nơi bạn định nghĩa phiên bản Terraform và các `required_providers`.
        
    - `provider {}`: Cấu hình thông tin xác thực cho một nhà cung cấp (VD: `provider "aws" { region = "us-east-1" }`).
        
    - `resource {}`: "Tôi muốn _tạo_ hoặc _quản lý_ thứ này." Đây là khối quan trọng nhất. (VD: `resource "aws_instance" "web" {}`).
        
    - `data {}`: "Tôi muốn _đọc_ thông tin về một thứ đã tồn tại mà không quản lý nó." (VD: `data "aws_ami" "latest_linux" {}`).
        

### 3. Best practice thực tế

- Luôn chạy `terraform fmt -recursive` trước khi commit. Lệnh này tự động định dạng code của bạn theo chuẩn HCL, đảm bảo tính nhất quán.
    
- Luôn chạy `terraform validate` trong CI (hoặc pre-commit hook) để kiểm tra cú pháp mà không cần gọi API.
    
- Luôn **"ghim" (pin)** phiên bản provider trong khối `terraform {}`. Ví dụ: `version = "~> 5.30.0"`. Dấu `~>` (tilde-wavy) cho phép các bản vá (patch) nhưng ngăn chặn các bản cập nhật lớn (major) (VD: 6.0) có thể phá vỡ (breaking changes) code của bạn.
    
- Đặt tên tài nguyên (resource names) một cách nhất quán (VD: `aws_instance.web_server`). Tên này chỉ dùng trong nội bộ Terraform. Sử dụng `tags` (thẻ) để đặt tên hiển thị trên giao diện Cloud (VD: `Name = "my-web-server"`).
    

### 4. Ví dụ minh họa (AWS): Tạo một EC2 Instance (Phiên bản đầy đủ)

Kịch bản: Tạo một môi trường web cơ bản, bao gồm VPC (mạng riêng), Subnet (mạng con), Security Group (tường lửa), và một EC2 Instance (máy chủ ảo).

Terraform

```json
# main.tf

# 1. Terraform Settings Block
# Defines providers required and pins their versions
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.30.0" # Best Practice: Pin provider version
    }
  }
}

# 2. Provider Configuration
# Configure the AWS provider (e.g., region).
# Assumes credentials are set via environment variables (AWS_ACCESS_KEY_ID, etc.)
provider "aws" {
  region = "us-east-1"
}

# 3. Create a VPC (Virtual Private Cloud)
# This provides network isolation
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "beginner-vpc"
  }
}

# 4. Create a Subnet
# A subnet within the VPC
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id # Reference: depends on the VPC
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "beginner-subnet"
  }
}

# 5. Create a Security Group (Firewall)
# Allows SSH (port 22) and HTTP (port 80)
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow SSH and HTTP inbound traffic"
  vpc_id      = aws_vpc.main.id # Reference: depends on the VPC

  # Ingress rule for SSH (Port 22)
  ingress {
    description = "SSH from My IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # WARNING: Open to the world. Use your_ip/32 in production.
  }

  # Ingress rule for HTTP (Port 80)
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Open to the world
  }

  # Egress rule (Allow all outbound traffic)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1" # -1 means "all protocols"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "beginner-web-sg"
  }
}

# 6. Find the latest Amazon Linux 2 AMI
# Use a 'data' source to read existing information
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["amazon"] # Trust AMIs from 'amazon'
}

# 7. Create the EC2 Instance
# This resource depends on the subnet, security group, and AMI
resource "aws_instance" "web_server" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  # This tag will be visible in the AWS Console
  tags = {
    Name = "beginner-web-server"
  }
}
```

### 5. Mô tả đầu ra mong đợi (Sau khi `apply`)

1. **`terraform init`:** Terraform sẽ tải về provider `hashicorp/aws` phiên bản `~>5.30.0`.
    
2. **`terraform plan`:** Terraform sẽ hiển thị một kế hoạch. Nó sẽ phân tích các phụ thuộc (VD: nó biết phải tạo VPC trước Subnet) và thông báo: `Plan: 4 to add, 0 to change, 0 to destroy.` (VPC, Subnet, SG, Instance).
    
3. **`terraform apply`:** Sau khi bạn gõ "yes", Terraform sẽ thực thi kế hoạch. Nó sẽ tạo các tài nguyên theo đúng thứ tự phụ thuộc.
    
4. **Kết quả:** `Apply complete! Resources: 4 added, 0 changed, 0 destroyed.`
    
5. Lúc này, một tệp tin mới xuất hiện: `terraform.tfstate`.
    

### 6. Cảnh báo bảo mật & vận hành (State, IAM)

- **(Vận hành) File `terraform.tfstate`:** Sau khi `apply`, Terraform tạo ra file `terraform.tfstate` cục bộ. File này là "bộ não" của Terraform, là "nguồn chân lý duy nhất" (Single Source of Truth) của nó. Nó chứa một bản đồ (mapping) giữa code của bạn (VD: `aws_instance.web_server`) và ID tài nguyên thực tế trên cloud (VD: `i-12345abcdef`).
    
- **CẢNH BÁO VẬN HÀNH (Mất State):** Nếu bạn _xóa_ file `terraform.tfstate` này và chạy `apply` một lần nữa, Terraform (bị "mất trí nhớ") sẽ nghĩ rằng nó chưa tạo gì cả. Nó sẽ cố gắng _tạo lại_ tất cả 4 tài nguyên, dẫn đến lỗi xung đột (vì các tài nguyên đó đã tồn tại) hoặc tạo ra các tài nguyên trùng lặp (orphaned resources). **Luôn coi file state quan trọng như code của bạn.**
    
- **CẢNH BÁO BẢO MẬT (Rò rỉ State):** Đây là một trong những lỗ hổng bảo mật cloud phổ biến và tàn khốc nhất.
    
    - File `terraform.tfstate` lưu trữ mọi thứ ở dạng **văn bản thuần (plain text)**.
        
    - Nếu bạn tạo một tài nguyên có chứa bí mật (secret)—ví dụ: một cơ sở dữ liệu—mật khẩu admin ban đầu (`initial_password`) có thể được lưu rõ ràng trong file state.
        
    - Một developer mới có thể vô tình chạy `git add.` và `git commit` file `terraform.tfstate` này. Nếu repository đó là public, các bot quét (scanner bots) sẽ tìm thấy file state trong vòng vài giây, trích xuất credentials (API keys, mật khẩu DB, private keys), và sửu dụng chúng để xâm nhập hệ thống (VD: đào tiền ảo, đánh cắp dữ liệu PII).
        
- **HÀNH ĐỘNG BẮT BUỘC (Action):** _Ngay lập tức_ tạo file `.gitignore` và thêm vào đó:
    
    ```bash
    # Local Terraform state file
    terraform.tfstate
    terraform.tfstate.backup
    
    #.terraform directory (contains provider plugins)
    ```
    

.terraform/

````bash
#.tfvars files (if they contain secrets)
*.tfvars
````

- **(Bảo mật) Access Keys:** Ví dụ trên giả định bạn đã cấu hình AWS credentials (VD: qua `aws configure` hoặc biến môi trường). **Không bao giờ** hard-code (viết thẳng) `access_key` và `secret_key` trong khối `provider`.
    

### 7. Liên hệ thực tế DevOps/Cloud

- Bạn vừa thay thế "click-ops". Thay vì một Kỹ sư Cloud (Cloud Engineer) phải tự tay tạo 4 tài nguyên (mất 5 phút và dễ sai sót), bạn đã có một script tự động 100%, lặp lại được (repeatable), và có thể review (auditable) trong Git.
    
- Đây là nền tảng của "môi trường theo yêu cầu" (on-demand environments). Một developer mới vào team có thể `git clone` repo này, `terraform apply`, và có ngay một môi trường phát triển (dev environment) hoàn chỉnh trong 3 phút.
    

---

## II. TRUNG CẤP (Intermediate): Tái sử dụng và Quản lý State

Ở cấp độ này, chúng ta chuyển tư duy từ "viết script" (scripting) sang "xây dựng hệ thống" (engineering). Trọng tâm là khả năng tái sử dụng (Modules), tham số hóa (Variables), và cộng tác (Remote State & Locking).

### 1. Mục tiêu học tập

- Hiểu tại sao quản lý một file `main.tf` 1000 dòng là một ý tưởng tồi (code "phẳng" - flat file).
    
- Tổ chức code thành các **Modules** (thư mục con) có thể tái sử dụng.
    
- Quản lý state tập trung cho team bằng **Remote Backend** (VD: AWS S3).
    
- Hiểu tại sao **State Locking** (VD: DynamoDB) là _bắt buộc_ khi làm việc nhóm.
    
- Sử dụng **Variables** (input) và **Outputs** (output) để truyền dữ liệu.
    
- Sử dụng vòng lặp: `count` (cũ) và `for_each` (mới, tốt hơn).
    
- Kiểm soát việc tạo/xóa tài nguyên với `lifecycle` và `dynamic` blocks.
    

### 2. Lý thuyết ngắn gọn

- **Variables (Input):** Khai báo các giá trị "đầu vào" cho code của bạn (VD: `instance_type`) bằng khối `variable {}`. Giá trị có thể được cung cấp qua file `terraform.tfvars`, qua tham số CLI (`-var="foo=bar"`), hoặc biến môi trường (`TF_VAR_foo`).
    
- **Outputs:** Khai báo các giá trị "đầu ra" (VD: `public_ip` của máy chủ) bằng khối `output {}`. Đây là cách các module "trả về" giá trị cho module cha.
    
- **Locals:** Các biến "nội bộ" dùng trong 1 file/module (khối `locals {}`). Dùng để giảm lặp code (DRY - Don't Repeat Yourself) và tăng tính dễ đọc.
    
- **Modules:** Một module là một tập hợp các file `.tf` trong một thư mục. (Cấu hình "root" của bạn cũng là một module). Chúng ta gọi các "child modules" (module con) bằng khối `module {}`. Module cho phép bạn đóng gói và tái sử dụng các "khối" hạ tầng (VD: một module "vpc" tiêu chuẩn).
    
- **State Backend (Remote):** Thay vì lưu `terraform.tfstate` ở máy local (rủi ro, không thể cộng tác), chúng ta lưu nó trên một kho lưu trữ chia sẻ. Phổ biến nhất là `backend "s3"` (cho AWS), `backend "gcs"` (cho GCP), hoặc `backend "azurerm"` (cho Azure).
    
- **State Locking:** Khi state được lưu từ xa, điều gì xảy ra nếu 2 developer cùng chạy `terraform apply` một lúc? Họ sẽ "giẫm chân" lên nhau và làm hỏng (corrupt) state. Locking ngăn chặn điều này. `backend "s3"` sử dụng một bảng DynamoDB để "khóa" file state trong khi một người đang `apply`.
    
- **`count` vs `for_each`:** Cả hai đều dùng để tạo nhiều bản sao của một resource.
    
    - `count = 3`: Tạo 3 bản sao, truy cập qua _index_ (VD: `aws_instance.server`).
        
    - `for_each = {"dev", "stg", "prod"}`: Tạo 3 bản sao, truy cập qua _key_ (VD: `aws_instance.server["dev"]`).
        
- **`lifecycle` Block:** Tùy chỉnh hành vi của resource: `create_before_destroy` (hữu ích cho zero-downtime), `prevent_destroy` (bảo vệ tài nguyên quan trọng như DB).
    
- **`dynamic` Block:** Dùng vòng lặp _bên trong_ một khối resource (VD: tạo N `ingress` rules cho 1 security group).
    

### 3. Best practice thực tế

- **Luôn ưu tiên `for_each` hơn `count`:** Đây là một _best practice_ cực kỳ quan trọng.
    
    - _Kịch bản:_ Beginner dùng `count = 3` để tạo 3 server (server-0, server-1, server-2).
        
    - _Vấn đề:_ Developer xóa server ở giữa (server-1) khỏi danh sách. Terraform thấy `count` giảm từ 3 xuống 2. Nó sẽ _xóa_ server `(index cuối cùng) và giữ lại`, ``.
        
    - _Thảm họa ("The Shuffle"):_ Nếu bạn dùng `count` cho một danh sách biến, và xóa một mục ở giữa danh sách, _tất cả_ các tài nguyên _sau_ nó trong danh sách sẽ bị _thay đổi_ (VD: `server-2` bị xóa và tạo lại với index ``). Điều này có thể gây _downtime hàng loạt_ (VD: 3 server, bạn xóa server 1, server 2 và 3 bị destroy/recreate).
        
    - _Giải pháp:_ `for_each` dùng map (key-value). `for_each = {"server-a" = {...}, "server-b" = {...}}`. Nếu bạn xóa "server-b", _chỉ "server-b"_ bị ảnh hưởng. "server-a" và "server-c" hoàn toàn không bị động đến. _Luôn dùng `for_each` khi các tài nguyên là độc nhất (như server, database)._
        
- **Tách State:** Không dùng một state file khổng lồ cho toàn bộ hạ tầng. Tách state theo logic (VD: state cho Network, state cho App-A, state cho App-B). Chúng ta sẽ nói thêm về điều này ở phần Advanced.
    
- **Cấu trúc Module:** Một module tốt cần có `main.tf`, `variables.tf`, `outputs.tf` và một `README.md` (giải thích cách dùng, input/output).
    

### 4. Ví dụ minh họa: Tách hạ tầng thành Module (Network, Compute)

Chúng ta sẽ tái cấu trúc ví dụ Beginner thành các module có thể tái sử dụng.

- **Cấu trúc thư mục mới
- 
```
    /terraform-intermediate
|-- /modules
| |-- /vpc
| | |-- main.tf
| | |-- variables.tf
| | -- outputs.tf | -- /ec2_instance
| |-- main.tf
| |-- variables.tf
| -- outputs.tf 
|-- main.tf # Root module (lắp ráp)
|-- variables.tf # Root variables 
|-- outputs.tf # Root outputs -- terraform.tfvars

```

- **`modules/vpc/variables.tf`** (Định nghĩa input cho module VPC)
    
    Terraform
    
    ```yml
    # modules/vpc/variables.tf
    variable "vpc_cidr" {
      description = "CIDR block for the VPC"
      type        = string
    }
    
    variable "subnet_cidr" {
      description = "CIDR block for the Subnet"
      type        = string
    }
    ```
    
- **`modules/vpc/main.tf`** (Module VPC)
    
    Terraform
    
    ```yml
    # modules/vpc/main.tf
    resource "aws_vpc" "main" {
      cidr_block = var.vpc_cidr
      tags = { Name = "module-vpc" }
    }
    
    resource "aws_subnet" "main" {
      vpc_id     = aws_vpc.main.id
      cidr_block = var.subnet_cidr
      tags = { Name = "module-subnet" }
    }
    ```
    
- **`modules/vpc/outputs.tf`** (Định nghĩa output cho module VPC)
    
    Terraform
    
    ```yml
    # modules/vpc/outputs.tf
    output "vpc_id" {
      description = "The ID of the created VPC"
      value       = aws_vpc.main.id
    }
    
    output "subnet_id" {
      description = "The ID of the created subnet"
      value       = aws_subnet.main.id
    }
    ```
    
- **`modules/ec2_instance/main.tf`** (Module EC2, tóm tắt)
    
    Terraform
    
    ```yml
    # modules/ec2_instance/main.tf
    variable "subnet_id" { type = string }
    variable "vpc_id" { type = string }
    variable "instance_type" { type = string }
    
    # We create the SG inside the EC2 module for this example
    resource "aws_security_group" "web_sg" {
      name        = "web-sg-module"
      vpc_id      = var.vpc_id
      #... ingress/egress rules...
    }
    
    data "aws_ami" "amazon_linux_2" {
      most_recent = true
      #... filters...
      owners = ["amazon"]
    }
    
    resource "aws_instance" "web" {
      ami           = data.aws_ami.amazon_linux_2.id
      instance_type = var.instance_type
      subnet_id     = var.subnet_id
      vpc_security_group_ids = [aws_security_group.web_sg.id]
      tags = { Name = "module-web-server" }
    }
    
    output "public_ip" {
      value = aws_instance.web.public_ip
    }
    ```
    
- **`main.tf` (Root Module)** - Đây là nơi "lắp ráp"
    
    Terraform
    
    ```yml
    # main.tf (Root)
    
    terraform {
      # Define remote backend
      # BỎ COMMENT và thay thế bucket/table CỦA BẠN để sử dụng
      # backend "s3" {
      #   bucket         = "my-terraform-state-bucket-unique-name"
      #   key            = "intermediate/terraform.tfstate"
      #   region         = "us-east-1"
      #   dynamodb_table = "terraform-lock-table"
      #   encrypt        = true
      # }
    
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.30.0"
        }
      }
    }
    
    provider "aws" {
      region = var.region
    }
    
    # 1. Call the VPC module
    module "networking" {
      source = "./modules/vpc" # Local module path
    
      # Pass input variables
      vpc_cidr    = var.vpc_cidr
      subnet_cidr = var.subnet_cidr
    }
    
    # 2. Call the EC2 module
    module "compute" {
      source = "./modules/ec2_instance"
    
      # Pass outputs from 'networking' module as inputs to 'compute' module
      vpc_id        = module.networking.vpc_id
      subnet_id     = module.networking.subnet_id
      instance_type = var.instance_type
    
      # Terraform automatically detects this dependency
      # but explicit dependency can be set with:
      # depends_on = [module.networking]
    }
    ```
    
- **`variables.tf` (Root)**
    
    Terraform
    
    ```yml
    # variables.tf (Root)
    variable "region" { default = "us-east-1" }
    variable "instance_type" { default = "t2.micro" }
    variable "vpc_cidr" { default = "10.10.0.0/16" }
    variable "subnet_cidr" { default = "10.10.1.0/24" }
    ```
    
- **`terraform.tfvars`** (Nơi cung cấp giá trị)
    
    Terraform
    
    ```yml
    # terraform.tfvars
    # Override defaults if needed
    instance_type = "t3.micro"
    ```
    
- **`outputs.tf` (Root)**
    
    Terraform
    
    ```yml
    # outputs.tf (Root)
    output "web_server_ip" {
      description = "Public IP of the web server"
      value       = module.compute.public_ip
    }
    ```
    

### 5. Mô tả đầu ra mong đợi (Sau khi `apply`)

- Khi chạy `terraform init` (sau khi bỏ comment `backend "s3"`), Terraform sẽ hỏi `Would you like to copy existing state to the new backend?`. Gõ "yes".
    
- Terraform sẽ tải lên file state cục bộ lên S3 bucket.
    
- `plan` sẽ hiển thị các tài nguyên được tạo _bên trong_ các module (VD: `module.networking.aws_vpc.main`, `module.compute.aws_instance.web`).
    
- `apply` sẽ tạo hạ tầng, và file `terraform.tfstate` sẽ được _đẩy lên S3 bucket_. File `terraform.tfstate` cục bộ sẽ biến mất.
    
- Đầu ra sẽ hiển thị `web_server_ip = "54.x.x.x"`.
    

### 6. Cảnh báo bảo mật & vận hành (State, Locking)

- **(Bảo mật) Mã hóa Backend:** Khối `backend` nên luôn có `encrypt = true`. Bản thân S3 bucket phải được thiết lập _private_ (không public), bật _versioning_ (để khôi phục state nếu bị hỏng), và bật mã hóa phía máy chủ (SSE-S3).
    
- **(Vận hành) State Locking là BẮT BUỘC:** Ví dụ trên sử dụng `dynamodb_table`.
    
    - _Kịch bản (Không có Locking):_ Alice và Bob trong cùng một team, dùng chung S3 backend.
        
    - Alice chạy `plan` (thêm 1 server). Bob cũng chạy `plan` (đổi 1 security group).
        
    - Alice chạy `apply`. Cùng lúc đó, Bob chạy `apply`.
        
    - Cả hai cùng đọc file state _cũ_ (version 10).
        
    - Alice `apply` xong, ghi file state (version 11).
        
    - Bob `apply` xong, _ghi đè_ file state (version 12).
        
    - _Thảm họa (State Corrupted):_ File state version 12 của Bob _không chứa_ thay đổi của Alice (server mới). Hạ tầng thực tế (có server của Alice) và file state (không có server của Alice) đã _lệch (drift)_. Lần `apply` tiếp theo, Terraform (đọc state 12) sẽ thấy "thiếu 1 server" và _xóa_ server của Alice.
        
    - _Giải pháp (Locking):_ Khi Alice chạy `apply`, Terraform trước tiên sẽ đặt một "khóa" (lock) trong DynamoDB. Khi Bob cố chạy `apply`, Terraform sẽ báo lỗi: `Error acquiring the state lock... Locked by: Alice@hostname`. Bob buộc phải đợi Alice làm xong. Điều này đảm bảo các thao tác `apply` là _tuần tự (serial)_ và an toàn.
        

### 7. Liên hệ thực tế DevOps/Cloud

- Đây là cách Platform Team (đội ngũ nền tảng) hoạt động. Họ không tạo EC2. Họ tạo ra _module_ `ec2_instance` đã được "cứng hóa" (hardened) (VD: đã cài sẵn agent giám sát, tuân thủ CIS benchmark, tag bắt buộc).
    
- Team ứng dụng (Application Team) chỉ cần gọi module đó (`module "my_app" { source = "github.com/company/terraform-modules/ec2"... }`) mà không cần biết chi tiết bên trong.
    
- Việc sử dụng `output` từ module này (`module.networking.vpc_id`) làm `input` cho module khác (`module.compute.vpc_id`) là nền tảng của _Dependency Graph_ (đồ thị phụ thuộc) mà Terraform tự động xây dựng.
    

---

## III. NÂNG CAO (Advanced): Tự động hóa và Môi trường Phức tạp

Phần này tập trung vào việc đưa Terraform vào một luồng CI/CD (Continuous Integration/Continuous Delivery) hoàn chỉnh, quản lý nhiều môi trường (dev/stg/prod) và xử lý "vấn đề khó": secrets.

### 1. Mục tiêu học tập

- Hiểu các chiến lược tách biệt môi trường (dev, staging, production).
    
- So sánh `terraform workspace` (của Terraform) và chiến lược _cấu trúc thư mục_ (tốt hơn).
    
- Xây dựng một pipeline CI/CD (GitHub Actions) đầy đủ: `plan` trên Pull Request, `apply` trên merge `main`.
    
- Đọc state từ xa của một môi trường khác (VD: app đọc state của network) bằng `terraform_remote_state`.
    
- Hiểu tại sao không bao giờ lưu secret trong `.tfvars` và cách tích hợp HashiCorp Vault hoặc AWS Secrets Manager.
    

### 2. Lý thuyết ngắn gọn

- **Tách biệt Môi trường:**
    
    - `terraform workspace`: Một tính năng tích hợp của Terraform. Nó cho phép bạn sử dụng _cùng một code base_ để quản lý _nhiều state file_ khác nhau (VD: `default`, `dev`, `prod`).
        
    - _Chiến lược thư mục (Directory-based):_ Sao chép (hoặc dùng chung module) code nhưng có các thư mục gốc (`/envs/dev`, `/envs/prod`) riêng biệt, mỗi thư mục có `backend.tf` (state file) riêng.
        
- **CI/CD Workflow:** Quy trình tự động hóa Terraform:
    
    - _Khi tạo Pull Request:_ Chạy `init`, `validate`, `fmt`, và `plan`. `plan` được lưu lại (artifact) và post lên PR để review.
        
    - _Khi Merge vào `main`:_ Chạy `init`, tải `plan` artifact, và `apply` `plan` đó.
        
- **`terraform_remote_state` Data Source:** Dùng để _đọc_ `outputs` từ một file state _khác_ (VD: môi trường `app-prod` đọc `vpc_id` từ `output` của state `network-prod`). Đây là chìa khóa để liên kết các state.
    
- **Quản lý Secrets:** Terraform _không phải_ là công cụ quản lý secret. Nó phải _lấy_ (fetch) secret từ một hệ thống chuyên dụng (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager) tại _thời điểm `apply`_.
    

### 3. Best practice thực tế

- **KHÔNG DÙNG `terraform workspace` cho Môi trường (dev/prod):**
    
    - _Cái bẫy:_ Beginner thấy `terraform workspace new dev` và `new prod` và nghĩ rằng đây là cách chuẩn.
        
    - _Vấn đề (The Trap):_ Họ bắt đầu viết code: `instance_type = terraform.workspace == "prod"? "m5.large" : "t2.micro"`. Code base trở nên lộn xộn, đầy rẫy các câu lệnh `if/else` và toán tử tam phân (`?:`). Rất khó để review một PR vì _cùng một code_ lại tạo ra hạ tầng khác nhau tùy thuộc vào một biến (workspace) không nằm trong code.
        
    - _Thảm họa:_ Một developer quên chuyển workspace (VD: đang ở `prod` mà nghĩ là ở `dev`). Họ chạy `apply` một thay đổi (nghĩ là cho `dev`) và _phá hỏng production_.
        
    - **Giải pháp (Enterprise Best Practice):** Sử dụng **cấu trúc thư mục**.
        
```
/envs
|-- /dev
| |-- main.tf # Gọi module "my_app"
| |-- terraform.tfvars
| -- backend.tf # Trỏ đến state: "env/dev/terraform.tfstate" |-- /prod | |-- main.tf # Gọi module "my_app" | |-- terraform.tfvars | -- backend.tf # Trỏ đến state: "env/prod/terraform.tfstate"
/modules
|-- /my_app
| |-- main.tf
| --... 
``` 
Cả dev/main.tfvàprod/main.tfđều gọimodule "my_app". Nhưng chúng là các "root module" *hoàn toàn độc lập*. Chúng có backend(state) riêng,.tfvars(biến) riêng, và (quan trọng nhất) pipeline CI/CD riêng. Điều này cung cấp **sự cô lập (isolation)** tối đa.workspace` chỉ nên dùng cho các thay đổi tạm thời (VD: test 1 feature branch).
- **OIDC cho CI/CD:** Không bao giờ lưu trữ AWS_ACCESS_KEY (static credentials) trong GitHub Secrets. Sử dụng **OIDC (OpenID Connect)** để cho phép GitHub Actions "giả định" (assume) một IAM Role. Role này có _thời hạn ngắn_ (short-lived) và được cấp quyền _tối thiểu_ (least privilege).
- **Tích hợp Vault:** Sử dụng `vault` provider. Terraform sẽ xác thực với Vault, đọc secret, và tiêm (inject) nó vào resource (VD: `aws_db_instance`) mà _không bao giờ_ hiển thị giá trị secret trong state file (nó chỉ lưu 1 placeholder).
    

### 4. Ví dụ minh họa: CI/CD Pipeline (GitHub Actions)

Kịch bản: Tự động hóa môi trường `prod` (theo cấu trúc thư mục `/envs/prod`) bằng GitHub Actions và OIDC.

- **`envs/prod/main.tf`** (Sử dụng backend S3)
    
    Terraform
    
    ```yml
    # envs/prod/main.tf
    terraform {
      backend "s3" {
        bucket         = "my-prod-terraform-state-bucket"
        key            = "prod/terraform.tfstate"
        region         = "us-east-1"
        dynamodb_table = "terraform-prod-lock"
        encrypt        = true
      }
      #... required_providers...
    }
    
    provider "aws" {
      region = "us-east-1"
    }
    
    #... gọi các modules...
    ```
    
- **`.github/workflows/terraform-prod.yml`** (Pipeline CI/CD cho `prod`)
    
    YAML
    
    ```yml
    #.github/workflows/terraform-prod.yml
    name: 'Terraform CD - Production'
    
    on:
      push:
        branches:
          - main # Trigger on merge/push to main
        paths:
          - 'envs/prod/**' # Only run if 'prod' code changes
      pull_request:
        branches:
          - main # Trigger on PRs targeting main
        paths:
          - 'envs/prod/**'
    
    # Grant OIDC permissions
    permissions:
      id-token: write # Required for AWS OIDC
      contents: read
      pull-requests: write # Required to comment 'plan' on PR
    
    jobs:
      terraform:
        name: 'Terraform'
        runs-on: ubuntu-latest
        defaults:
          run:
            shell: bash
            working-directory:./envs/prod # Run all steps in 'prod' directory
    
        steps:
          - name: Checkout
            uses: actions/checkout@v4
    
          - name: Configure AWS Credentials (OIDC)
            uses: aws-actions/configure-aws-credentials@v4
            with:
              # Đây là IAM Role đã được setup để trust GitHub OIDC
              role-to-assume: arn:aws:iam::123456789012:role/GitHubAction-Prod-Terraform-Role
              aws-region: us-east-1
    
          - name: Setup Terraform
            uses: hashicorp/setup-terraform@v3
            with:
              terraform_version: 1.6.0 # Pin version
    
          - name: Terraform Init
            run: terraform init
    
          - name: Terraform Format
            run: terraform fmt -check # Fail if not formatted
    
          - name: Terraform Validate
            run: terraform validate
    
          - name: Terraform Plan (for PR)
            id: plan
            if: github.event_name == 'pull_request'
            run: terraform plan -no-color -out=tfplan.out
            # Tools như 'tfnotify' hoặc 'github-script' có thể post plan lên PR
            # continue-on-error: true # Để các bước sau vẫn chạy
    
          # - name: Add Plan to PR
          #   if: github.event_name == 'pull_request'
          #   uses: actions/github-script@v6
          #   with:
          #     script: |
          #       // Post plan to PR logic here
    
          - name: Terraform Apply (for Merge)
            if: github.event_name == 'push' && github.ref == 'refs/heads/main'
            run: |
              echo "Running Apply for merge to main"
              # Trong kịch bản thực tế, bạn sẽ tải tfplan.out từ PR
              # Ở đây, chúng ta chạy apply trực tiếp để đơn giản hóa
              terraform apply -auto-approve
    ```
    

### 5. Mô tả đầu ra mong đợi (Sau khi `apply`)

- **Khi tạo Pull Request (PR):** Job "Terraform" sẽ chạy. Nó sẽ chạy `terraform plan`. Một bot (hoặc action) sẽ post kết quả `plan` (VD: "2 to add, 1 to change, 0 to destroy") vào comment của PR. Team SRE/DevOps có thể review `plan` này.
    
- **Khi Merge PR vào `main`:** Job "Terraform" chạy lại. Lần này, nó sẽ bỏ qua bước `plan` (vì `if: github.event_name == 'pull_request'` là false) và chạy thẳng bước `Terraform Apply` (vì `if: github.event_name == 'push'` là true).
    
- Hạ tầng `prod` được cập nhật tự động.
    

### 6. Cảnh báo bảo mật & vận hành (IAM, CI/CD)

- **(Bảo mật) Vai trò IAM cho CI/CD:** Đây là điểm yếu lớn nhất trong hệ thống GitOps.
    
    - _Kịch bản (Quá nhiều quyền):_ Nếu Role OIDC (`GitHubAction-Prod-Terraform-Role`) có quyền `AdministratorAccess`, bất kỳ ai có thể merge PR vào `main` đều trở thành Admin của tài khoản AWS.
        
    - _Tấn công:_ Một nhân viên bất mãn (disgruntled employee) hoặc một kẻ tấn công (attacker) chiếm được tài khoản GitHub có thể _thêm code Terraform độc hại_ (VD: `resource "aws_iam_user" "attacker" {}`) vào PR.
        
    - _Hậu quả:_ CI/CD (với quyền Admin) sẽ vui vẻ _thực thi_ nó, tạo ra một IAM user cho kẻ tấn công.
        
    - _Giải pháp (Least Privilege):_ Role OIDC phải được giới hạn _cực kỳ_ chặt chẽ. Chỉ cấp các quyền mà Terraform _thực sự_ cần (VD: `ec2:CreateInstance`, `rds:CreateDBInstance`).
        
    - _Giải pháp (Tách biệt vai trò):_ Tách biệt pipeline `plan` và `apply`. `plan` (an toàn) có thể chạy tự động. `apply` (nguy hiểm) cần _phê duyệt thủ công_ (manual approval) trong GitHub Actions, chỉ có SRE Lead mới được approve.
        
- **(Vận hành) `terraform apply -auto-approve`:** Lệnh này rất nguy hiểm, vì nó bỏ qua bước kiểm tra cuối cùng của con người. Chỉ nên dùng nó khi bạn _hoàn toàn_ tin tưởng vào `plan` đã được review trên PR. Các công cụ như **Atlantis** ra đời để giải quyết chính vấn đề này (bạn `plan` và `apply` bằng cách comment trên GitHub, nó quản lý locking và approval).
    

### 7. Liên hệ thực tế DevOps/Cloud

- Đây là **GitOps** cho hạ tầng. Nguồn chân lý duy nhất (Single Source of Truth) là Git repo (nhánh `main`).
    
- Mọi thay đổi hạ tầng _phải_ đi qua một PR, được review, được `plan`, và được merge.
    
- **Không ai** (kể cả SRE/DevOps) được phép `terraform apply` từ máy tính cá nhân của họ nữa (trừ trường hợp khẩn cấp). Mọi quyền truy cập (IAM) được gỡ bỏ khỏi người dùng và chuyển cho Service Account/Role của CI/CD. Đây là một bước tiến lớn về an ninh và kiểm toán (auditing).
    

---

## IV. CHUYÊN GIA (Expert): Quản trị, Tối ưu hóa và Đa Đám mây

Phần này nói về việc quản lý Terraform ở quy mô rất lớn (hàng trăm developer, hàng ngàn tài nguyên), tập trung vào Quản trị (Governance), Tối ưu hóa (Optimization) và các kịch bản phức tạp (Kubernetes, Multi-Cloud).

### 1. Mục tiêu học tập

- Thực thi "Chính sách dưới dạng Code" (Policy as Code - PaC) để kiểm soát chi phí và bảo mật.
    
- Hiểu về OPA (Open Policy Agent) và Sentinel (HashiCorp).
    
- Quản lý tài nguyên Kubernetes (K8s) bằng Terraform (Providers `kubernetes` và `helm`).
    
- Hiểu các mô hình Multi-Cloud và Hybrid-Cloud (VD: AWS + GCP).
    
- Tối ưu hóa `dependency graph` (đồ thị phụ thuộc) và `parallelism` để tăng tốc `apply`.
    

### 2. Lý thuyết ngắn gọn

- **Policy as Code (PaC):** Thay vì review thủ công (VD: "PR này có mở port 22 không?" hoặc "Server này có phải `t2.micro` không?"), bạn viết _code (chính sách)_ để tự động kiểm tra.
    
    - **Sentinel:** Ngôn ngữ policy của HashiCorp, tích hợp sâu với Terraform Enterprise/Cloud.
        
    - **OPA (Open Policy Agent):** Chuẩn mở (CNCF), dùng ngôn ngữ `Rego`. Có thể kiểm tra file `plan` (dạng JSON) trong CI/CD.
        
- **Terraform & K8s:** Terraform có thể quản lý _bên trong_ K8s.
    
    - `kubernetes` provider: Quản lý tài nguyên K8s (Deployment, Service, ConfigMap).
        
    - `helm` provider: Quản lý (deploy, upgrade) các Helm chart.
        
    - `kubernetes_manifest`: Cách linh hoạt để áp dụng YAML (khi không có resource cụ thể).
        
- **Multi-Cloud:** Sử dụng nhiều `provider` (VD: `provider "aws"` và `provider "google"`) trong _cùng một_ cấu hình Terraform. Dùng để tạo các kết nối (VPN, Interconnect) hoặc cho các kịch bản DR (Disaster Recovery).
    
- **Tối ưu hóa:** Terraform xây dựng một Đồ thị Phụ thuộc (DAG). Nó chạy song song (parallel) các tài nguyên không phụ thuộc nhau (mặc định là 10). `depends_on` (phụ thuộc tường minh) chỉ nên dùng khi Terraform không thể suy ra phụ thuộc ngầm (implicit).
    

### 3. Best practice thực tế

- **Tách biệt "Vòng đời" (Lifecycles) bằng cách Tách State:** Đây là khái niệm quan trọng nhất ở cấp độ expert.
    
    - _Kịch bản:_ Một developer muốn "một cú `apply`" tạo cả EKS Cluster (hạ tầng) VÀ deploy Nginx (ứng dụng) lên đó.
        
    - _Vấn đề:_ Họ dùng `provider "aws"` (tạo EKS) và `provider "helm"` (deploy Nginx) trong _cùng một_ state file. `provider "helm"` cần `kubeconfig` (để xác thực), vốn là _output_ của `aws_eks_cluster`. Điều này tạo ra một vòng đời (lifecycle) _chặt chẽ_.
        
    - _Thảm họa (Blast Radius):_ Bạn đã gộp "Hạ tầng" (thay đổi chậm, ổn định, quản lý bởi Platform team) với "Ứng dụng" (thay đổi nhanh, hàng ngày, bởi App team). Nếu App team chỉ muốn đổi `image:tag` của Nginx, họ _vô tình_ chạy `plan` trên _toàn bộ_ EKS cluster. Một thay đổi nhỏ (như `depends_on`) có thể khiến Terraform muốn _tạo lại_ (recreate) cả cluster. Bán kính ảnh hưởng (blast radius) quá lớn.
        
    - **Giải pháp (Enterprise Best Practice):** **Tách State (Tách vòng đời)**.
        
        1. **State 1 (Platform):** Terraform tạo EKS Cluster. `output` ra `kubeconfig` và `cluster_endpoint`.
            
        2. State 2 (Application): Một Terraform hoàn toàn riêng biệt (repo riêng, CI/CD riêng) của App team. Nó dùng data "terraform_remote_state" để đọc output từ State 1, tự cấu hình provider "helm" và chỉ deploy app.
            
            Kết quả: Platform team có thể thay đổi EKS (VD: nâng cấp K8s) mà không ảnh hưởng App. App team có thể deploy 50 lần/ngày mà không bao giờ "chạm" vào state của hạ tầng. Đây là blast radius reduction (giảm bán kính ảnh hưởng).
            
- **Sử dụng OPA trong CI/CD:** Chạy `opa eval` trên file `plan.json` _trước khi_ cho phép `apply`.
    
    - VD Policy (Rego): `deny["Instance type t2.micro not allowed in prod"]`
        
- **Module "Wrapper":** Tạo các module nội bộ (wrapper) "bọc" các module cộng đồng (VD: "Terraform AWS VPC module"). Bên trong wrapper, bạn thêm các giá trị mặc định (defaults), tag (gắn thẻ), và _policy_ của công ty. Developer chỉ được phép dùng module wrapper này.
    

### 4. Ví dụ minh họa: Terraform + K8s (GKE) + OPA

Kịch bản: Tạo GKE Cluster, lấy credentials, deploy Nginx (Helm), và kiểm tra policy (OPA) trong CI.

- **`main.tf` (Root)**
    
    Terraform
    
    ```yml
    # 0. Setup providers
    terraform {
      required_providers {
        google = { source = "hashicorp/google", version = "~> 5.0" }
        kubernetes = { source = "hashicorp/kubernetes", version = "~> 2.20" }
        helm = { source = "hashicorp/helm", version = "~> 2.10" }
      }
    }
    
    variable "gcp_project" {}
    variable "gcp_region" { default = "us-central1" }
    
    provider "google" {
      project = var.gcp_project
      region  = var.gcp_region
    }
    
    # 1. Create a GKE (Kubernetes) Cluster
    resource "google_container_cluster" "primary" {
      name     = "expert-gke-cluster"
      location = var.gcp_region
    
      # Simplified config
      initial_node_count = 1
    
      # This is required for our OPA policy
      # monitoring_config {
      #   monitoring_service = "monitoring.googleapis.com/kubernetes"
      # }
    }
    
    # 2. Configure Kubernetes & Helm providers dynamically
    # These providers will use the output from the GKE cluster
    
    data "google_client_config" "default" {}
    
    provider "kubernetes" {
      host = "https://${google_container_cluster.primary.endpoint}"
      token = data.google_client_config.default.access_token
      cluster_ca_certificate = base64decode(google_container_cluster.primary.master_auth.cluster_ca_certificate)
    }
    
    provider "helm" {
      kubernetes {
        host = "https://${google_container_cluster.primary.endpoint}"
        token = data.google_client_config.default.access_token
        cluster_ca_certificate = base64decode(google_container_cluster.primary.master_auth.cluster_ca_certificate)
      }
    }
    
    # 3. Deploy an application (Nginx) using the Helm provider
    resource "helm_release" "nginx" {
      name       = "nginx-ingress"
      repository = "https://helm.nginx.com/stable"
      chart      = "nginx-ingress"
      namespace  = "default"
    
      # This depends on the cluster being ready and auth configured
      depends_on = [google_container_cluster.primary]
    }
    ```
    
- **`policy/gke_monitoring.rego` (Policy cho OPA)**
    
    Đoạn mã
    
    ```yml
    # policy/gke_monitoring.rego
    package terraform.analysis
    
    # Deny if any GKE cluster is created without monitoring enabled
    deny[msg] {
      resource := input.resource_changes[_]
      resource.type == "google_container_cluster"
    
      # Check if change is 'create' or 'update'
      resource.change.actions[_] == "create"
    
      # Check if monitoring_service is NOT set correctly
      # (Simplified check for demo)
      not resource.change.after.monitoring_config
    
      msg = "GKE clusters must have GKE monitoring enabled"
    }
    ```
    
- **`ci.sh` (Kịch bản CI/CD tích hợp OPA)**
    
    Bash
    
    ```yml
    #!/bin/bash
    set -e
    
    echo "Running Terraform Plan..."
    terraform init
    terraform plan -out=tfplan.bin
    
    echo "Converting plan to JSON..."
    # 'terraform show' converts the binary plan to JSON
    terraform show -json tfplan.bin > tfplan.json
    
    echo "Running OPA Policy Check..."
    # Download OPA: https://www.openpolicyagent.org/docs/latest/
    
    # 'opa eval' runs the policy against the JSON plan
    # It checks the 'deny' rule in our policy
    OPA_RESULT=$(opa eval -i tfplan.json -d policy/ "data.terraform.analysis.deny")
    
    if" ]; then
      echo "--- POLICY VIOLATION DETECTED ---"
      echo $OPA_RESULT
      echo "Aborting apply."
      exit 1
    else
      echo "Policy check passed."
    fi
    
    echo "Applying Terraform (if on main branch)..."
    # if; then
    #   terraform apply tfplan.bin
    # fi
    ```
    

### 5. Mô tả đầu ra mong đợi (Sau khi `apply`)

- Nếu `main.tf` (tạo GKE) _không_ bao gồm khối `monitoring_config`, pipeline CI/CD sẽ thất bại ở bước `opa eval`.
    
- Đầu ra sẽ là:
    
    ```bash
    --- POLICY VIOLATION DETECTED ---
    ["GKE clusters must have GKE monitoring enabled"]
    Aborting apply.
    ```
    
- Lệnh `apply` sẽ _không bao giờ_ được chạy. Hạ tầng không tuân thủ (non-compliant) bị _ngăn chặn_ trước khi được tạo (shift-left security).
    
- Nếu `main.tf` tuân thủ (thêm `monitoring_config`), OPA pass, `apply` chạy, GKE cluster sẽ được tạo, và Nginx sẽ được deploy bởi Helm.
    

### 6. Cảnh báo bảo mật & vận hành (Governance, State)

- **(Bảo mật) Quản trị K8s:** Khi Terraform quản lý K8s (deployments, secrets), file state của Terraform giờ đây chứa _dữ liệu bên trong K8s_. Bảo vệ file state này càng trở nên quan trọng hơn (VD: nó có thể chứa `ConfigMap` với API key, hoặc thậm chí `kubernetes_secret`).
    
- **(Vận hành) Phụ thuộc Provider động:** Ví dụ trên cấu hình `provider "helm"` bằng _output_ của `resource "google_container_cluster"`.
    
    - Điều này _ràng buộc_ `apply` phải diễn ra theo thứ tự. Tuy nhiên, nếu cluster bị _xóa_ (VD: bằng tay trên giao diện web), lần `plan` tiếp theo sẽ _thất bại_ vì `provider "helm"` không thể xác thực (vì cluster không tồn tại để lấy `endpoint`).
        
    - Điều này tạo ra một tình huống "con gà - quả trứng" (chicken-and-egg) khó gỡ. Đây là lý do tại sao việc **tách state** (Best Practice) là cực kỳ quan trọng. State (Platform) tạo cluster. State (App) đọc state (Platform) và _chỉ_ chạy nếu cluster tồn tại.
        
- **(Vận hành) OPA vs Sentinel:** OPA là chuẩn mở, linh hoạt, nhưng phải tích hợp thủ công vào CI/CD (như script `ci.sh`). Sentinel là độc quyền (proprietary) của HashiCorp, tích hợp mượt mà vào Terraform Cloud/Enterprise, nhưng khóa bạn vào hệ sinh thái của họ.
    

### 7. Liên hệ thực tế DevOps/Cloud

- Đây là "Platform Engineering" thực thụ. Team Platform cung cấp một "đường băng" (paved road).
    
- Terraform (State 1) tạo K8s cluster (Đường băng).
    
- OPA (Governance) là "lan can" (guardrails) của đường băng (cấm dev làm bậy, VD: cấm tạo LoadBalancer public, cấm dùng instance quá to).
    
- Terraform (State 2) + Helm provider là "chiếc xe" mà App team dùng để "lái" (deploy) ứng dụng của họ trên đường băng.
    
- Ở cấp độ này, Terraform không còn là công cụ tạo VM. Nó là **hệ thống điều phối (orchestration system)** cho toàn bộ nền tảng công nghệ của công ty, từ multi-cloud (AWS, GCP) đến edge (K8s).
    

---

## V. PHỤ LỤC (Appendix)

### A. Terraform CLI Cheatsheet

|**Lệnh**|**Mục đích**|**Cảnh báo / Best Practice**|
|---|---|---|
|`terraform init`|Khởi tạo backend, tải providers, tải modules.|Chạy mỗi khi có thay đổi `backend` hoặc `provider` (hoặc clone repo mới).|
|`terraform validate`|Kiểm tra cú pháp (HCL) cục bộ.|Nhanh. Chạy trong pre-commit hook. Không kiểm tra logic.|
|`terraform fmt`|Định dạng code theo chuẩn HCL.|Chạy `fmt -recursive` để đảm bảo tính nhất quán.|
|`terraform plan`|Hiển thị "trạng thái mong muốn" vs "hiện tại".|**Luôn luôn** kiểm tra `plan` kỹ. `plan -out=tfplan.bin` để lưu plan.|
|`terraform apply`|Áp dụng thay đổi.|`apply tfplan.bin` (an toàn hơn) hoặc `apply -auto-approve` (nguy hiểm, chỉ dùng trong CI/CD).|
|`terraform destroy`|Xóa _tất cả_ tài nguyên trong state.|Cực kỳ nguy hiểm. Luôn `plan` trước khi `destroy`.|
|`terraform import`|_Mang_ tài nguyên (đã tạo bằng tay) vào state.|`terraform import aws_instance.web i-12345`. Dùng để "sửa chữa", không tạo code.|
|`terraform state`|Các lệnh "phẫu thuật" state (VD: `list`, `mv`, `rm`).|**CỰC KỲ NGUY HIỂM.** `state rm` xóa khỏi state, _không_ xóa tài nguyên thực. Dùng khi state bị hỏng.|
|`terraform workspace`|Quản lý các state file khác nhau cho cùng 1 code.|Tránh dùng cho `dev/prod`. Dùng cho feature-branch.|
|`terraform output`|Hiển thị giá trị `output` từ state.|`terraform output web_server_ip` (tiện lợi).|
|`terraform taint`|Đánh dấu 1 resource là "hỏng" (tainted).|(Đã cũ) Buộc Terraform `destroy` và `recreate` resource đó ở lần `apply` tiếp theo.|
|`terraform apply -replace=...`|Thay thế `taint`.|`apply -replace=aws_instance.web`. An toàn hơn.|

### B. Cấu trúc Project Mẫu (Enterprise-Grade)

Cấu trúc này được tối ưu hóa cho sự _cô lập (isolation)_ (theo môi trường) và _tách biệt vòng đời (lifecycle)_ (network vs. app).

```
.
├──.github/workflows/      # CI/CD Pipelines
│   ├── app-prod.yml        # Pipeline cho app (trigger on /envs/prod/app)
│   └── network-prod.yml    # Pipeline cho network (trigger on /envs/prod/network)
│
├── envs/                   # Môi trường (Root Modules) - Mỗi thư mục là 1 state file
│   ├── prod/               # Môi trường Production
│   │   ├── app/            # Vòng đời "app" (thay đổi nhanh)
│   │   │   ├── main.tf     # Đọc state 'network' và gọi module 'k8s_app'
│   │   │   ├── backend.tf  # State: s3://my-bucket/prod/app.tfstate
│   │   │   └── terraform.tfvars
│   │   └── network/        # Vòng đời "network" (thay đổi chậm)
│   │       ├── main.tf     # Gọi module 'vpc' và 'eks'
│   │       ├── backend.tf  # State: s3://my-bucket/prod/network.tfstate
│   │       ├── outputs.tf  # Output VpcID, EKS_Endpoint
│   │       └── terraform.tfvars
│   │
│   └── dev/                # Môi trường Development (tương tự prod)
│       ├── app/
│       └── network/
│
├── modules/                # Modules nội bộ (tái sử dụng)
│   ├── aws_vpc_wrapper/    # Bọc module VPC cộng đồng, thêm tag/policy của công ty
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── k8s_app/            # Module deploy K8s (dùng provider helm/kubernetes)
│   │   ├── main.tf
│   │   └── variables.tf
│   └── gcp_gke_cluster/
│       ├── main.tf
│       └──...
│
├── policy/                 # Chính sách OPA/Sentinel
│   └── gke_monitoring.rego
│
└──.gitignore              # Quan trọng: bỏ qua.terraform, *.tfstate, *.tfvars
```

- **Dòng chảy (Flow):**
    
    1. Team Platform `apply` `envs/prod/network/`. State `network.tfstate` được tạo.
        
    2. Team App `apply` `envs/prod/app/`. `main.tf` của họ dùng `data "terraform_remote_state"` để đọc `outputs` (VD: `eks_endpoint`) từ `network.tfstate`.
        
    3. Hai team làm việc độc lập với CI pipeline riêng biệt.
        

### C. Bảng so sánh: Terraform vs Ansible vs Pulumi

|**Tiêu chí**|**Terraform (HashiCorp)**|**Ansible (Red Hat)**|**Pulumi**|
|---|---|---|---|
|**Triết lý**|**Infrastructure as Code (IaC)**|**Configuration Management (CM)**|**Infrastructure as Code (IaC)**|
|**Mô hình**|**Declarative** (Khai báo - "Tôi muốn 5 server")|**Procedural/Imperative** (Thủ tục - "Chạy lệnh A, B, C")|**Declarative** (Nhưng dùng code Imperative)|
|**Ngôn ngữ**|HCL (HashiCorp Config Language)|YAML (với Jinja2)|**Code thực thụ:** Python, Go, TypeScript, C#|
|**Quản lý State**|**Bắt buộc (Stateful)**. Dùng file `tfstate` để biết "trạng thái hiện tại".|**Gần như Stateless** (Trạng thái không bắt buộc). Không biết trạng thái, chạy lại từ đầu.|**Bắt buộc (Stateful)**. Quản lý state (tương tự Terraform) trên backend (Pulumi Service, S3, GCS...).|
|**Thế mạnh**|**Provisioning (Cấp phát)** hạ tầng cloud (VPC, VM, DB, K8s). Quản lý vòng đời.|**Configuring (Cấu hình)** _bên trong_ máy chủ (cài Nginx, config file, user).|Provisioning hạ tầng _và_ logic ứng dụng phức tạp (VD: `if/else`, vòng lặp, test).|
|**Điểm yếu**|Cấu hình bên trong máy chủ (CM) yếu. Logic (vòng lặp) phức tạp trong HCL.|Provisioning (IaC) yếu. Dễ bị "drift" (lệch trạng thái) vì không có state.|Rào cản cao hơn (phải là dev). Dễ viết code "quá phức tạp" cho hạ tầng.|
|**Khi nào dùng?**|Khi bạn cần tạo và quản lý hạ tầng Cloud/K8s. (Nền móng).|Khi bạn cần cấu hình 100 máy chủ đã tồn tại. (Nội thất).|Khi team bạn mạnh về Python/Go và muốn dùng 1 ngôn ngữ cho cả App và Infra.|

- **Mô hình kết hợp (Terraform + Ansible):** Dùng _Terraform_ để tạo máy chủ (EC2) và `output` ra IP. Sau đó, _Ansible_ (chạy trong CI/CD) sẽ đọc IP đó, SSH vào và _cấu hình_ (cài đặt Nginx/Apache).
    

### D. Tài liệu chính thức & Các bài lab thực hành

- **HashiCorp Learn (Quan trọng nhất):** [https://learn.hashicorp.com/terraform](https://learn.hashicorp.com/terraform) (Các bài lab tương tác theo từng bước cho AWS, GCP, Azure).
    
- **Terraform Registry (Nơi tìm Module/Provider):** [https://registry.terraform.io/](https://registry.terraform.io/)
    
- **Tài liệu Terraform CLI:** [https://www.terraform.io/cli/commands](https://www.terraform.io/cli/commands)
    
- **Tài liệu OPA (Open Policy Agent) với Terraform:** [https://www.openpolicyagent.org/docs/latest/terraform/](https://www.openpolicyagent.org/docs/latest/terraform/)
    
- **Atlantis (Công cụ CI/CD cho Terraform):** [https://www.runatlantis.io/](https://www.runatlantis.io/)
    
- **Terraform Best Practices (Bởi Gruntwork):** [https://gruntwork.io/guides/terraform/terraform-best-practices-summary/](https://gruntwork.io/guides/terraform/terraform-best-practices-summary/) (Gruntwork là tác giả của Terragrunt, một công cụ "wrapper" cho Terraform).