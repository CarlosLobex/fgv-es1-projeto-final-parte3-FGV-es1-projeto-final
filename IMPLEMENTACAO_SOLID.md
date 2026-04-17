# Implementação com SOLID — Busca de Perguntas por Palavra-Chave

## Funcionalidade Implementada

**Busca de perguntas por palavra-chave** — o usuário pode pesquisar perguntas pelo título ou corpo usando um termo de busca. A implementação segue os princípios SOLID, organizando o código em camadas bem definidas.

---

## Estrutura de Arquivos Criada

```
backend/
├── repositories/
│   └── PerguntaRepository.js      ← acesso ao banco (DIP + SRP)
├── services/
│   └── BuscaService.js            ← lógica de negócio (SRP + OCP)
├── controllers/
│   └── BuscaController.js         ← controle HTTP (SRP)
└── routes/
    └── busca.js                   ← registro de rota (SRP)
```

---

## Código Implementado

### 1. `repositories/PerguntaRepository.js`

```js
// Responsabilidade única: acesso ao banco de dados para perguntas
// Aplica SRP (apenas persistência) e DIP (isolamento da implementação concreta)

const db = require('../db');

class PerguntaRepository {
  /**
   * Retorna todas as perguntas cujo título ou corpo contém o termo buscado.
   * @param {string} termo - Palavra-chave para busca
   * @returns {Promise<Array>}
   */
  buscarPorTermo(termo) {
    return new Promise((resolve, reject) => {
      const sql = `
        SELECT * FROM perguntas
        WHERE LOWER(titulo) LIKE LOWER(?) OR LOWER(corpo) LIKE LOWER(?)
        ORDER BY id DESC
      `;
      const param = `%${termo}%`;
      db.all(sql, [param, param], (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
      });
    });
  }

  /**
   * Retorna todas as perguntas (sem filtro).
   * @returns {Promise<Array>}
   */
  listarTodas() {
    return new Promise((resolve, reject) => {
      db.all('SELECT * FROM perguntas ORDER BY id DESC', [], (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
      });
    });
  }
}

module.exports = new PerguntaRepository();
```

---

### 2. `services/BuscaService.js`

```js
// Responsabilidade única: lógica de negócio da busca
// Aplica SRP (regras de negócio isoladas), OCP (pode ser estendido com novos filtros)
// e DIP (depende da abstração do repository, não do banco diretamente)

const perguntaRepository = require('../repositories/PerguntaRepository');

class BuscaService {
  /**
   * Executa a busca de perguntas com validação do termo.
   * Se o termo estiver vazio, retorna todas as perguntas.
   * @param {string} termo
   * @returns {Promise<Array>}
   */
  async buscarPerguntas(termo) {
    if (!termo || termo.trim() === '') {
      // Termo vazio: retorna tudo (comportamento esperado pelo frontend)
      return perguntaRepository.listarTodas();
    }

    const termoSanitizado = termo.trim();

    if (termoSanitizado.length < 2) {
      throw new Error('O termo de busca deve ter pelo menos 2 caracteres.');
    }

    return perguntaRepository.buscarPorTermo(termoSanitizado);
  }
}

module.exports = new BuscaService();
```

---

### 3. `controllers/BuscaController.js`

```js
// Responsabilidade única: receber requisição HTTP e retornar resposta
// Aplica SRP (nenhuma lógica de negócio ou banco aqui)
// Aplica DIP (depende do service, não do repository diretamente)

const buscaService = require('../services/BuscaService');

class BuscaController {
  async buscar(req, res) {
    try {
      const { q } = req.query;
      const resultados = await buscaService.buscarPerguntas(q);
      res.json(resultados);
    } catch (err) {
      const status = err.message.includes('pelo menos') ? 400 : 500;
      res.status(status).json({ error: err.message });
    }
  }
}

module.exports = new BuscaController();
```

---

### 4. `routes/busca.js`

```js
// Responsabilidade única: registrar as rotas de busca no Express
// Aplica OCP: novas rotas de busca (por tag, por autor) podem ser adicionadas
// sem modificar as rotas existentes

const express = require('express');
const router = express.Router();
const buscaController = require('../controllers/BuscaController');

// GET /busca?q=termo
router.get('/', (req, res) => buscaController.buscar(req, res));

module.exports = router;
```

---

### 5. Registro em `app.js`

```js
// Adição ao app.js existente — sem modificar rotas já existentes (OCP)
const buscaRouter = require('./routes/busca');
app.use('/busca', buscaRouter);
```

---

### 6. Integração no Frontend (React)

```js
// src/services/api.js — chamada ao endpoint de busca
export async function buscarPerguntas(termo) {
  const url = termo
    ? `http://localhost:3000/busca?q=${encodeURIComponent(termo)}`
    : `http://localhost:3000/perguntas`;

  const response = await fetch(url);
  if (!response.ok) throw new Error('Erro na busca');
  return response.json();
}
```

```jsx
// src/components/CampoBusca.jsx
import { useState } from 'react';
import { buscarPerguntas } from '../services/api';

export function CampoBusca({ onResultados }) {
  const [termo, setTermo] = useState('');

  const handleBusca = async (e) => {
    const valor = e.target.value;
    setTermo(valor);
    const resultados = await buscarPerguntas(valor);
    onResultados(resultados);
  };

  return (
    <input
      type="text"
      placeholder="Buscar perguntas..."
      value={termo}
      onChange={handleBusca}
    />
  );
}
```

---

## Como Cada Princípio SOLID Foi Aplicado

### S — Single Responsibility Principle
Cada módulo tem exatamente uma responsabilidade:
- `PerguntaRepository` → apenas queries SQL
- `BuscaService` → apenas regras de negócio (validação do termo, decisão de buscar tudo ou filtrar)
- `BuscaController` → apenas interpretar a requisição HTTP e formatar a resposta
- `busca.js` → apenas mapear URLs para controllers

### O — Open/Closed Principle
- Para adicionar busca por **tag**, basta criar `TagRepository.buscarPorTag()` e `BuscaService.buscarPorTag()` — sem modificar o código existente
- Para adicionar busca por **autor**, o mesmo padrão se aplica
- O `app.js` recebe novas rotas sem modificar as existentes

### D — Dependency Inversion Principle
- `BuscaController` não conhece o banco de dados — depende de `BuscaService`
- `BuscaService` não conhece o SQLite — depende de `PerguntaRepository`
- Se o banco mudar de SQLite para PostgreSQL, apenas `PerguntaRepository` precisa ser alterado; o `BuscaService` e o `BuscaController` permanecem intactos

---

## Exemplo de Uso

```bash
# Buscar perguntas com a palavra "node"
GET http://localhost:3000/busca?q=node

# Resposta
[
  { "id": 3, "titulo": "Como usar async/await no Node.js?", "corpo": "...", "autor": "Carlos" },
  { "id": 1, "titulo": "Node.js vs Deno — qual usar em 2024?", "corpo": "...", "autor": "Ana" }
]

# Campo vazio — retorna todas as perguntas
GET http://localhost:3000/busca?q=

# Termo muito curto — erro 400
GET http://localhost:3000/busca?q=a
{ "error": "O termo de busca deve ter pelo menos 2 caracteres." }
```

---

*Documento preparado para o Projeto Final de Engenharia de Software — Parte 3, Iteração 1.*
