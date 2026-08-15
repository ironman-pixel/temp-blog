---
date: 2025-09-02
tags:
  - til
  - backend
  - alembic
  - python
---
# ❓ Information
* Alembic 사용법 (python data migrations)

---
# ❗ Relevant data
## 🎯 What Is The Objective
Python 에서 사용하는 Database Migration Tool
주로 Alembic + Sqlarchemy로 사용하여 통합 관리하게 됨
## 📦 Information Resources
[tistory: Alembic 사용법](https://magpienote.tistory.com/286)

# 🔰 Content ->  

## 1️⃣ Alembic 설치
```bash
pip install alembic
```

## 2️⃣ Alembic의 환경 생성 or 초기화
alembic init 으로 migaration의 환경 생성
migrations 폴더와 alembi.ini 파일도 생성됨

```bash
# alembic init {migration의 환경명}
alembic init migrations
```

### 폴더 설명
- versions - 마이그레이션 할 **스크립트 코드**가 들어감
- env.py - DB 마이그레이션 시 실행 되는 **서버 연결 및 마이그레이션 실행 코드**
- script.py.mako - 마이그레이션 **템플릿 파일**
- alembic.ini - env.py 파일에서 **Configuration으로 사용되는 alembic 설정** 파일

## 3️⃣ Alembic database 설정
alembic.ini 파일에 sqlalchemy.url 에 DB 주소를 설정

```ini
sqlalchemy.url = postgresql+psycopg2://postgres:postgres@localhost:5432/lms
```

_ini파일은 동적으로 sql url을 할당할 수 없기에 class를 만들어 사용하든 python 코드로 사용하든 하면 더 좋다_

```python
config = context.config

if not config.get_main_option("sqlalchemy.url"):
	config.set_main_option(
		"sqlalchemy.url",
		"postgresql+psycopg2://{username}:{password}@{host}:{port}/{db_name}".format(
			username="postgres",
			password="postgres",
			host="localhost",
			port="5432",
			db_name="lms"
		)
	)
```

## 4️⃣ alembic 마이그레이션 스크립트 생성

아래 명령어를 사용하면 revision 스크립트가 생기며, revision 버전도 생성된다.
```bash
# alembic revision -m "{commit 메시지}"
alembic revision -m "init"
```

```python
"""init

Revision ID: 2a1080a73181
Revises:
Create Date: 2024-05-05 11:03:38.853884

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa

# revision identifiers, used by Alembio.
revision: str =  '2a1080a73181'
down_revision: Union[str, Node] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sqeunce[str], None] = None

def upgrade() -> None:
	pass
	
def downgrade() -> None:
	pass
```

upgrade, downgrade 부분을 수정해주면 되는데 
revision 해시값을 이용하며 이 버전으로 upgrade 시에는 upgrade 함수가 downgrade 시에는 downgrade 함수가 호출되어 실행됨.

user, book 테이블을 생성하는 예시:
```python
def upgrade() -> None:
	op.create_table(
		"user",
		sa.Column('id', sa.Integer),
		sa.Column('name', sa.String),
		sa.PrimaryKeyConstraint('id)
	)
	
	op.create_table(
		'book',
		sa.Column('id', sa.Integer),
		sa.Column('title', sa.String),
		sa.Column('author', sa.Stirng),
		sa.Column('borrowed', sa.String),
		sa.PrimaryKeyConstraint('id')
	)

def downgrade() -> None:
	op.drop_table('user')
	op.drop_table('book')
```


## 5️⃣ Migration 실행

### upgrade
upgrade head를 진행하면 db안에 있는 현재 alembic_version의 해시값보다 높은  revision 해시값에 해당하는 스크립트를 실행한다

```log
alembic upgrade head
INFO [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO [alembic.runtime.migration] Will assume transactional DDL.
INFO [alembic.runtime.migration] Running upgrade -> 2a1080a73181, init
```

### downgrade
위에 작성 한 것과 같이 downgrade를 하면 삭제됨
```log
alembic downgrade -1
INFO [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO [alembic.runtime.migration] Will assume transactional DDL.
INFO [alembic.runtime.migration] Running downgrade 2a1080a73181 -> , init
```

## 6️⃣ revision 코드 자동 생성

alembic은 sqlalchemy와 연동 되어 있다 보니 sqlalchemy에 있는 model을 기반으로 자동으로 revision migration 코드를 생성 해 줄 수 있다.
먼저 아래와 같이 declarative_base를 이용하여 base를 선언한다.

```python
from sqlalchemy.ext.decalrative import declarative_base

Base = declarative_base()
```

Base에 등록된 모델들의 생성이나 변경점을 찾아 자동으로 generate함

```python
from sqlalchemy import Column, Integer, String, Boolean

class test(Base):
	__tablename__ = 'test'
	id = Column(Integer, primary_key=True)
	title = Column(String)
	test = Column(String)
```

Target_metadata를 바꿨기 때문에 위에 Base를 기준으로 재설정된다.
그래서 User와 Book은 삭제되고, Test 하나만 남게 되는 revision script 가 완성된다

```bash
alembic revision --autogenerate -m "create table test"
```

```python
"""create table test

Revision ID: 536d2ba9a373
Revises: 2a1080a73181
Create Date: 2024-05-05 11:39:10.684664

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa

revision: str = '536d2ba9a373'
down_revision: Union[str, None] = '2a1080a73181'
branch_labels: Union[str, Sequence[str], None] = None
depneds_on: Union[str, Sequence[str], None] = None

def upgrade() -> None:
	op.create_table('test',
		sa.Column('id', sa.Integer(), nullable=False),
		sa.Column('title', sa.String(), nullable=True),
		sa.Column('test', sa.String(), nullable=True),
		sa.Column('test2', sa.String(), nullable=True),
		sa.PrimaryKeyConstraint('id')
	)
	op.drop_table('book')
	op.drop_table('user')

def downgrade() -> None:
	op.create_table('user',
		sa.Column('id', sa.INTEGER(), autoincrement=True, nullable=False),
		sa.Column('name', sa.VARCHAR(), autoincrement=False, nullable=True),
		sa.PrimaryKeyConstraint('id', name='user_key')
	)
	op.create_table('book'
		sa.Column('id', sa.INTEGER(), autoincrement=True, nullable=False),
		sa.Column('title', sa.VARCHAR(), autoincrement=False, nullable=True),
		sa.Column('author', sa.VARCHAR(), autoincrement=False, nullable=True),
		sa.Column('borrowed', sa.VARCHAR(), autoincrement=False, nullable=True),
		sa.PrimaryKeyConstraint('id', name='book_pkey')
	)
	op.drop_table('test')
```

여기서 add column 을 한번 해보고 generate 를 다시 해보면 add Column Sctipt 가 생성된걸 알 수 있다.

```python
class test(Base):
	__tablename__ = 'test'
	id = Column(Integer, primary_key=True)
	title = Column(String)
	test = Column(String)
	test2 = Column(String) # 추가
```

```python
"""add column test2

Revision ID: e3e5f02ba243
Revise: 536d2ba9a373
Create Date: 2024-05-05 11:42:25.873999

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa

revision: str = 'e3e5f02ba243'
down_revision: Union[str, None] = '536d2ba9a373'
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None

def upgrade() -> None:
	op.add_column('test', sa.Column('test2', sa.String(), nullable=True))

def downgrade() -> None:
	op.drop_column('test', 'test2')
```

## Alembic history

alembic으로 마이그레이션은 마쳤지만 현재 상태를 확인해 볼 필요성이 있다.
아래와 같이 명령어를 실행하면 현재 어떤 것을 진행했는지 나오게 된다.

```bash
alembic histroy --verbose
```

```
Rev: e3e5f02ba243 (head)
Parent: 536d2ba9a373
Path: C:\\dev\\projectName\\migrations\\versions\\2024_05_05_1142-e3e5f02ba243_add_column_test2.py

	add column test2
	
	Revision ID: e3e5f02ba243
	Revise: 536d2ba9a373
	Create Date: 2024-05-05 11:42:25.873999

Rev: 536d2ba9a373
Parent: 2a1080a73181
Path:  C:\\dev\\projectName\\migrations\\versions\\2024_05_05_1139-536d2ba9a373_create_table_test.py
	
	create table test
	
	Revision ID: 536d2ba9a373
	Revises: 2a1080a73181
	Create Date: 2024-05-05 11:39:10.684664

Rev: 2a1080a73181
Parent: <base>
Path:  C:\\dev\\projectName\\migrations\\versions\\2024_05_05_1103-2a1080a73181_init.py
	
	init
	
	Revision ID: 2a1080a73181
	Revises:
	Create Date: 2024-05-05 11:03:38.853884
```

