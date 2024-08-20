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

```t
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
```
## adicionando um banco de dados ao servidor

-Vá na pasta `app.ts` e substitua o código que está lá pelo código abaixo:

```t
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
``` 
## testando a inserção de dados

- No vscode, vá na barra da esquerda e clique no ultimo ícone, nisso, voce pesquisará po `REST client` e baixe-o.
- Crie uma pasta `ts.http` e insira o seguinte código:

```
{
  "name": "John Doe",
  "email": "
}

- E se tudo ocorrer bem, voce verá a resposta com o usuário e o email inseridos

 ## listando os usuários

 - Para listar os usuários, voce deverá colocar a rota `/users` no servidor, para isso, vá na pasta `app.ts` e adicione o seguinte código no final:
 
 ```
 app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

## editando os usuários

- crie a rota '/usaers/:id' adicionando o seguinte código na pasta app.ts
```
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});
```

## deletando um usuário

- adicione na rota '/users/:id' o seguinte código também na pasta app.ts

```
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```

## criando a pasta public
- crie uma pasta com o nome public fora do src, e em seguida copie e cole o seguite código:

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>
```

- agora como último passo, copie e cole o seguinte código na pasta `app.ts`

```t
import express from 'express'
import cors from 'cors'
import { connect } from './database'

const port = 3333
const app = express()

app.use(cors())
app.use(express.json())
app.use(express.static(__dirname + '/../public'))

app.get('/users', async (req, res) => {
  const db = await connect()
  const users = await db.all('SELECT * FROM users')
  res.json(users)
})

app.post('/users', async (req, res) => {
  const db = await connect()
  const { name, email } = req.body
  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email])
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID])
  res.json(user)
})

app.put('/users/:id', async (req, res) => {
  const db = await connect()
  const { name, email } = req.body
  const { id } = req.params
  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id])
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id])
  res.json(user)
})

app.delete('/users/:id', async (req, res) => {
  const db = await connect()
  const { id } = req.params
  await db.run('DELETE FROM users WHERE id = ?', [id])
  res.json({ message: 'User deleted' })
})

app.listen(port, () => {
  console.log(`Server running on port ${port}`)
})
```