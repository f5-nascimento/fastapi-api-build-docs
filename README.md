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

## 📁 Criando a pasta `static` e os arquivos iniciais

Nesta etapa, vamos estruturar a pasta responsável por armazenar os arquivos estáticos da aplicação — ou seja, os arquivos que serão servidos diretamente pelo navegador: HTML, CSS e JavaScript. Essa pasta será utilizada posteriormente quando conectarmos o FastAPI ao frontend básico.

---

### 📂 Criando a pasta `static`

Na raiz do projeto (no mesmo nível do arquivo `main.py`), crie uma pasta chamada `static`.  

Dentro dessa pasta, crie os seguintes arquivos:

- **index.html**
- **style.css**
- **script.js**

Esses arquivos serão utilizados para montar a interface inicial que será exibida quando acessarmos a aplicação pelo navegador.

## 📝 Código completo do arquivo `index.html`

Abaixo está o arquivo `index.html` que ficará dentro da pasta `static`.  
Ele contém o formulário de cadastro e a tabela onde os cursos serão exibidos.

```html

<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Gerenciar Cursos</title>
  <link rel="stylesheet" href="style.css">
  </head>
<body>
  <h1>Gerenciar Cursos</h1>

  <form id="formCurso">
    <input type="text" id="cod_curso" placeholder="Código do Curso (ex: INF01)">
    <input type="text" id="nome_curso" placeholder="Nome do Curso">
    <input type="text" id="carga_horaria" placeholder="Carga Horária">
    <input type="text" id="modalidade" placeholder="Modalidade">
    <input type="text" id="nivel" placeholder="Nível">
    <input type="text" id="coordenador" placeholder="Coordenador">
    <input type="text" id="descricao" placeholder="Descrição">
    <input type="text" id="turno" placeholder="Turno (M/V/N)">
    <input type="text" id="data_inicio" placeholder="Data Início">
    <input type="text" id="data_fim" placeholder="Data Fim">
    <br>
    <button type="button" id="btnCadastrar"><i class="fas fa-plus"></i>Cadastrar</button>
    <button type="button" id="btnConsultar"><i class="fas fa-search"></i>Consultar</button>
    <button type="button" id="btnLimpar"><i class="fas fa-eraser"></i>Limpar</button>
  </form>

  <h2>Lista de Cursos</h2>

  <table>
    <thead>
      <tr>
        <th>Código</th>
        <th>Nome</th>
        <th>Carga Horária</th>
        <th>Modalidade</th>
        <th>Nível</th>
        <th>Coordenador</th>
        <th>Descrição</th>
        <th>Turno</th>
        <th>Início</th>
        <th>Fim</th>
      </tr>
    </thead>
    <tbody id="tabelaCursos"></tbody>
  </table>

  <script src="script.js"></script>
</body>
</html>

```

## 🧩 Código completo do arquivo `script.js`

Abaixo está o conteúdo do arquivo `script.js`, responsável por realizar as operações de comunicação entre o frontend e a API FastAPI, incluindo consultas, cadastro e atualização automática da tabela.

```javascript
const apiUrl = "/curso";

    async function carregarTabela() {
      try {
        const response = await fetch(apiUrl);
        const data = await response.json();
        const lista = data.cursos || [];
        const tabela = document.getElementById('tabelaCursos');
        tabela.innerHTML = '';

        lista.forEach(curso => {
          const row = `
            <tr>
              <td>${curso.cod_curso}</td>
              <td>${curso.nome_curso}</td>
              <td>${curso.carga_horaria}</td>
              <td>${curso.modalidade}</td>
              <td>${curso.nivel}</td>
              <td>${curso.coordenador}</td>
              <td>${curso.descricao}</td>
              <td>${curso.turno}</td>
              <td>${curso.data_inicio}</td>
              <td>${curso.data_fim}</td>
            </tr>`;
          tabela.innerHTML += row;
        });
      } catch (err) {
        console.error("Erro ao carregar tabela:", err);
      }
    }


    async function cadastrarCurso() {
      const novoCurso = {
        cod_curso: document.getElementById('cod_curso').value,
        nome_curso: document.getElementById('nome_curso').value,
        carga_horaria: document.getElementById('carga_horaria').value,
        modalidade: document.getElementById('modalidade').value,
        nivel: document.getElementById('nivel').value,
        coordenador: document.getElementById('coordenador').value,
        descricao: document.getElementById('descricao').value,
        turno: document.getElementById('turno').value,
        data_inicio: document.getElementById('data_inicio').value,
        data_fim: document.getElementById('data_fim').value
      };

      try {
        const response = await fetch(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(novoCurso)
        });

        if (!response.ok) {
          alert("Erro ao cadastrar curso. Verifique os dados.");
          return;
        }

        alert("Curso cadastrado com sucesso!");
        limparFormulario();
        carregarTabela();
      } catch (err) {
        console.error('Erro ao cadastrar curso:', err);
      }
    }


    async function consultarCurso() {
      const codigo = document.getElementById('cod_curso').value;
      if (!codigo) {
        alert("Digite o código do curso para consultar!");
        return;
      }

      try {
        const response = await fetch(`${apiUrl}/${codigo}`);
        if (!response.ok) {
          alert("Curso não encontrado!");
          return;
        }

        const data = await response.json();
        const curso = data.cursos;
        if (curso) {
          document.getElementById('nome_curso').value = curso.nome_curso;
          document.getElementById('carga_horaria').value = curso.carga_horaria;
          document.getElementById('modalidade').value = curso.modalidade;
          document.getElementById('nivel').value = curso.nivel;
          document.getElementById('coordenador').value = curso.coordenador;
          document.getElementById('descricao').value = curso.descricao;
          document.getElementById('turno').value = curso.turno;
          document.getElementById('data_inicio').value = curso.data_inicio;
          document.getElementById('data_fim').value = curso.data_fim;
        }
      } catch (err) {
        console.error('Erro ao consultar curso:', err);
      }
    }

  
    function limparFormulario() {
      document.querySelectorAll('#formCurso input').forEach(input => input.value = '');
    }


    document.getElementById('btnCadastrar').addEventListener('click', cadastrarCurso);
    document.getElementById('btnConsultar').addEventListener('click', consultarCurso);
    document.getElementById('btnLimpar').addEventListener('click', limparFormulario);


    carregarTabela();

```


## 🎨 Código completo do arquivo `style.css`

Abaixo está o arquivo responsável pela estilização da interface.  
Ele controla o layout, cores, fontes, tamanhos, espaçamentos, bordas e comportamento visual dos elementos da página.

```css
/* ====== Estilo geral ====== */
body {
  font-family: "Segoe UI", Arial, sans-serif;
  margin: 20px;
  background-color: #f8f9fb;
  color: #333;
}

/* ====== Títulos ====== */
h1 {
  color: #3f51b5;
  font-size: 28px;
  margin-bottom: 20px;
}

h2 {
  color: #444;
  margin-top: 40px;
}

/* ====== Formulário ====== */
form {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  width: calc(33% - 10px);
  font-size: 14px;
  transition: border-color 0.3s;
}

input:focus {
  border-color: #3f51b5;
  outline: none;
}

/* ====== Botões ====== */
button {
  padding: 10px 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  font-size: 14px;
  color: white;
  display: inline-flex;
  align-items: center;
  transition: 0.2s ease;
}

button i {
  margin-right: 6px;
}

/* Cores dos botões */
#btnCadastrar {
  background-color: #4caf50;
}
#btnConsultar {
  background-color: #2196f3;
}
#btnLimpar {
  background-color: #9e9e9e;
}

button:hover {
  opacity: 0.85;
  transform: translateY(-1px);
}

/* ====== Tabela ====== */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 25px;
  background: #fff;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

th,
td {
  padding: 10px 12px;
  text-align: left;
  border-bottom: 1px solid #e0e0e0;
}

th {
  background-color: #3f51b5;
  color: white;
  font-weight: 600;
}

tr:hover {
  background-color: #f5f7fa;
}

/* ====== Campo de pesquisa (se tiver) ====== */
#pesquisa {
  width: 250px;
  margin-top: 15px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
}

#pesquisa:focus {
  border-color: #3f51b5;
  outline: none;
}





