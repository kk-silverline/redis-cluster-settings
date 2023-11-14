# redis cluster 테스트 하는 방법
1. redis 실행
   ```shell
   sh redis_start.sh
   ```

2. 프로세스 확인
   ```shell
   ps -ef | grep redis
   ```

3. 마스터 설정
   ```shell
   redis-cli --cluster create 127.0.0.1:6300 127.0.0.1:6301 127.0.0.1:6302
   ```

4. 슬레이브 등록 (6309<->6409 제외)
   ```shell
   redis-cli --cluster add-node 127.0.0.1:6400 127.0.0.1:6300 --cluster-slave
   redis-cli --cluster add-node 127.0.0.1:6401 127.0.0.1:6301 --cluster-slave
   redis-cli --cluster add-node 127.0.0.1:6402 127.0.0.1:6302 --cluster-slave
   ```

5. 클러스터 확인
   ```shell
   redis-cli --cluster info 127.0.0.1:6300
   redis-cli --cluster check 127.0.0.1:6300
   redis-cli -p 6300
   ```

6. 클러스터 접속
   ```shell
   redis-cli -c -p 6300 -h localhost
   ```

# slot 캐시 제거
e.g. 단일 master일 때 발생하는 Not all 16384 slots are covered by nodes. 에러 처리하기 위함
   ```shell
   redis-cli --cluster call 127.0.0.1:6309 flushall
   redis-cli --cluster call 127.0.0.1:6309 reset
   redis-cli -p 6309 cluster addslots {0..16383}
   ```

# 다운된 노드 클러스터에서 제거
- 다른 노드에서도 정상적으로 처리했을 경우
   ```shell
   redis-cli -p 6300 cluster forget {node-id}
   ```
- 다른 노드는 다 꺼져있는 상태일 경우 (정상 클러스터 접속)
   ```shell
   cluster reset {soft | hard}
   ```
# 다운됐지만 이미 연결된 node-id 알아내기
1. 정상적으로 떠 있는 클러스터 접속
2. cluster nodes

# redis process all kill
```shell
ps -ef |grep practices/redis/subscriber.js |grep node | awk '{print $2}' | xargs kill -9
ps -ef |grep practices/redis/publisher.js |grep node | awk '{print $2}' | xargs kill -9
```

# 참고
redis.conf, redis_cluster.conf 파일은 참고용
