# Padrões de Projeto Existentes — ESM Forum

## Introdução

Mesmo em sistemas minimalistas com objetivo didático, padrões de projeto surgem naturalmente à medida que boas práticas de desenvolvimento são seguidas. A análise do código do ESM Forum revela três padrões presentes, alguns completos e outros parcialmente aplicados.

---

## Padrão 1: Module (Módulo) — Padrão Estrutural

**Onde está aplicado:** todos os arquivos do backend (`db.js`, `routes/perguntas.js`, `routes/respostas.js`, `app.js`)

**Descrição:**
O padrão Module é amplamente utilizado no ecossistema Node.js através do sistema CommonJS de `require/module.exports`. Ele encapsula código em unidades independentes com interface pública bem definida, escondendo detalhes de implementação.

**Exemplo no código:**
```js
// db.js — exporta apenas a conexão configurada
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database('./forum.db');
module.exports = db; // interface pública: apenas o objeto db

// routes/perguntas.js — consome o módulo sem conhecer sua implementação interna
const db = require('../db');
```

**Avaliação:** Implementação **completa e correta**. O padrão está presente em todos os módulos do sistema. Cada arquivo expõe apenas o necessário através de `module.exports`, mantendo o encapsulamento adequado.

---

## Padrão 2: Router / Front Controller — Padrão Comportamental

**Onde está aplicado:** `app.js` + `routes/perguntas.js` + `routes/respostas.js`

**Descrição:**
O Express Router implementa uma variação do padrão **Front Controller**: existe um ponto central de entrada (`app.js`) que delega as requisições para roteadores especializados com base no prefixo da URL. Isso evita que um único arquivo gerencie todos os endpoints.

**Exemplo no código:**
```js
// app.js — ponto central de entrada (Front Controller)
const perguntasRouter = require('./routes/perguntas');
const respostasRouter = require('./routes/respostas');

app.use('/perguntas', perguntasRouter); // delega para o roteador especializado
app.use('/respostas', respostasRouter);

// routes/perguntas.js — roteador especializado
const router = express.Router();
router.get('/', listarPerguntas);
router.post('/', criarPergunta);
router.delete('/:id', deletarPergunta);
module.exports = router;
```

**Avaliação:** Implementação **parcialmente completa**. O roteamento está correto, mas o padrão Front Controller completo prevê também um mecanismo de middleware centralizado para autenticação, logging e tratamento de erros — o que não está implementado no sistema atual.

**Oportunidade de melhoria:**
```js
// app.js — adicionar middlewares centralizados (completaria o padrão)
app.use(express.json());
app.use(loggerMiddleware);       // logging centralizado
app.use(authMiddleware);         // autenticação centralizada
app.use('/perguntas', perguntasRouter);
app.use(errorHandlerMiddleware); // tratamento de erros centralizado
```

---

## Padrão 3: Singleton — Padrão Criacional

**Onde está aplicado:** `db.js`

**Descrição:**
O padrão Singleton garante que uma classe tenha apenas uma instância e fornece um ponto global de acesso a ela. No ESM Forum, a conexão com o banco de dados é criada uma única vez em `db.js` e compartilhada por todo o sistema via `require()`.

**Exemplo no código:**
```js
// db.js — instância única criada uma vez
const db = new sqlite3.Database('./forum.db', (err) => {
  if (err) console.error(err.message);
});

module.exports = db; // sempre a mesma instância é exportada
```

```js
// routes/perguntas.js e routes/respostas.js
const db = require('../db'); // ambos recebem a mesma instância
```

**Por que é Singleton:**
O sistema de módulos do Node.js garante que o resultado de `require()` seja cacheado após a primeira execução. Ou seja, não importa quantas vezes `require('../db')` seja chamado — sempre retorna o mesmo objeto `db`. Isso é um Singleton natural provido pelo runtime.

**Avaliação:** Implementação **completa e idiomática** para o ecossistema Node.js. Não há necessidade de implementar o padrão manualmente, pois o `require` já o garante. Funciona corretamente para conexões SQLite de arquivo único.

---

## Resumo

| Padrão | Categoria | Arquivos | Status |
|---|---|---|---|
| Module | Estrutural | todos os `.js` | ✅ Completo |
| Front Controller / Router | Comportamental | `app.js`, `routes/` | ⚠️ Parcial |
| Singleton | Criacional | `db.js` | ✅ Completo |

---

*Documento preparado para o Projeto Final de Engenharia de Software — Parte 3, Iteração 2.*
