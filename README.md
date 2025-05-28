### 배포 파이프라인 다이어그램

![aws-workflow](https://github.com/user-attachments/assets/c6923be3-7644-4c8b-aac0-875c8983085e)

### [ 구성 요소 ]
#### 1. Github
- main 브랜치에 push 시,
- .github/workflows/deployment.yml 트리거되어 자동배포

#### 2. Github Actions
- CI/CD 파이프라인 정의 (.github/workflows/deployment.yml)
- 워크플로우 파일 (deployment.yml) 읽고 단계별로 실행
  - jobs: 아래의 코드들이 차례로 실행
    ```js
    jobs:
      deploy:
        runs-on: ubuntu-latest
        
        steps:
        - name: Checkout repository
          uses: actions/checkout@v4
    
        - name: Install dependencies
          run: npm ci
    
        - name: Build
          run: npm run build
    ```
- Next.js 빌드 & 빌드물 AWS S3에 업로드

#### 3. AWS S3
- 정적 파일 호스팅 (빌드된 Next.js 프로젝트 정적 파일 저장)
- 빌드물 (out 폴더)에 있는 html/css/js 정적 파일들을 S3 버킷으로 복사
  (IAM 액세스키가 github 환경변수로 잘 설정되어 있어야 함.)

#### 4. AWS CloudFront
- S3 버킷을 오리진으로 사용하여,
- CDN 통해 정적 리소스를 사용자에 제공 (콘텐츠, 서비스..)
  <br/>
  <br/>
  <br/>  
#### ✅S3 버킷 웹사이트 엔드포인트 : http://mydogissuperstar.s3-website.us-east-2.amazonaws.com/
#### ✅CloudFront 배포 도메인 이름 : https://d3v0y563zymiaj.cloudfront.net/
