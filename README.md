# Documentação – Construção de API com FastAPI
**fastapi-api-build-docs**

## 🟦 Introdução

Repositório dedicado à documentação do desenvolvimento de APIs utilizando FastAPI, com explicações passo a passo e exemplos práticos.

O objetivo desta documentação é apresentar a construção de uma API desde o início, seguindo práticas recomendadas, estrutura organizada e exemplos funcionais. Cada etapa será acompanhada de explicações detalhadas, permitindo que qualquer pessoa — iniciante ou intermediária — compreenda os fundamentos da arquitetura e implementação.

## 📁 Estrutura inicial do projeto

A estrutura de diretórios usada neste projeto segue exatamente o modelo exibido na imagem enviada:

<img width="806" height="538" alt="image" src="https://github.com/user-attachments/assets/a93f4914-41c9-42b7-93e3-6192ae613f96" />

## 📌 Observação:
> Esse formato mostra um projeto FastAPI simples, com uma pasta static para arquivos HTML/CSS/JS e um arquivo principal main.py, onde está toda a configuração do servidor, conexão com banco de dados e definição do modelo.

## 🧩 Código completo do arquivo main.py

Abaixo está o código exatamente como aparece na imagem:

```python
from fastapi import FastAPI
from sqlalchemy import create_engine, Column, String
from sqlalchemy.orm import sessionmaker, declarative_base
from pydantic import BaseModel
from fastapi.staticfiles import StaticFiles

app = FastAPI()

#Usando o Render
#database_url = "postgresql://user:password@host.oregon-postgres.render.com/database"

#Usando conexão local
database_url = "postgresql://postgres:aluno@localhost/postgres"

engine = create_engine(database_url)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class Curso(Base):
    __tablename__ = "curso"
    cod_curso = Column(String(5), primary_key=True, nullable=False)
    nome_curso = Column(String(60), nullable=False)
    carga_horaria = Column(String(10), nullable=False)
    modalidade = Column(String(20), nullable=False)
    nivel = Column(String(20), nullable=False)
    coordenador = Column(String(60), nullable=False)
    descricao = Column(String(100), nullable=False)
    turno = Column(String(1), nullable=False)
    data_inicio = Column(String(10), nullable=False)
    data_fim = Column(String(10), nullable=False)


Base.metadata.create_all(bind=engine)


class ModelCurso(BaseModel):
    cod_curso: str
    nome_curso: str
    carga_horaria: str
    modalidade: str
    nivel: str
    coordenador: str
    descricao: str
    turno: str
    data_inicio: str
    data_fim: str
  
    model_config = {"from_attributes": True}


@app.get("/curso")
def listarCursos():
    conexao = SessionLocal()
    cursos = conexao.query(Curso).all()
    conexao.close()
    return {"cursos": [ModelCurso.model_validate(c).dict() for c in cursos]}


@app.get("/curso/{cod_curso}")
def listarPorID(cod_curso: str):
    conexao = SessionLocal()
    c = conexao.query(Curso).filter(Curso.cod_curso == cod_curso).first()
    conexao.close()
    if c:
        return {"cursos": ModelCurso.model_validate(c).dict()}
    return {"mensagem": "Curso não encontrado"}


@app.post("/curso")
def criarCurso(c: ModelCurso):
    conexao = SessionLocal()
    novoCurso = Curso(**c.dict())
    conexao.add(novoCurso)
    conexao.commit()
    conexao.refresh(novoCurso)
    conexao.close()
    return {"mensagem": "Curso criado com sucesso!", "curso": ModelCurso.model_validate(novoCurso).dict()}


app.mount("/", StaticFiles(directory="static", html=True), name="static")

```
## 📄 Criação do arquivo principal

Antes de prosseguir, é necessário criar um arquivo chamado **main.py** na raiz do projeto.  
Esse arquivo será responsável por toda a lógica da API, incluindo configuração, conexão com o banco de dados e definição dos endpoints.

## Explicando passo a passo o código

### 📦 Realizando as Importações

As primeiras linhas trazem todas as bibliotecas necessárias para que a API funcione corretamente. Aqui são importados os módulos responsáveis pela criação da aplicação FastAPI, conexão com o banco via SQLAlchemy, validação de dados com Pydantic e configuração para servir arquivos estáticos.

```python
from fastapi import FastAPI
from sqlalchemy import create_engine, Column, String
from sqlalchemy.orm import sessionmaker, declarative_base
from pydantic import BaseModel
from fastapi.staticfiles import StaticFiles
```

### ⚙️ Instalando as Dependências

Antes de executar a API, instale as dependências essenciais.
Esses comandos devem ser executados no terminal do VS Code (ou no terminal do seu sistema):

```powershell
pip install "fastapi[standard]"
pip install sqlalchemy
pip install pydantic
pip install psycopg2
```

---

### 🔧 Configurações Iniciais

Aqui é feita a inicialização da aplicação e a definição da string de conexão (database_url) que aponta para o PostgreSQL.
Você pode utilizar tanto a conexão local quanto a conexão remota do Render.
- engine cria a conexão com o banco.
- SessionLocal controla as transações.
- Base serve de base para os modelos ORM.
  
```python
app = FastAPI()

#Usando o Render
#database_url = "postgresql://user:password@host.oregon-postgres.render.com/database"

#Usando conexão local
database_url = "postgresql://postgres:aluno@localhost/postgres"

engine = create_engine(database_url)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

---

### 🧱 Modelo do Banco

O modelo abaixo representa a tabela curso no banco PostgreSQL.
Cada atributo corresponde a uma coluna da tabela.

```python
class Curso(Base):
    __tablename__ = "curso"
    cod_curso = Column(String(5), primary_key=True, nullable=False)
    nome_curso = Column(String(60), nullable=False)
    carga_horaria = Column(String(10), nullable=False)
    modalidade = Column(String(20), nullable=False)
    nivel = Column(String(20), nullable=False)
    coordenador = Column(String(60), nullable=False)
    descricao = Column(String(100), nullable=False)
    turno = Column(String(1), nullable=False)
    data_inicio = Column(String(10), nullable=False)
    data_fim = Column(String(10), nullable=False)
```

---

### 🧱 Criando a Tabela no Banco

Caso o banco ainda não contenha a tabela, o comando abaixo irá criá-la automaticamente:

```python
Base.metadata.create_all(bind=engine)
```

---

### 🧩 Modelo Pydantic

Este modelo garante a validação dos dados recebidos e enviados pela API.

```python
class ModelCurso(BaseModel):
    cod_curso: str
    nome_curso: str
    carga_horaria: str
    modalidade: str
    nivel: str
    coordenador: str
    descricao: str
    turno: str
    data_inicio: str
    data_fim: str
  
    model_config = {"from_attributes": True}
```
---

### 🔍 GET — Consultar Todos os Registros
Endpoint que retorna todos os registros cadastrados:

```python
@app.get("/curso")
def listarCursos():
    conexao = SessionLocal()
    cursos = conexao.query(Curso).all()
    conexao.close()
    return {"cursos": [ModelCurso.model_validate(c).dict() for c in cursos]}
```

---

### 🔎 GET — Consultar por ID
Busca um registro específico usando uma primary key:

```python
@app.get("/curso/{cod_curso}")
def listarPorID(cod_curso: str):
    conexao = SessionLocal()
    c = conexao.query(Curso).filter(Curso.cod_curso == cod_curso).first()
    conexao.close()
    if c:
        return {"cursos": ModelCurso.model_validate(c).dict()}
    return {"mensagem": "Curso não encontrado"}
```
---

### ➕ POST — Cadastrar Novo Registro
Cria um novo registro no banco:

```python
@app.post("/curso")
def criarCurso(c: ModelCurso):
    conexao = SessionLocal()
    novoCurso = Curso(**c.dict())
    conexao.add(novoCurso)
    conexao.commit()
    conexao.refresh(novoCurso)
    conexao.close()
    return {"mensagem": "Curso criado com sucesso!", "curso": ModelCurso.model_validate(novoCurso).dict()}
```

---

### ▶️Conectando com a pasta Static
Esta etapa serve para permitir que sua API entregue arquivos HTML, CSS e JavaScript diretamente para o navegador.

```python
app.mount("/", StaticFiles(directory="static", html=True), name="static")
```

### ▶️Executando o Servidor
Para rodar a API:

```powershell
fastapi dev main.py
```



