---
title: "FastAPIでCRUD操作を実装する簡単なWebサーバーの構築方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["FastAPI", "Python", "CRUD", "API", "MacOS"]
published: false
---

# FastAPIでCRUD操作を実装する簡単なWebサーバーの構築方法

こんにちは！この記事では、MacOS環境でFastAPIを使用してCRUD（Create, Read, Update, Delete）操作を実装する簡単なWebサーバーの構築方法について解説します。

## はじめに

FastAPIは、Pythonで書かれた高速なWebフレームワークで、APIの開発に特化しています。特徴として:

- 高速: Starlette と Pydantic をベースにしており、NodeJSやGoと同等のパフォーマンスを発揮します
- 直感的: 簡単な構文とAPIドキュメントの自動生成機能があります
- 型ヒント: Pythonの型ヒントを活用した開発が可能です
- 自動ドキュメント: OpenAPIとSwaggerUIによる自動ドキュメント生成機能があります

それでは、実際に開発環境のセットアップから始めていきましょう。

## 1. 開発環境のセットアップ

### 必要なソフトウェアのインストール

MacOSでは、Homebrewを使用してPythonをインストールするのが一般的です。まだHomebrewがインストールされていない場合は、以下のコマンドでインストールしてください。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

次に、Pythonをインストールします:

```bash
brew install python
```

### プロジェクトの作成

まず、プロジェクト用のディレクトリを作成し、仮想環境を設定します:

```bash
# プロジェクトディレクトリを作成
mkdir fastapi-crud-demo
cd fastapi-crud-demo

# 仮想環境を作成して有効化
python -m venv venv
source venv/bin/activate
```

### 必要なパッケージのインストール

FastAPIと関連パッケージをインストールします:

```bash
pip install fastapi uvicorn sqlalchemy
```

## 2. データベースの設定

SQLAlchemyを使用してSQLiteデータベースを設定します。まず、`database.py`ファイルを作成します:

```python
from sqlalchemy import create_engine, Column, Integer, String, MetaData
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# SQLiteデータベースのURL
SQLALCHEMY_DATABASE_URL = "sqlite:///./crud.db"

# エンジンの作成
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)

# セッションの作成
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Baseクラスの作成
Base = declarative_base()

# DB接続のためのDependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## 3. モデルの作成

次に、データモデルを定義します。`models.py`ファイルを作成しましょう:

```python
from sqlalchemy import Column, Integer, String
from database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String)
```

## 4. スキーマの定義

Pydanticを使用してAPIのリクエストとレスポンスのスキーマを定義します。`schemas.py`ファイルを作成します:

```python
from pydantic import BaseModel

class ItemBase(BaseModel):
    name: str
    description: str = None

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int

    class Config:
        orm_mode = True
```

## 5. CRUDロジックの実装

CRUD操作のロジックを実装します。`crud.py`ファイルを作成します:

```python
from sqlalchemy.orm import Session
import models
import schemas

# アイテムを作成する
def create_item(db: Session, item: schemas.ItemCreate):
    db_item = models.Item(name=item.name, description=item.description)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

# 全てのアイテムを取得する
def get_items(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Item).offset(skip).limit(limit).all()

# 特定のアイテムを取得する
def get_item(db: Session, item_id: int):
    return db.query(models.Item).filter(models.Item.id == item_id).first()

# アイテムを更新する
def update_item(db: Session, item_id: int, item: schemas.ItemCreate):
    db_item = db.query(models.Item).filter(models.Item.id == item_id).first()
    if db_item:
        db_item.name = item.name
        db_item.description = item.description
        db.commit()
        db.refresh(db_item)
    return db_item

# アイテムを削除する
def delete_item(db: Session, item_id: int):
    db_item = db.query(models.Item).filter(models.Item.id == item_id).first()
    if db_item:
        db.delete(db_item)
        db.commit()
    return db_item
```

## 6. APIエンドポイントの実装

最後に、APIエンドポイントを実装します。`main.py`ファイルを作成します:

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
import crud
import models
import schemas
from database import engine, get_db

# データベーステーブルの作成
models.Base.metadata.create_all(bind=engine)

# FastAPIアプリケーションのインスタンス化
app = FastAPI(title="FastAPI CRUD Demo")

# ルートエンドポイント
@app.get("/")
def read_root():
    return {"message": "Welcome to FastAPI CRUD Demo"}

# Create: アイテムを作成する
@app.post("/items/", response_model=schemas.Item)
def create_item(item: schemas.ItemCreate, db: Session = Depends(get_db)):
    return crud.create_item(db=db, item=item)

# Read: 全てのアイテムを取得する
@app.get("/items/", response_model=list[schemas.Item])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    items = crud.get_items(db, skip=skip, limit=limit)
    return items

# Read: 特定のアイテムを取得する
@app.get("/items/{item_id}", response_model=schemas.Item)
def read_item(item_id: int, db: Session = Depends(get_db)):
    db_item = crud.get_item(db, item_id=item_id)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item

# Update: アイテムを更新する
@app.put("/items/{item_id}", response_model=schemas.Item)
def update_item(item_id: int, item: schemas.ItemCreate, db: Session = Depends(get_db)):
    db_item = crud.update_item(db, item_id=item_id, item=item)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item

# Delete: アイテムを削除する
@app.delete("/items/{item_id}", response_model=schemas.Item)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    db_item = crud.delete_item(db, item_id=item_id)
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item
```

## 7. アプリケーションの実行

アプリケーションを実行するには、以下のコマンドを使用します:

```bash
uvicorn main:app --reload
```

これで、サーバーがlocalhostの8000番ポートで起動します。以下のURLでアクセス可能です:

- API: http://127.0.0.1:8000
- APIドキュメント（Swagger UI）: http://127.0.0.1:8000/docs
- APIドキュメント（ReDoc）: http://127.0.0.1:8000/redoc

## 8. APIのテスト

APIをテストするためにcurlコマンドを使用することができます。以下はいくつかの例です:

### アイテムの作成 (Create)

```bash
curl -X 'POST' \
  'http://127.0.0.1:8000/items/' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "テストアイテム",
  "description": "これはテスト用のアイテムです"
}'
```

### アイテム一覧の取得 (Read)

```bash
curl -X 'GET' \
  'http://127.0.0.1:8000/items/' \
  -H 'accept: application/json'
```

### 特定のアイテムの取得 (Read)

```bash
curl -X 'GET' \
  'http://127.0.0.1:8000/items/1' \
  -H 'accept: application/json'
```

### アイテムの更新 (Update)

```bash
curl -X 'PUT' \
  'http://127.0.0.1:8000/items/1' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "更新されたアイテム",
  "description": "アイテムの説明を更新しました"
}'
```

### アイテムの削除 (Delete)

```bash
curl -X 'DELETE' \
  'http://127.0.0.1:8000/items/1' \
  -H 'accept: application/json'
```

## まとめ

この記事では、FastAPIを使用してCRUD操作を備えた簡単なWebサーバーを構築する方法を紹介しました。FastAPIは直感的なAPIであり、自動ドキュメント生成機能やPythonの型ヒントを活用した開発が可能で、高速なパフォーマンスを発揮します。

この基本的な実装を拡張して、認証機能の追加や、より複雑なデータモデルの実装、テストの追加などを行うことができます。

ぜひ、FastAPIを使ってあなた自身のWebアプリケーションを開発してみてください！

## 参考リンク

- [FastAPI公式ドキュメント](https://fastapi.tiangolo.com/)
- [SQLAlchemy公式ドキュメント](https://www.sqlalchemy.org/)
- [Pydantic公式ドキュメント](https://pydantic-docs.helpmanual.io/)
