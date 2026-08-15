---
date: 2025-06-16
tags:
  - til
  - backend
  - neo4j
---
# 🔰 Content ->  

## Docker로 Neo4j실행

> https://basketdeveloper.tistory.com/entry/Neo4j%EB%A5%BC-docker-compose%EB%A1%9C-%EB%9D%84%EC%9B%8C%EB%B3%B4%EA%B8%B0

docker-compose.yml 파일로 실행해봤다.

```
services:
  neo4j:
    container_name: neo4j-boot
    image: neo4j:5.22.0
    ports:
      - 7474:7474 # for browser console
      - 7687:7687 # for db
    volumes:
      - ./:/data ## volume mount
    environment:
      NEO4J_AUTH: neo4j/12345678 ## admin/password
```

```
$ docker-compose -f ./docker-compose.yml up -d
```

log에 잘 실행이 되었다 뜨면 *localhost:7474* 접속


## MCP로 Cursor에 Knowledge Graph 바로 연결

> https://forum.cursor.com/t/mcp-add-persistent-memory-in-cursor/57497

어느 똑똑한 양반이 이미 만들었어