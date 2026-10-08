# Fincore-gastos

Sistema de controle de gastos pessoais, desenvolvido para a matéria de **Programação Orientada a Objetos (Java)**. É um CRUD baseado no nosso trabalho de banco de dados, o **FinCore**.

- **Backend + API REST:** Java com Spring Boot
- **Frontend:** React (Vite)
- **Banco de dados:** MySQL (em container Docker)
- **CI:** GitHub Actions

> Backend e API são o mesmo projeto: o Spring Boot expõe os endpoints que o React consome.

```
React (localhost:5173)  ->  Spring Boot (localhost:8080)  ->  MySQL (localhost:3307)
                            Controller -> Service -> Repository
```

---

## Pré-requisitos

Instale antes de começar:

| Ferramenta | Versão | Observação |
|---|---|---|
| Git | qualquer recente | |
| Docker Desktop | qualquer recente | precisa estar **aberto e rodando** |
| JDK | 21 ou superior | a variável `JAVA_HOME` precisa apontar pro JDK 21+ |
| Node.js | 22 (LTS) | |

Para conferir:

```bash
git --version
docker --version
java -version
node -v
```

---

## Como rodar o projeto

### 1. Clonar

```bash
git clone https://github.com/Soarezzsemj/Fincore-gastos.git
cd Fincore-gastos
```

### 2. Subir o banco de dados

Abra o Docker Desktop, espere ele ficar com **Engine running** e rode na raiz do projeto:

```bash
docker compose up -d
docker compose ps
```

O container `gastos-db` deve aparecer como `healthy`.

### 3. Subir o backend

```bash
cd backend
./mvnw spring-boot:run        # Linux/Mac
.\mvnw spring-boot:run        # Windows (PowerShell)
```

Na primeira vez ele baixa as dependências, então demora um pouco. Para conferir, abra:

http://localhost:8080/actuator/health

Deve responder com `"status": "UP"`.

### 4. Subir o frontend

Em **outro terminal**:

```bash
cd frontend
npm install
npm run dev
```

Abra http://localhost:5173

---

## Banco de dados

| Campo | Valor |
|---|---|
| Host | `localhost` |
| Porta | **3307** (não é a 3306) |
| Banco | `gastos` |
| Usuário | `app` |
| Senha | `app123` |

Essas credenciais são **só de desenvolvimento**. Para conectar por um cliente (DataGrip, DBeaver, MySQL Workbench), use os dados acima.

Usamos a porta **3307** para não conflitar com um MySQL que algum de nós possa ter instalado no computador (porta 3306).

Comandos úteis:

```bash
docker compose down        # para o banco (os dados ficam guardados)
docker compose down -v     # para e APAGA os dados (zera o banco)
docker compose logs db     # mostra o log do banco
```

### Migrations (Flyway)

As tabelas são criadas e atualizadas por scripts SQL versionados em:

```
backend/src/main/resources/db/migration/
```

Regras:

- Nome do arquivo: `V<numero>__<descricao>.sql`, com **dois** underlines. Exemplo: `V2__criar_lancamento.sql`.
- **Nunca edite** uma migration que já foi para a `main`. Se precisar mudar algo, crie uma nova (`V3__...`).
- Antes de criar uma migration, **avise no grupo qual número você vai usar**, para dois colegas não criarem o mesmo `V2`.
- Quando o backend sobe, o Flyway aplica sozinho as migrations que faltam.

---

## Estrutura do repositório

```
Fincore-gastos/
├── backend/               Spring Boot (API, regras de negócio, acesso ao banco)
│   └── src/main/java/br/ucb/gastos_backend/
├── frontend/              React + Vite
├── docker-compose.yml     MySQL para desenvolvimento
├── .github/workflows/     Pipelines do GitHub Actions (CI)
└── README.md
```

Camadas do backend:

- **controller:** recebe as requisições HTTP e devolve JSON
- **service:** regras de negócio
- **repository:** acesso ao banco (Spring Data JPA)
- **model / entity:** classes de domínio (aqui entram herança, polimorfismo e encapsulamento)

---

## Fluxo de trabalho em grupo

**Nunca fazemos push direto na `main`.** Ela é protegida: só entra código por Pull Request com o CI passando.

```bash
# 1. Atualize a main
git checkout main
git pull

# 2. Crie sua branch
git checkout -b feature/nome-da-tarefa

# 3. Programe e faça commits pequenos
git add .
git commit -m "Adiciona CRUD de categoria"

# 4. Envie a branch
git push -u origin feature/nome-da-tarefa
```

Depois:

1. Abra o **Pull Request** no GitHub.
2. Espere o **CI** ficar verde (Backend e Frontend).
3. Peça para um colega revisar.
4. Faça o merge e apague a branch.

Combinados:

- **Avisem no grupo** qual tarefa cada um pegou, para ninguém fazer a mesma coisa.
- Nome de branch: `feature/...` para funcionalidade e `fix/...` para correção.
- Mensagem de commit: curta, em português, começando com verbo (`Adiciona`, `Corrige`, `Remove`).
- Um PR por tarefa, pequeno e fácil de revisar.
- Rodem o projeto na máquina antes de abrir o PR.

---

## CI (GitHub Actions)

A cada Pull Request, o GitHub roda automaticamente:

- **Backend (Spring Boot):** sobe um MySQL de teste, roda `./mvnw verify` (compila e executa os testes)
- **Frontend (React):** `npm ci` e `npm run build`

Se algum ficar vermelho, clique em **Details** ao lado do check, leia o final do log e corrija antes do merge.

---

## Problemas comuns

**`failed to connect to the docker API` / `dockerDesktopLinuxEngine`**
O Docker Desktop não está rodando. Abra ele e espere o status **Engine running**.

**`ports are not available` na porta 3306 ou 3307**
Outra coisa está usando a porta. Descubra quem com `netstat -ano | findstr :3307` (Windows). Se for outro serviço, mude a porta da esquerda em `docker-compose.yml` (`"3307:3306"`) e a URL no `application.properties`.

**`release version 21 not supported`**
O Maven está usando um Java antigo. Rode `.\mvnw -version` e veja qual Java ele usa. Aponte o `JAVA_HOME` para um JDK 21 ou superior e abra um terminal novo.

**`Communications link failure` ao subir o backend**
O banco não está de pé. Rode `docker compose ps` e confira se o `gastos-db` está `healthy`.

**`Access denied for user`**
Provavelmente você está conectando num MySQL instalado no Windows (porta 3306) em vez do container (porta 3307). Confira a porta. Se mudou as credenciais do compose, rode `docker compose down -v` para recriar o banco.

**O banco ficou bagunçado**
`docker compose down -v` e depois `docker compose up -d`. O Flyway recria tudo ao subir o backend.
