ap-northeast-1
========

parsoid 서버
--------
Debian stretch

```sh
#
# 도커 설치
# Reference: https://docs.docker.com/install/linux/docker-ce/debian/
#
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/debian \
  $(lsb_release -cs) \
  stable"
sudo apt-get update && sudo apt-get install -y docker-ce

#
# 스왑 메모리 생성
#
sudo fallocate -l 3G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo swapon

#
# 서비스 시작
#
git clone https://github.com/femiwiki/swarm.git ~/swarm
cp ~/swarm/parsoid.sample.yml ~/swarm/parsoid.yml
# ~/swarm/parsoid.yml를 수정하세요
sudo docker swarm init
sudo mkdir /srv/mysql
sudo docker stack deploy -c ~/swarm/parsoid.yml parsoid
```

페미위키 실서버/테스트서버
--------
```sh
# 이미 있는 리눅스 인스턴스에 ENA를 활성화하는 과정이 필요함

# 커널 업그레이드
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install linux-aws
sudo apt-get autoremove

# 인스턴스 '중지' 상태로 만듦
sudo poweroff
```

이후 로컬에서 아래 커맨드 실행
```sh
INSTANCE_ID='i-00ed7086ef9b999e6'

# 체크
aws ec2 describe-instances --instance-ids "${INSTANCE_ID}" --query 'Reservations[].Instances[].EnaSupport'
# 활성화
aws ec2 modify-instance-attribute --instance-id "${INSTANCE_ID}" --ena-support

# AMI 체크
AMI_ID='ami-029d4bdda4bd270dc'
aws ec2 describe-images --image-id "${AMI_ID}" --query 'Images[].EnaSupport'
```

이후 인스턴스 재시작

```sh
#
# Caddy 세팅
#
sudo apt-get install php7.0-fpm

# /usr/local/bin/caddy 에 바이너리 준비
# /etc/init/caddy.conf 에 서비스파일 준비
sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy
cat <<'EOF' | sudo tee /etc/caddy/Caddyfile
*:80 {
  root /var/www/femiwiki.com
  index index.php
  fastcgi / unix:/var/run/php/php7.2-fpm.sock php
  rewrite /w/api.php {
    to /api.php
  }
  rewrite /w {
    r  /(.*)
    to /index.php
  }
}
EOF

sudo service caddy start
```
