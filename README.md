import os
import sqlite3
from memori import Memori
from openai import OpenAI
from pymongo import MongoClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker


def get_sqlite_conn():
    return sqlite3.connect("memori.db")


def run_sqlite_chat():
    client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

    mem = Memori(conn=get_sqlite_conn).llm.register(client)
    mem.attribution(entity_id="user_123", process_id="support_agent")
    mem.config.storage.build()

    client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": "My favorite color is blue."}],
    )

    mem.augmentation.wait()  # needed in short scripts so background memory finishes

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": "What's my favorite color?"}],
    )
    print(response.choices[0].message.content)


def setup_postgres():
    engine = create_engine(
        os.getenv("DATABASE_URL", "postgresql+psycopg://user:password@localhost:5432/memori"),
        pool_pre_ping=True,
    )
    SessionLocal = sessionmaker(bind=engine)
    pg_mem = Memori(conn=SessionLocal)
    pg_mem.config.storage.build()
    return pg_mem


def setup_mongo():
    mongo_client = MongoClient(os.getenv("MONGODB_URI", "mongodb://localhost:27017"))
    db = mongo_client["memori"]
    mongo_mem = Memori(conn=lambda: db)
    mongo_mem.config.storage.build()
    return mongo_mem


if __name__ == "__main__":
    run_sqlite_chat()

