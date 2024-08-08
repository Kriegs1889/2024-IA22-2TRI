# 2024-IA22-2TRI

# Iniciando um projeto Node.js com TypeScript
- Abra o github e crie um repositório,para criar um repositório, va na barra da esquerda, clique no botao "new", vá em "repository name" e de um nome a seu seu repositório, depois,  vá para a opção "add a README file" e a habilite. Após isso, apenas clique na opção "create repository" e parabens, voce criou um repositório :]

- Depois disso, crie uma pasta, abra o github, va na opção "clone git repository" vá no repositório que voce criou, vá na opção "code", copie o link e coloque na barra de pesquisa a cima. 

- Nisso, selecione a pasta que voce criou antes. Agora, va na opção "terminal", depois na opção "abrir novo terminal" e escreva as linhas a baixo para criar o seu servidor. (não escreva a ultima linha, ela não vai rodar, voce tem que ir em "scr" e criar umarquivo app.ts manualmente)

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Configuranado o `tsconfig.json`

- Com o server criado, voce irá para a pasta "tsconfig.json" voce irá adicionar `"outDir": "./",` para `"outDir": "./dist",` e adicione a linha `"rootDir": "./src",`nas linhas 58 e 59  sem deixar como comentário e deve ficar mais ou menos assim:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

## Configurando o `package.json`
- Agora, vá para a pasta "package.json" e substitua o que estiver dentro das chaves de "scripts" pelo o que está dentro das cheves deste código abaixo:

```json
"scripts": {
  "dev": "npx nodemon src/app.ts"
}
```

## Criando arquivo inicial do servidor

- Vá para a pasta app.ts que voce havia criado antes e escreva o código abaixo:

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inicializando o servidor

- Abra  terminal novamente, e digite 'npm run dev' 
- E se tudo ocorrer bem, você verá a mensagem "Server running on port 3333" no terminal.

## Testando o servidor

- Abra o navegador e digite `http://localhost:3333`, você deverá ver a mensagem `Hello World`. Parabens voce agora tem um local host :D

## configurendo o banco de dados

- Abra o vscode, ligue o server, e crie um arquivo `database.ts` dentro da pasta `src` e adicione o seguinte código:

import { open, Database} from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  instance = db;
  return db;
}

## adicionando um banco de daod ao servidor

-Vá na pasta `app.ts` e substitua o código que está lá pelo código abaixo:


import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});

## testando a inserção de dados

- No vscode, vá na barra da esquerda e clique no ultimo ícone, nisso, voce pesquisará po `REST client` e baixe-o.
- Crie uma pasta `ts.http` e insira o seguinte código:

{
  "name": "John Doe",
  "email": "
}

- E se tudo ocorrer bem, voce verá a resposta com o usuário e o email inseridos

 ## listando os usuários

 - Para listar os usuários, voce deverá colocar a rota `/users` no servidor, para isso, vá na pasta `app.ts` e adicione o seguinte código no final:

 app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});

## editando os usuários

- crie a rota '/usaers/:id' adicionando o seguinte código na pasta app.ts

app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});

## deletando um usuário

- adicione na rota '/users/:id' o seguinte código também na pasta app.ts

app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
