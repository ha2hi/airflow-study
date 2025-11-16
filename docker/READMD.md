### Docker 설치
Ubuntu OS 기준  
- 기존 Docker 설치 시 제거
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```  

- 패키지 업데이트
```
sudo apt-get update
```  

- 종속 패키지 설치
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```  

- GPG 키 추가
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```  

- 패키지 업데이드
```
sudo apt-get update
```  

- Docker 설치
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```  

- 설치 확인
```
docker --version
```  

### Docker-Compose V2.5.0 설치
Docker Compose를 사용하여 Airflow를 실행하는 경우 v2.14.0 이상을 설치해야 함.  
- 도커 컴포즈 설치
```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```  

- 권한 부여
```
sudo chmod +x /usr/local/bin/docker-compose
```  

- 심볼릭 링크
```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```  

- 설치 확인
```
docker-compose --version
```