import os, sqlite3
from memori import Memori
from openai import OpenAI
from pymongo import MongoClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

def get_sqlite_conn(): return sqlite3.connect("memori.db")
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
mem = Memori(conn=get_sqlite_conn).llm.register(client)
mem.attribution(entity_id="user_123", process_id="support_agent")
mem.config.storage.build()
client.chat.completions.create(model="gpt-4o-mini", messages=[{"role": "user", "content": "My favorite color is blue."}])
mem.augmentation.wait()
response = client.chat.completions.create(model="gpt-4o-mini", messages=[{"role": "user", "content": "What's my favorite color?"}])
print(response.choices[0].message.content)
engine = create_engine("postgresql+psycopg://user:password@localhost:5432/memori", pool_pre_ping=True)
SessionLocal = sessionmaker(bind=engine)
pg_mem = Memori(conn=SessionLocal)
pg_mem.config.storage.build()
mongo_db = MongoClient(os.getenv("MONGODB_URI", "mongodb://localhost:27017"))["memori"]
mongo_mem = Memori(conn=lambda: mongo_db)
mongo_mem.config.storage.build()
