### 배포 파이프라인 다이어그램

![aws-workflow](https://github.com/user-attachments/assets/c6923be3-7644-4c8b-aac0-875c8983085e)

### [ 구성 요소 ]
#### 1. Github
- main 브랜치에 push 시 자동 배포.

#### 2. Github Actions
- CI/CD 파이프라인 정의 (.github/workflows/deployment.yml)
- Next.js 빌드 & AWS S3에 업로드

#### 3. AWS S3
- 정적 파일 호스팅 (빌드된 Next.js 프로젝트 정적 파일 저장)

#### 4. AWS CloudFront
- S3를 오리진으로 사용하여 CDN 제공
- 사용자에 서비스 제공
  <br/>
  <br/>
  <br/>  
#### ✅S3 버킷 웹사이트 엔드포인트 : http://mydogissuperstar.s3-website.us-east-2.amazonaws.com/
#### ✅CloudFront 배포 도메인 이름 : https://d3v0y563zymiaj.cloudfront.net/
