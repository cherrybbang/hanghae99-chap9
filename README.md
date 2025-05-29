### 배포 파이프라인 다이어그램

<img src="https://github.com/user-attachments/assets/c6923be3-7644-4c8b-aac0-875c8983085e" width="700px" height="700px">

### [배포 파이프라인 순서]
1. GitHub 코드 Push
2. GitHub Actions 실행
3. 프로젝트 빌드
4. S3에 정적 파일 업로드 (빌드물)
5. CloudFront 캐시 무효화 -> 최신 버전으로 업데이트
6. 최종적으로 CloudFront(= cdn) 통해 사용자에 서비스 제공
   
### [구성 요소]
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
- 빌드물 (out 폴더)에 있는 html/css/js 정적 파일들을 S3 버킷으로 복사 (IAM 액세스키가 github 환경변수로 잘 설정되어 있어야 함.)

#### 4. AWS CloudFront
- S3 버킷을 오리진으로 사용하여,
- CDN 통해 정적 리소스를 사용자에 제공 (콘텐츠, 서비스..)
- 📍 캐시무효화 -> CloudFront가 예전에 캐싱해 둔 오래된 파일들을 강제로 삭제해서, 새 파일로 갱신하는 과정

##### <캐시 무효화가 필요한 이유>
1. CloudFront는 속도를 위해 한 번 로딩한 S3 정적 파일들을 캐시에 저장해서 뿌림
2. S3에 새 버전을 올려도 CloudFront는 예전 버전을 계속 사용자에게 보여줄 수 있음
3. 따라서, 강제로 캐시를 무효화해서 새 버전으로 갱신해줘야 함<br/>
##### <캐시 무효화하는 방법>
- 해결 방법: GitHub Actions에서 CloudFront 캐시 무효화 명령어를 추가
- --distribution-id: CloudFront 배포 ID. CLOUDFRONT_DISTRIBUTION_ID 로 GitHub Secrets에 저장
- --paths "/*": 모든 파일을 새로고침하도록 무효화함. IAM 사용자에 cloudfront:CreateInvalidation 권한이 있어야 함
- CloudFront 캐시 무효화는 S3 업로드 후, 사용자에게 새 파일을 보이게 하기 위한 필수 단계
- 안 하면 사용자에게 계속 이전 버전이 보여서 “배포했는데 왜 안 바뀌지?” 현상이 생겨
- 그런데, 무료가 아니고 유료라는게 함정..🥲
```js
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      ...

      - name: Invalidate CloudFront cache
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"

```  
### [CDN 도입 후 성능 개선]
#### 1. 페이지를 불러오는 속도가 빠르다.<br/>
#### <cdn 도입 전>
<img src="https://github.com/user-attachments/assets/1be90842-aa3b-44ba-b087-ec6cc7fa4042" width="400" height="400">

#### <cdn 도입 후>
<img src="https://github.com/user-attachments/assets/c77d6ed2-decf-4f6c-959b-2d79b4e2a041" width="400" height="400">


  
#### ✅S3 버킷 웹사이트 엔드포인트 : http://mydogissuperstar.s3-website.us-east-2.amazonaws.com/
#### ✅CloudFront 배포 도메인 이름 : https://d3v0y563zymiaj.cloudfront.net/
