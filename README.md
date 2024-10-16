# 🚀 Terraform을 이용한 AWS S3 버킷 설정 가이드

## 📁 S3 버킷 생성 및 설정

### 🪣 S3 버킷 생성
```hcl
resource "aws_s3_bucket" "bucket1" {
  bucket = "xx-bucket2"
}
```
- 이 코드는 "xx-bucket2"라는 이름의 S3 버킷을 생성합니다.

### 🔓 Public Access 설정
```hcl
resource "aws_s3_bucket_public_access_block" "bucket1_public_access_block" {
  bucket = aws_s3_bucket.bucket1.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}
```
- 이 설정은 버킷의 public access를 허용합니다. 

### 🌐 웹사이트 호스팅 설정
```hcl
resource "aws_s3_bucket_website_configuration" "xweb_bucket_website" {
  bucket = aws_s3_bucket.bucket1.id

  index_document {
    suffix = "index.html"
  }
}
```
- S3 버킷을 정적 웹사이트로 호스팅하도록 설정합니다.

### 📜 버킷 정책 설정
```hcl
resource "aws_s3_bucket_policy" "public_read_access" {
  bucket = aws_s3_bucket.bucket1.id

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [ "s3:GetObject" ],
      "Resource": [
        "arn:aws:s3:::xx-bucket2",
        "arn:aws:s3:::xx-bucket2/*"
      ]
    }
  ]
}
EOF
}
```
- 이 정책은 버킷의 모든 객체에 대한 public read 접근을 허용합니다.

### 📄 index.html 파일 업로드
```hcl
resource "aws_s3_object" "index" {
  bucket       = aws_s3_bucket.bucket1.id
  key          = "index.html"
  source       = "index.html"  
  content_type = "text/html"
}
```
- 로컬의 index.html 파일을 S3 버킷에 업로드합니다.

## 👤 IAM 설정

### 🔑 IAM 역할 생성
```hcl
resource "aws_iam_role" "s3_create_bucket_role" {
  name = "s3-create-bucket-role"
   
  assume_role_policy = jsonencode({
    "Version": "2012-10-17",
    "Statement": [
      {
        "Action": "sts:AssumeRole",
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        }
      }
    ]
  })
}
```
- EC2 인스턴스가 사용할 수 있는 IAM 역할을 생성합니다.

### 📋 IAM 정책 정의
```hcl
resource "aws_iam_policy" "s3_full_access_policy" {
  name        = "s3-full-access-policy"
  description = "Full access to S3 resources"
   
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:*"  # 모든 S3 액세스 허용
        ]
        Resource = [
          "*"  # 모든 S3 리소스에 대한 권한
        ]
      }
    ]
  })
}
```
- S3에 대한 모든 권한을 가진 IAM 정책을 정의합니다.

### 🔗 IAM 역할에 정책 연결
```hcl
resource "aws_iam_role_policy_attachment" "attach_s3_policy" {
  role       = aws_iam_role.s3_create_bucket_role.name
  policy_arn = aws_iam_policy.s3_full_access_policy.arn
}
```
- 생성한 IAM 역할에 S3 full access 정책을 연결합니다.

## 🔍 주의사항
1. S3 버킷을 public으로 설정하면 보안 위험이 있을 수 있습니다. 필요한 경우에만 사용하세요.
2. IAM 정책에서 S3에 대한 모든 권한을 부여하고 있습니다. 실제 환경에서는 필요한 권한만 부여하는 것이 좋습니다.
3. 버킷 이름, 정책 등은 실제 사용 시 적절히 수정해야 합니다.

이 설정들을 통해 S3 버킷을 생성하고, 정적 웹사이트로 호스팅하며, 필요한 IAM 역할과 정책을 설정할 수 있습니다. Terraform을 사용하면 이러한 리소스들을 코드로 관리하고 쉽게 재현할 수 있습니다.
