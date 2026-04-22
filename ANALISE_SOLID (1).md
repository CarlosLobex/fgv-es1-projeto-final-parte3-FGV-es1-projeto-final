# Análise SOLID — ESM Forum Backend

## Introdução

Os princípios SOLID são cinco diretrizes para design de software orientado a objetos que promovem código mais legível, manutenível e extensível:

- **S** — Single Responsibility Principle (SRP)
- **O** — Open/Closed Principle (OCP)
- **L** — Liskov Substitution Principle (LSP)
- **I** — Interface Segregation Principle (ISP)
- **D** — Dependency Inversion Principle (DIP)

A análise foi realizada sobre os arquivos principais do backend: **`server.js`** (rotas, configuração do Express e acesso ao banco) e **`modelo.js`** (estrutura de dados e queries).

---

## Parte 1: Pontos Positivos (trechos que seguem SOLID)

### ✅ Ponto Positivo 1 — SRP: Separação entre lógica de perguntas e respostas

**Princípio:** Single Responsibility Principle

**Trecho:**
```
server.js
  ├── rotas de perguntas  (GET /perguntas, POST /perguntas, DELETE /perguntas/:id)
  └── rotas de respostas  (GET /respostas/:id, POST /respostas)

modelo.js
  └── funções de acesso ao banco (queries SQLite)
```

**Explicação:**
Dentro de `server.js`, as rotas de perguntas e respostas são registradas separadamente com prefixos distintos, e as operações de banco ficam isoladas em `modelo.js`. Cada seção tem uma única responsabilidade bem definida. Isso segue o SRP: se a lógica de perguntas mudar, apenas as rotas de perguntas em `server.js` precisam ser alteradas; se as queries mudarem, apenas `modelo.js` é afetado.

---

### ✅ Ponto Positivo 2 — SRP: Isolamento das queries de banco em `modelo.js`

**Princípio:** Single Responsibility Principle

**Trecho:**
```js
// modelo.js — responsabilidade única: queries e acesso ao banco de dados
const db = new sqlite3.Database('./forum.db');

function listarPerguntas(callback) {
  db.all('SELECT * FROM perguntas ORDER BY id DESC', [], callback);
}

function criarPergunta(titulo, corpo, autor, callback) {
  db.run('INSERT INTO perguntas (titulo, corpo, autor) VALUES (?, ?, ?)',
    [titulo, corpo, autor], callback);
}

module.exports = { listarPerguntas, criarPergunta, /* ... */ };
```

**Explicação:**
As operações de banco de dados ficam centralizadas em `modelo.js`, separadas da lógica de roteamento HTTP em `server.js`. Isso significa que trocar de SQLite para outro banco exigiria mudanças apenas em `modelo.js`, sem tocar nas rotas. Cada arquivo tem um único motivo para mudar.

---

### ✅ Ponto Positivo 3 — OCP: Rotas extensíveis via Express Router

**Princípio:** Open/Closed Principle

**Trecho:**
```js
// app.js
const perguntasRouter = require('./routes/perguntas');
const respostasRouter = require('./routes/respostas');

app.use('/perguntas', perguntasRouter);
app.use('/respostas', respostasRouter);
```

**Explicação:**
A estrutura com `express.Router()` permite adicionar novos módulos de rota (ex: `/tags`, `/votos`, `/usuarios`) sem modificar o código existente em `app.js` além de registrar a nova rota. O sistema está **aberto para extensão** (novos endpoints) e **fechado para modificação** (as rotas existentes não precisam ser alteradas para acomodar as novas).

---

## Parte 2: Oportunidades de Melhoria (violações de SOLID)

### ❌ Violação 1 — SRP: Rota em `server.js` mistura lógica de negócio com acesso a dados

**Princípio violado:** Single Responsibility Principle

**Trecho atual:**
```js
// server.js — rota POST de perguntas
app.post('/perguntas', (req, res) => {
  const { titulo, corpo, autor } = req.body;

  // Validação (lógica de negócio) — dentro da rota
  if (!titulo || !corpo || !autor) {
    return res.status(400).json({ error: 'Campos obrigatórios ausentes' });
  }

  // Acesso direto ao banco via modelo.js (lógica de dados) — dentro da rota
  modelo.criarPergunta(titulo, corpo, autor, function (err) {
    if (err) return res.status(500).json({ error: err.message });
    res.json({ id: this.lastID });
  });
});
```

**Problema:** A função de rota está fazendo três coisas ao mesmo tempo: (1) receber e interpretar a requisição HTTP, (2) validar os dados de entrada e (3) acionar a persistência. Isso viola o SRP porque qualquer mudança na lógica de validação ou na lógica de persistência força a edição do mesmo bloco em `server.js`.

**Proposta de melhoria:**
```js
// models/Pergunta.js — responsabilidade: acesso a dados
class PerguntaModel {
  static criar(titulo, corpo, autor) {
    return new Promise((resolve, reject) => {
      db.run(
        'INSERT INTO perguntas (titulo, corpo, autor) VALUES (?, ?, ?)',
        [titulo, corpo, autor],
        function (err) {
          if (err) reject(err);
          else resolve({ id: this.lastID });
        }
      );
    });
  }
}

// services/PerguntaService.js — responsabilidade: regras de negócio
class PerguntaService {
  static async criar(titulo, corpo, autor) {
    if (!titulo || !corpo || !autor) {
      throw new Error('Campos obrigatórios ausentes');
    }
    return PerguntaModel.criar(titulo, corpo, autor);
  }
}

// routes/perguntas.js — responsabilidade: HTTP
router.post('/', async (req, res) => {
  try {
    const { titulo, corpo, autor } = req.body;
    const resultado = await PerguntaService.criar(titulo, corpo, autor);
    res.json(resultado);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});
```

---

### ❌ Violação 2 — DIP: `server.js` depende diretamente da implementação concreta em `modelo.js`

**Princípio violado:** Dependency Inversion Principle

**Trecho atual:**
```js
// server.js
const modelo = require('./modelo'); // dependência direta da implementação concreta

app.get('/perguntas', (req, res) => {
  modelo.listarPerguntas((err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});
```

**Problema:** As rotas em `server.js` dependem diretamente do módulo `modelo.js` (implementação concreta do acesso ao SQLite). Se o banco de dados precisar ser trocado ou mockado nos testes, seria necessário alterar `modelo.js` e potencialmente `server.js`. O DIP diz que módulos de alto nível (rotas) não devem depender de módulos de baixo nível (banco); ambos devem depender de abstrações.

**Proposta de melhoria:**
```js
// interfaces/IPerguntaRepository.js — abstração (contrato)
class IPerguntaRepository {
  listarTodas() { throw new Error('Método não implementado'); }
  buscarPorId(id) { throw new Error('Método não implementado'); }
  criar(titulo, corpo, autor) { throw new Error('Método não implementado'); }
}

// repositories/SQLitePerguntaRepository.js — implementação concreta
class SQLitePerguntaRepository extends IPerguntaRepository {
  listarTodas() {
    return new Promise((resolve, reject) => {
      db.all('SELECT * FROM perguntas ORDER BY id DESC', [], (err, rows) => {
        if (err) reject(err); else resolve(rows);
      });
    });
  }
  // ... outros métodos
}

// routes/perguntas.js — depende da abstração, não da implementação
const repository = new SQLitePerguntaRepository(); // injetado ou configurado em app.js

router.get('/', async (req, res) => {
  const perguntas = await repository.listarTodas();
  res.json(perguntas);
});
```

Nessa estrutura, seria possível substituir `SQLitePerguntaRepository` por `PostgresPerguntaRepository` ou `MockPerguntaRepository` (para testes) sem alterar as rotas.

---

*Documento preparado para o Projeto Final de Engenharia de Software — Parte 3, Iteração 1.*
