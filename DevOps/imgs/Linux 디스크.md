
```bash
# 디스크 용량 확인
df -h

# 현재 디렉터리의 사용량 파일단위 출력
du -h

# docker 저장소 확인
docker info | grep "Docker Root Dir"

# 일반 권한으로는 해당 위치에 접속 불가
# root 접속
su root
septecace1303

cd /var/lib/docker/overlay2

du -sh */diff/tmp | sort -nr

# 용량 많이 차지하는 디렉터리 삭제
# 디렉터리 삭제후 기존 docker-compose가 실행이 안될 수 있음
# 에러에 뜨는 디렉터리를 그대로 생성하면 재기동 됨


sudo find /dev -type d -exec du -h --max-depth=4 {} + | sort -h

docker system prune -a
```

https://confluence.curvc.com/pages/releaseview.action?pageId=109642597

