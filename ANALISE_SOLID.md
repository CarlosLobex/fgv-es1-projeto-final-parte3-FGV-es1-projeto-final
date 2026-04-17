# Análise SOLID — ESM Forum Backend

## Introdução

Os princípios SOLID são cinco diretrizes para design de software orientado a objetos que promovem código mais legível, manutenível e extensível:

- **S** — Single Responsibility Principle (SRP)
- **O** — Open/Closed Principle (OCP)
- **L** — Liskov Substitution Principle (LSP)
- **I** — Interface Segregation Principle (ISP)
- **D** — Dependency Inversion Principle (DIP)

---

## Parte 1: Pontos Positivos (trechos que seguem SOLID)

### ✅ Ponto Positivo 1 — SRP: Separação entre rotas de perguntas e respostas

**Princípio:** Single Responsibility Principle

**Trecho:**
```
backend/
├── routes/
│   ├── perguntas.js   ← responsável apenas por operações de perguntas
│   └── respostas.js   ← responsável apenas por operações de respostas
```

**Explicação:**
Cada arquivo de rota possui uma única responsabilidade bem definida. `perguntas.js` cuida exclusivamente do CRUD de perguntas, enquanto `respostas.js` cuida do CRUD de respostas. Não há mistura de responsabilidades entre os dois módulos. Isso segue o SRP: cada módulo tem apenas um motivo para mudar — se a regra de negócio de perguntas mudar, apenas `perguntas.js` precisa ser alterado.

---

### ✅ Ponto Positivo 2 — SRP: Isolamento da configuração do banco em módulo próprio

**Princípio:** Single Responsibility Principle

**Trecho:**
```js
// db.js — responsabilidade única: configurar e exportar a conexão com o banco
const sqlite3 = require('sqlite3').verbose();

const db = new sqlite3.Database('./forum.db', (err) => {
  if (err) console.error('Erro ao conectar ao banco:', err.message);
  else console.log('Conectado ao SQLite.');
});

module.exports = db;
```

**Explicação:**
A configuração da conexão com o banco de dados está isolada em um único arquivo (`db.js`). As rotas não sabem como o banco é configurado — apenas importam e usam a conexão. Isso significa que trocar de SQLite para PostgreSQL, por exemplo, exigiria mudanças apenas em `db.js`, sem tocar nas rotas.

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

### ❌ Violação 1 — SRP: Rota mistura lógica de negócio com acesso a dados

**Princípio violado:** Single Responsibility Principle

**Trecho atual:**
```js
// routes/perguntas.js
router.post('/', (req, res) => {
  const { titulo, corpo, autor } = req.body;

  // Validação (lógica de negócio)
  if (!titulo || !corpo || !autor) {
    return res.status(400).json({ error: 'Campos obrigatórios ausentes' });
  }

  // Acesso direto ao banco (lógica de dados) — dentro da rota
  db.run(
    'INSERT INTO perguntas (titulo, corpo, autor) VALUES (?, ?, ?)',
    [titulo, corpo, autor],
    function (err) {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ id: this.lastID });
    }
  );
});
```

**Problema:** A função de rota está fazendo três coisas ao mesmo tempo: (1) receber e interpretar a requisição HTTP, (2) validar os dados de entrada e (3) executar a query SQL. Isso viola o SRP porque qualquer mudança na lógica de persistência ou de validação força a edição do mesmo bloco de código.

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

### ❌ Violação 2 — DIP: Dependência direta do módulo de banco nas rotas

**Princípio violado:** Dependency Inversion Principle

**Trecho atual:**
```js
// routes/perguntas.js
const db = require('../db'); // dependência direta da implementação concreta

router.get('/', (req, res) => {
  db.all('SELECT * FROM perguntas ORDER BY id DESC', [], (err, rows) => {
    // ...
  });
});
```

**Problema:** As rotas dependem diretamente do módulo `db` (implementação concreta do SQLite). Se o banco de dados precisar ser trocado ou mockado nos testes, seria necessário alterar todas as rotas. O DIP diz que módulos de alto nível (rotas) não devem depender de módulos de baixo nível (banco); ambos devem depender de abstrações.

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
