페미위키 미디어위키 서버 [![Container Registry]][Container Registry Link] [![Github checks Status]][Github checks Link]
========
한국의 페미니즘 위키인 [femiwiki.com]에 사용되는 미디어위키 서버입니다.
Dockerfile, 도커 컴포즈 파일 등 다양한 코드를 담고있습니다.
데이터베이스와 memcached, 각종 봇들이 실행됩니다.

[Docker Swarm]을 이용해, 아래와 같이 간편하게 페미위키를 로컬에서 실행할 수
있습니다.

```bash
cp configs/secret.php.example configs/secret.php
docker stack deploy --prune -c development.yml mediawiki
```

페미위키 개발하실때엔 아래 커맨드들을 참고해주세요.

```bash
# 도커이미지 빌드
docker build -t femiwiki/mediawiki .
# 수정된 도커이미지를 실행할때엔 아래와 같이
docker service update --force femiwiki_fastcgi

# configs/LocalSettings.php 검사
composer install
composer test
# configs/LocalSettings.php 자동 교정
composer fix
```

&nbsp;

### Production
페미위키는 프로덕션 배포에도 [Docker Swarm]을 사용합니다. 페미위키에서 사용하는
AWS EC2 AMI는 [femiwiki/ami]를 참고해주세요.

프로덕션 배포를 할때엔 [secret.php] 에서 개발자모드를 반드시 꺼주세요.

```sh
sudo mkdir -p /srv/tweetbot
sudo docker swarm init
sudo docker stack deploy --prune -c ~/mediawiki/production.yml mediawiki
sudo docker stack deploy --prune -c ~/mediawiki/bots.yml bots
```

### About Docker image

페미위키를 위한 [PHP-FPM] 서버입니다.
동일한 이미지를 `caddy run` 커맨드로 사용할 경우에는 웹 서버를 실행할 수 있습니다.
다음 예시 Compose file를 참고해 주세요.

```yml
http:
  image: ghcr.io/femiwiki/mediawiki
  command: caddy run
  ports:
    - 80:80
  volumes:
    - ./caddy/Caddyfile.dev:/srv/femiwiki.com/Caddyfile:ro
      window: 120s
fastcgi:
  image: ghcr.io/femiwiki/mediawiki
  volumes:
    - ./configs:/a:ro
```

&nbsp;

--------

The source code of *femiwiki/mediawiki* is primarily distributed under the terms
of the [GNU Affero General Public License v3.0] or any later version. See
[COPYRIGHT] for details.

[Container Registry]: https://badgen.net/badge/icon/docker?icon=docker&label
[Container Registry Link]: https://github.com/orgs/femiwiki/packages/container/mediawiki
[Github checks Status]: https://badgen.net/github/checks/femiwiki/docker-mediawiki
[Github checks Link]: https://github.com/femiwiki/docker-mediawiki
[femiwiki.com]: https://femiwiki.com
[Docker Swarm]: https://docs.docker.com/engine/swarm/
[femiwiki/ami]: https://github.com/femiwiki/ami
[secret.php]: configs/secret.php.example
[php-fpm]: https://php-fpm.org/
[GNU Affero General Public License v3.0]: LICENSE
[COPYRIGHT]: COPYRIGHT
