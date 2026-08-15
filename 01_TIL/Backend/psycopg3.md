---
date: 2025-09-02
tags:
  - til
  - backend
  - fastapi
  - postgresql
  - python
---
# ❓ Information
* Python 에서  Postgresql을 다루기 위한 psycopg 라이브러리

---
# ❗ Relevant data
## 🎯 What Is The Objective
psycopg 라이브러리란
## 📦 Information Resources
[blog: psycopg3-postgres-example](https://taejoone.jeju.onl/posts/2023-05-29-psycopg3-postgres-example/)


# 🔰 Content ->  

## 1️⃣ psycopg3 설치
```bash
pip install "psycopg[binary]"
pip install "psycopg[binary,pool]"
```

## 2️⃣ psycopg 동기식 사용 (sync)

### 1. DB 접속
_autocommit 옵션을 사용하면 모든 변경 사항이 즉시 반영된다._

```python
import psycopg
from psycopg import connection, sql
from psycopg.rows import class_row
from pydantic import BaseModel

def connect_db(DATABASE_URL: str) -> connection.Connection | None:
	"""Connect to the PostgreSQL database server
	
	참고:
		- current_date => datetime.date (time 데이터 없음)
		- current_timestamp, now() => datetime.datetime
	"""
	try: 
		conn = psycopg.connect(DATABASE_URL, autocommit=True)
		
		# Test Connection
		with conn.cursor() as cur:
			cur.execute("select current_timestamp, 'ok' as result")
			data = cur.fetchone()
			print("data[0]:", data[0], type(data[0]))
			print("data[1]:", data[1], type(data[1]))
			assert data[1] == "ok"
		return conn
	except psycopg.Error as e:
		print("Unable to connect!", e)

def main(DATABASE_URL: str):
	conn = connect_db(DATABASE_URL)
	if conn is None:
		return

if __name__ == "__main__":
	# load_dotenv()
	DATABASE_URL = os.getenv("DATABASE_URL")
	main(DATABASE_URL)
```

### 2. pydantic 자료구조

dataclass 또는 pydantic을 사용하면 select 할때 편리하다

- insert 할 때는 클래스 사용이 별 도움이 못된다
- pydantic 의 validator 데코레이터를 이용하면 제약사항을 기술 할 수 있다.

```python
from pydantic import BaseModel
from pytz import timezone

class TestRow(BaseMode):
	id: int | None
	logdate: datetime
	info: dict
	phones: List[str] = []
	content: str | None = ""
	
	@validator("content")
	def content_default(cls, v):
		print(f"validator(content): {v}")  # => None
		return v or "no data"  # if null, set default value

def main(DATABASE_URL: str):
	conn = connect_db(DATABASE_URL)
	
	# 직접 정의하거나 dict 으로부터 parse_obj 로 생성
	data = TestRow.parse_obj(
		{
			"logdate": datetime.now(timezone("Asia/Seoul")),
			"info": {
				"customer": "Alex Cross",
				"items": {"product": "Tea", "qty": 6},
			},
			"phones": ["010-1234-5678", "064-1234-5678"],
			"content": "얼어붙은 플레이어의 귀환 (미완) - `제리엠`",			
		}
	)
	print("data:", data)
```

### 3. insert 데이터

SQL 인젝션을 방어하기 위해 `sql.Identifier`, `sql.Literal` 등을 적극 사용하자.

- json 데이터는 한번 dumps 시킨 후에 사용해야 한다

```python
from psycopg import connection, sql

def insert_data(conn: connection.Connection, data: TestRow):
	with conn.cursor() as cur:
		stmt = sql.SQL(
			"INSERT INTO {} (logdate, info, phones, content) VALUES ({}, {}, {}, {})"
		).format(
			sql.Identifier("test"),  # table name
			sql.Litetal(data.logdate),
			sql.Literal(json.dumps(data.info)),  # json -> str
			sql.Literal(data.phones),
			data.content,
		)
		# print("SQL:", stmt.as_string)
		cur.execute(stmt)

def main(DATABASE_URL: str):
	conn = connect_db(DATABASE_URL)
	with conn:
		try:
			insert_data(conn, data)
		except psycopg.Error as e:
			print("Unable to insert data!", e)
```

### select 데이터

row_factory 를 사용하여 class 생성자로 레코드를 가공하도록 하였다

```python
from psycopg import connection, sql
from psycopg.rows import class_row

def select_data(conn: connection.Connection) -> TestRow | None:
	# use row_factory with pydantic BaseModel
	with conn.cursor(row_factory=class_row(TestRow)) as cur:
		# Query the database and obtain data as Python objects.
		cur.execute(
			sql.SQL("SELECT * FROM {}").format(sql.Identifier("public", "test"))
		)
		obj = cur.fetchone()
		if not obj:
			print("No data!")
			return None
		return obj

def main(DATABASE_URL: str):
	conn = connect_db(DATABASE_URL)
	with conn:
		try: 
			row = select_data(conn)
			print("\n==>", row)
		except psycopg.Error as e:
			print("Unable to insert data!", e)
```

## 2️⃣ psycopg 비동기식 사용 (async)

psycopg3 에서는 asyncpg 등을 사용하지않아도 자체적으로 비동기 처리를 지원한다

### 1. DB 접속

`autocommit=False` 상태이면 반드시 `commit()` 을 해주어야 반영된다.

- autocommit 옵션의 기본값을 False 이다

```python
import asyncio
import psycopg
from psycopg import AsyncConnection, sql

async def connect_db(DATABASE_URL: str) # autocommit=False
	"""Connect to the PostgreSQL database server
	
	참고:
		- current_date => datetime.date (time 데이터 없음)
		- current_timestamp, now() => datetime.datetime
	"""
	try:
		aconn = await AsyncConnection.connect(DATABSE_URL) # autocommit=False
		async with aconn:
			# Test connection
			async with aconn.cursor() as cur:
				await cur.execute("select current_timestamp, 'ok' as result")
				data = await cur.fetchone()
				print("data[0]:", data[0], type(data[0]))
				print("data[1]:", data[1], type(data[1]))
				assert data[1] == "ok"
		return True
	except psycopg.Error as e:
		print("Unable to connect!", e)
	return False

async def main(DATABASE_URL: str):
	if not await connect_db(DATABASE_URL):
		print("cannot connect to db!")
		return

if __name__ == "__main__":
	# load_dotenv()
	DATABASE_URL = os.getenv("DATABASE_URL")
	
	# async call from main
	loop = asyncio.get_event_loop()
	asyncio.run(main(DATABASE_URL))
	loop.close()
```

### 2. insert 데이터

비동기 연결 객체는 with 구문과 강하게 연결되어있어서 함께 사용해야 한다.

- 다름 함수로 연결 객체를 전달하려면 with  구문 아래에서 해야 한다.

```python
async def insert_data(aconn: AsyncConnection, data: TestRow):
	asycn with aconn.cursor() as cur:
		stmt = sql.SQL(
			"INSERT INTO {} (logdate, info, phones, content) VALUES ({}, {}, {}, {})"
		).format(
			sql.Identifier("test_async"),  # table name
			sql.Literal(data.logdate),
			sql.Literal(json.dumps(data.info)),  # json -> str
			sql.Literal(data.phones),
			data.content,
		)
		# print("SQL:", stmt.as_string)
		await cur.execute(stmt)
	await aconn.commit()

async def main(DATABASE_URL: str):
	aconn = awaut AsyncConnection.connect(DATABASE_URL)
	async with aconn:
		loop.add_signal_handler(signal.SIGINT, aconn.cancel)
		try:
			await insert_data(aconn, data)
			rows = await select_data(aconn)
			for record in rows:
				print(record)
		except psycopg.Error as e:
			print("Unable to insert data!", e)

if __name__ == "__main__":
	# load_dotenv()
	DATABASE_URL = os.getenv("DATABASE_URL")
	
	# async call from main
	loop = asyncio.get_event_loop()
	asyncio.run(main(DATABASE_URL))	
	loop.close()
```

### 3. select 데이터

async/await 키워드 외에 특별한 사항은 없다. (asyncio 인터페이스)

```python
async def select_data(aconn: AsyncConnection) -> TestRow | None:
	# use row_factory with pydantic BaseModel
	async with aconn.cursor(row_factory=class_row(TestTow)) as cur:
		# Query the database and obtain data as Python objects.
		await cur.execute(
			sql.SQL("SELECT * FROM {}").format(sql.Identifier("public", "test_async"))
		)
		return await cur.fetchall()
```


## 3️⃣ psycopg_pool 을 이용한 fastapi 와 연계 사용

fastapi 에서 psycopg 를 사용하기 위해서는 psycopg_pool 이 필요하다.

### 1. AsyncConnectionPool 생성 및 해제

- startup 이벤트: AsyncConnectionPool 생성
- shutdown 이벤트: AsyncConnectionPool 해제
- endpoint 사용시: Pool 에서 async connection 객체를 얻어 사용

```python
from psycopg_pool import AsyncConnectionPool
from fastapi import FastAPI

app = FastAPI()

@app.on_event("startup")
def open_pool():
	"""create database connection pool"""
	app.state.pool = AsyncConnectionPool(DATABASE_URL, max_size=500)

@app.on_event("shutdown")
async def close_pool():
	"""close database connection pool"""
	await app.state.pool.close()
```

### 2. Pool 로 부터 비동기 연결 객체를 가져와 사용하기

```python
@app.get("/my_data")
async def get_my_data():
	return await my_query(app.state.pool)

async def my_query(pool: AsycnConnectionPool):
	async with pool.connection() as conn:
		async with conn.cursor()(row_factory=class_row(TestRow)) as cur:
			await cur.execute("SELECT * FORM public.test_async")
			rows = await cur.fetchall()
			return {"data": rows}

if __name__ == "__main__":
	uvicorn.run(app, host="172.0.0.1", port=8000)
```

