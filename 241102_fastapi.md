## ドメイン駆動設計（DDD）とオニオンアーキテクチャについての解説


### ドメイン駆動設計（DDD）とは？

#### 1. ドメインの概念
ドメイン駆動設計（DDD）は、システム開発におけるビジネスの知識をプログラムに反映させ、複雑なビジネスロジックをモデル化する設計手法です。**ドメイン**とは、システムが解決しようとする特定の業務領域や問題領域を指します。DDDでは、ドメインを中心に据え、エンティティや値オブジェクト、リポジトリ、サービスなど、さまざまなコンポーネントを組み合わせて、業務知識を正確に表現することを目指します。

#### 2. DDDの主要な概念
DDDには、主に以下の4つの基本概念が含まれます。

- **エンティティ（Entity）**: 一意のIDを持ち、ライフサイクルがあるオブジェクト。例えば、ユーザーや商品など、明確に識別可能な要素。
- **値オブジェクト（Value Object）**: 不変の小さなオブジェクトで、エンティティのプロパティとして利用されることが多い。例えば、金額や住所など。
- **アグリゲート（Aggregate）**: 関連するエンティティと値オブジェクトを一つにまとめ、外部からの操作を一貫して行うための集合単位。これにより、データの整合性を保つ。
- **リポジトリ（Repository）**: データの保存や取得を行うインターフェースで、データの永続化処理を隠蔽。ドメインオブジェクトとデータベースとの橋渡しをする役割を持つ。

#### 3. ドメインサービス
業務ロジックがエンティティや値オブジェクトに収まらない場合、そのロジックを**ドメインサービス**として切り出します。ドメインサービスは、ドメインオブジェクトに対してビジネスルールや操作を実行するための役割を担います。

---

### オニオンアーキテクチャとは？

オニオンアーキテクチャは、依存関係が内側から外側にのみ向かう構造を持ち、システムを層ごとに分割するアーキテクチャの一つです。DDDと組み合わせることで、ビジネスロジックの独立性が高まり、外部の技術やフレームワークからの影響を受けにくい構造を実現します。

#### 1. オニオンアーキテクチャの層構造

オニオンアーキテクチャは、以下のような層に分けられます。

1. **ドメイン層（Domain Layer）**: システムの中心となるビジネスロジックが含まれる層。エンティティや値オブジェクト、ドメインサービスなど、DDDで定義した主要なコンポーネントがここに位置します。
2. **アプリケーション層（Application Layer）**: ドメイン層にアクセスし、ビジネスロジックの調整を行う層。アプリケーションのユースケースやサービスが含まれます。
3. **インフラストラクチャ層（Infrastructure Layer）**: 外部との接続部分を扱う層で、リポジトリや外部サービスの連携などが含まれます。データベースやメッセージキュー、サードパーティAPIとのやりとりもここで行います。
4. **ユーザーインターフェース層（Presentation Layer）**: エンドユーザーとシステムがやりとりするインターフェースを提供します。APIエンドポイントやユーザーインターフェースが含まれ、システム全体の外側に位置します。

#### 2. オニオンアーキテクチャの依存関係

オニオンアーキテクチャでは、依存関係が内側から外側にのみ向かうように設計されています。これにより、インフラ層が変わってもドメイン層には影響を与えません。また、ビジネスロジックがどの層にも依存しないため、テストや再利用がしやすくなります。

---

### DDDとオニオンアーキテクチャの利点

1. **変更に強い**: ビジネスロジックが独立しているため、フレームワークやデータベースの変更による影響を最小限に抑えることができます。
2. **テストが容易**: ドメイン層が外部依存から独立しているため、ビジネスロジックを単体テストで検証しやすいです。
3. **再利用性が高い**: ビジネスルールをシステム全体の中心に配置することで、異なるプロジェクトでもロジックを再利用しやすくなります。

---

### 具体的なプロジェクト構成例

以下は、FastAPIでDDDとオニオンアーキテクチャを用いたプロジェクト構成の例です。

```plaintext
fastapi-googlekeep-integration/
├── app/
│   ├── api/                # APIエンドポイント
│   │   ├── v1/
│   │   │   └── googlekeep.py
│   ├── domain/             # ドメイン層
│   │   ├── models/         # エンティティや値オブジェクト
│   │   │   └── note.py
│   ├── infrastructure/     # インフラ層（リポジトリや外部API連携）
│   │   └── googlekeep_repository.py
│   ├── usecases/           # アプリケーション層
│   │   └── create_note_usecase.py
│   └── main.py             # エントリーポイント
└── pyproject.toml
```

この構造の各フォルダについて簡単に説明します。

- **domain**: ビジネスロジックを管理する層で、`note.py`にはメモを表すエンティティが定義されます。
- **usecases**: ユースケースやサービスの実装が含まれ、ビジネスロジックを組み合わせて具体的な処理を行います。
- **infrastructure**: データベース接続や外部サービス（ここではGoogle Keep）との連携を管理します。
- **api**: FastAPIのエンドポイントを定義し、外部からのリクエストを受け付けます。

---


## FAT-API作成
まず、基本的な動作を確認できる最小限の構成で、FastAPIアプリケーションを構築していきます。この段階では、Google Keep APIの認証や連携部分は含めず、シンプルに動作確認ができるアプリケーションを提示します。その後、Google Keepとの連携部分を追加していきます。

### ステップ1: プロジェクト構成

まずは、プロジェクトのディレクトリ構成を以下のように作成します。

```plaintext
fastapi-googlekeep-integration/
├── app/
│   ├── api/
│   │   ├── v1/
│   │   │   └── googlekeep.py
│   ├── domain/
│   │   └── models/
│   │       └── note.py
│   ├── infrastructure/
│   │   └── googlekeep_repository.py
│   ├── usecases/
│   │   └── create_note_usecase.py
│   └── main.py
└── pyproject.toml
```

### ステップ2: 各ファイルの実装

#### 1. `pyproject.toml`
Poetryで依存関係を管理するための設定ファイルです。以下の内容で作成してください。

```toml
[tool.poetry]
name = "fastapi-googlekeep-integration"
version = "0.1.0"
description = "API to receive title and content, integrating with Google Keep"
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.8"
fastapi = "^0.78.0"
uvicorn = "^0.17.6"

[tool.poetry.dev-dependencies]
pytest = "^7.1.2"
```

#### 2. `app/main.py`
FastAPIアプリケーションのエントリーポイントを作成します。

```python
from fastapi import FastAPI
from app.api.v1.googlekeep import router as googlekeep_router

app = FastAPI(title="Google Keep Integration API")

# エンドポイントを登録
app.include_router(googlekeep_router, prefix="/api/v1")

@app.get("/")
async def root():
    return {"message": "Welcome to the Google Keep Integration API"}
```

#### 3. `app/api/v1/googlekeep.py`
APIエンドポイントのコードを記述します。リクエストからタイトルとコンテンツを受け取り、メモを作成するユースケースを呼び出します。

```python
from fastapi import APIRouter
from app.usecases.create_note_usecase import CreateNoteUseCase
from app.infrastructure.googlekeep_repository import GoogleKeepRepository

router = APIRouter()

@router.post("/keep")
async def create_keep_entry(title: str, content: str):
    google_keep_repo = GoogleKeepRepository()
    use_case = CreateNoteUseCase(google_keep_repo)
    result = use_case.execute(title, content)
    return {"result": result}
```

#### 4. `app/domain/models/note.py`
メモのエンティティ（データモデル）を定義します。

```python
from datetime import datetime

class Note:
    def __init__(self, title: str, content: str):
        self.title = title
        self.content = content
        self.created_at = datetime.now()
```

#### 5. `app/usecases/create_note_usecase.py`
メモ作成のユースケースを定義します。このユースケースは、メモを作成し、リポジトリを通して保存を行います。

```python
from app.domain.models.note import Note
from app.infrastructure.googlekeep_repository import GoogleKeepRepository

class CreateNoteUseCase:
    def __init__(self, google_keep_repo: GoogleKeepRepository):
        self.google_keep_repo = google_keep_repo

    def execute(self, title: str, content: str):
        note = Note(title, content)
        return self.google_keep_repo.save(note)
```

#### 6. `app/infrastructure/googlekeep_repository.py`
Google Keepに保存する部分は、ここでは実装せず、簡単なメッセージ表示で動作確認をします。後ほど、Google Keep API連携の部分を実装します。

```python
from app.domain.models.note import Note

class GoogleKeepRepository:
    def save(self, note: Note):
        # Google Keep APIを使用してメモを保存するロジックを後で実装
        print(f"Saving to Google Keep: {note.title} - {note.content}")
        return {"status": "success", "title": note.title, "content": note.content}
```

### ステップ3: アプリケーションの実行

ここまでの構成でアプリケーションを実行し、動作確認を行います。以下の手順でFastAPIを起動します。

1. Poetry環境を起動し、仮想環境に入ります。

   ```bash
   poetry shell
   ```

2. Uvicornでアプリケーションを実行します。

   ```bash
   uvicorn app.main:app --reload
   ```

3. ブラウザで `http://127.0.0.1:8000/docs` にアクセスし、Swagger UIが表示されることを確認します。

4. `POST /api/v1/keep` エンドポイントで、テストデータ（タイトルとコンテンツ）を入力してリクエストを送信し、以下のようなレスポンスが返ることを確認します。

   ```json
   {
       "result": {
           "status": "success",
           "title": "テストタイトル",
           "content": "テストコンテンツ"
       }
   }
   ```

これで、基本的な動作確認が完了しました。次のステップでは、Google Keep APIとの連携部分を実装していきます。

