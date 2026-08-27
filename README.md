# Ficha OP — Hexatombe

Ficha de personagem online para as suas mesas de **Ordem Paranormal**.

Cada ficha fica guardada num **slot numerado** e pode ter uma **senha**. Todo mundo
que abrir o mesmo endereço enxerga os mesmos slots — é assim que a mesa inteira
compartilha as fichas. Também tem uma página `/portraits` com os retratos dos
personagens, ótima pra deixar numa segunda tela ou na live.

> Este repositório já vem com o site **compilado** (pasta `dist/`). Você não precisa
> "buildar" nada — só rodar o servidor e apontar pra um banco de dados MongoDB.

---

## Índice

1. [Como funciona (leia antes)](#como-funciona-leia-antes)
2. [Passo 0 — Banco de dados grátis (MongoDB Atlas)](#passo-0--banco-de-dados-grátis-mongodb-atlas)
3. [Opção A — Hospedar de graça a partir do seu fork (recomendado)](#opção-a--hospedar-de-graça-a-partir-do-seu-fork-recomendado)
4. [Opção B — Rodar no seu PC e jogar com os amigos](#opção-b--rodar-no-seu-pc-e-jogar-com-os-amigos)
5. [Opção C — Docker (self-host completo, banco incluso)](#opção-c--docker-self-host-completo-banco-incluso)
6. [Variáveis de ambiente](#variáveis-de-ambiente)
7. [Usando na mesa](#usando-na-mesa)
8. [Mantendo seu fork atualizado](#mantendo-seu-fork-atualizado)
9. [E o GitHub Pages?](#e-o-github-pages)
10. [Problemas comuns (FAQ)](#problemas-comuns-faq)

---

## Como funciona (leia antes)

O projeto tem duas partes, e **as duas rodam juntas no mesmo servidor**:

| Parte | O que é | Onde está |
|---|---|---|
| **Servidor** | App Node.js (Express) que salva/carrega fichas e serve o site | `dist/index.mjs` |
| **Site** | A interface que abre no navegador | `dist/public/` |

O servidor precisa de um **MongoDB** para guardar as fichas. É só isso de
infraestrutura: **1 processo Node + 1 banco MongoDB**.

Resumindo as opções deste tutorial:

| Você quer... | Use a | Precisa de |
|---|---|---|
| Um link fixo no ar 24/7 pra mesa toda | **Opção A** (fork + host grátis) | Conta no GitHub + MongoDB Atlas + conta no host (Render/Railway) |
| Subir rapidinho só durante a sessão de hoje | **Opção B** (seu PC + túnel) | Node instalado + MongoDB Atlas |
| Controlar tudo numa máquina/servidor seu | **Opção C** (Docker) | Docker instalado |

---

## Passo 0 — Banco de dados grátis (MongoDB Atlas)

Serve para as Opções **A** e **B**. (Na Opção C o banco já vem junto.)

1. Crie uma conta em **<https://www.mongodb.com/cloud/atlas/register>**.
2. Crie um cluster **grátis** (tipo **M0**). Escolha uma região perto de você.
3. Em **Database Access** → **Add New Database User**: crie um usuário e senha
   (anote os dois; evite caracteres estranhos na senha pra não ter que "escapar").
4. Em **Network Access** → **Add IP Address** → **Allow access from anywhere**
   (`0.0.0.0/0`). É o jeito mais simples para hosts na nuvem funcionarem.
5. Em **Database** → **Connect** → **Drivers** → copie a **connection string**.
   Ela se parece com:

   ```
   mongodb+srv://USUARIO:SENHA@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```

6. **Troque `USUARIO` e `SENHA`** pelos que você criou e acrescente o nome do banco
   (`fichaop`) logo depois do `.net/`:

   ```
   mongodb+srv://USUARIO:SENHA@cluster0.xxxxx.mongodb.net/fichaop?retryWrites=true&w=majority
   ```

Essa string final é o valor da variável **`MONGODB_URI`** que você vai usar nos
próximos passos.

---

## Opção A — Hospedar de graça a partir do seu fork (recomendado)

A ideia: você **forka** este repositório para a sua conta do GitHub e liga o fork a
um serviço que roda Node de graça. Toda vez que você atualizar o fork, o site
atualiza sozinho.

### A.1 — Forkar

1. No topo desta página do GitHub, clique em **Fork**.
2. Confirme. Agora existe `https://github.com/SEU-USUARIO/ProjetoHexatombe` na sua conta.

### A.2 — Subir no Render (passo a passo)

O [Render](https://render.com) tem um plano **Free** para web services e puxa o
código direto do GitHub.

1. Crie conta em **<https://dashboard.render.com/register>** e conecte sua conta do GitHub.
2. **New +** → **Web Service** → escolha o seu fork.
3. Preencha:
   - **Language / Runtime:** `Node`
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Instance Type:** `Free`
4. Em **Environment Variables**, adicione:
   - **Key:** `MONGODB_URI` — **Value:** a string que você montou no Passo 0
5. **Create Web Service** e aguarde o deploy (uns 1–3 min).
6. Quando aparecer **"Live"**, o Render te dá uma URL tipo
   `https://ficha-op.onrender.com`. **Esse é o link da sua mesa.** Teste abrindo
   `https://ficha-op.onrender.com/api/healthz` — deve responder `{"status":"ok"}`.

> Este repositório já inclui um arquivo **`render.yaml`**. Se preferir, use
> **New + → Blueprint**, escolha o fork, e o Render preenche tudo sozinho — você só
> informa o `MONGODB_URI`.

**Sobre o plano Free do Render:** o serviço "dorme" depois de ~15 min sem acesso.
O primeiro acesso depois disso demora ~30–60s pra acordar. Para uma mesa isso é
tranquilo — abra o link uns minutos antes de começar.

### A.3 — Alternativas ao Render

Todas funcionam do mesmo jeito: conectar o fork, definir `MONGODB_URI`, e usar
`npm install` / `npm start`.

| Serviço | Observações |
|---|---|
| **Railway** (`railway.app`) | Deploy direto do repo. Detecta Node automaticamente. Crédito mensal grátis. Defina `MONGODB_URI` em *Variables*. Pode até adicionar um plugin de MongoDB no próprio Railway. |
| **Koyeb** (`koyeb.com`) | Plano gratuito com 1 serviço. Buildpack Node. Start command `npm start`. |
| **Fly.io** (`fly.io`) | Precisa do CLI `flyctl`. Rode `fly launch` (ele cria o `fly.toml`), depois `fly secrets set MONGODB_URI="..."` e `fly deploy`. |
| **Cyclic / Northflank / etc.** | Mesma receita. |

> **Vercel e Netlify não servem** para este projeto: eles hospedam site estático /
> funções serverless, e aqui você precisa de um servidor Node rodando de forma
> contínua com conexão persistente ao MongoDB.

---

## Opção B — Rodar no seu PC e jogar com os amigos

Bom para "só a sessão de hoje" ou para testar.

### B.1 — Pré-requisitos

- **Node.js 20 ou mais novo** — <https://nodejs.org> (baixe a versão **LTS**).
- Um `MONGODB_URI` (faça o **Passo 0** acima).

### B.2 — Baixar e instalar

Se você **baixou o .zip** do GitHub, extraia e abra um terminal dentro da pasta.
Se prefere `git`:

```bash
git clone https://github.com/SEU-USUARIO/ProjetoHexatombe.git
cd ProjetoHexatombe
```

Instale a dependência do servidor:

```bash
npm install
```

### B.3 — Configurar o banco

Crie um arquivo chamado **`.env`** na raiz do projeto (do lado do `package.json`)
com o conteúdo:

```
MONGODB_URI=mongodb+srv://USUARIO:SENHA@cluster0.xxxxx.mongodb.net/fichaop?retryWrites=true&w=majority
PORT=8080
```

(Há um `.env.example` de referência no repositório.)

### B.4 — Rodar

```bash
npm start
```

Deve aparecer no terminal:

```
INFO: Server listening
    port: 8080
```

Abra **<http://localhost:8080>** no navegador. Funcionou? Então bora compartilhar.

### B.5 — Deixar os amigos entrarem

**Se todos estão na mesma casa / mesma rede Wi-Fi:**

1. Descubra o IP local do seu PC:
   - Windows: `ipconfig` → procure **IPv4 Address** (algo como `192.168.0.15`)
   - macOS/Linux: `ip addr` ou `ifconfig`
2. Os amigos abrem `http://192.168.0.15:8080` (troque pelo seu IP).
3. Pode ser preciso **liberar a porta 8080 no firewall** do seu sistema.

**Se os amigos estão pela internet (caso mais comum):** use um *túnel*. Ele cria
um endereço público temporário que aponta pro seu `localhost:8080`.

- **Cloudflare Tunnel** (não precisa criar conta):

  ```bash
  cloudflared tunnel --url http://localhost:8080
  ```

  Ele imprime uma URL `https://algo-aleatorio.trycloudflare.com` — mande pros amigos.
  Instalação do `cloudflared`: <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/>

- **ngrok** (precisa de conta grátis):

  ```bash
  ngrok http 8080
  ```

  Copie a URL `https://....ngrok-free.app`.

O túnel só existe enquanto esse comando estiver aberto e o `npm start` rodando.
Fechou o terminal, acabou o link. Para algo permanente, use a **Opção A**.

---

## Opção C — Docker (self-host completo, banco incluso)

Sobe **o app + um MongoDB local** com um comando só. Bom se você tem um servidor
caseiro, uma VPS, ou só quer não depender do Atlas.

Pré-requisito: **Docker** + **Docker Compose** (o Docker Desktop já traz os dois).

Este repositório inclui `Dockerfile` e `docker-compose.yml`. Na raiz do projeto:

```bash
docker compose up -d
```

Pronto: <http://localhost:8080>. As fichas ficam salvas num volume Docker
(`mongo-data`), então sobrevivem a reinícios.

Para expor pra internet, combine com um túnel (ver **B.5**) ou coloque atrás de um
proxy reverso (Caddy, Nginx, Cloudflare Tunnel nomeado, etc.).

Parar tudo:

```bash
docker compose down
```

---

## Variáveis de ambiente

| Variável | Obrigatória? | Padrão | Para que serve |
|---|---|---|---|
| `MONGODB_URI` | **Sim** | — | String de conexão do MongoDB. Sem ela o servidor sobe mas nenhuma ficha carrega/salva. |
| `PORT` | Não | `8080` | Porta em que o servidor escuta. Hosts como Render/Railway definem isso sozinhos — **não force um valor lá**. |
| `LOG_LEVEL` | Não | `info` | Nível de log (`debug`, `info`, `warn`, `error`). |
| `NODE_ENV` | Não | — | Use `production` em produção (logs mais enxutos). |

---

## Usando na mesa

Depois que o site está no ar (qualquer opção acima), o fluxo para os jogadores é:

1. Todo mundo abre **a mesma URL**.
2. Cada jogador escolhe um **slot** (número) para a sua ficha. Combinem quem fica
   com qual slot para ninguém escrever por cima do outro.
3. Preenche a ficha normalmente. As alterações são **salvas no servidor**, então
   qualquer pessoa que abrir aquele slot vê a versão mais recente.
4. **Senha (opcional):** ao salvar, dá pra definir uma senha para o slot. Depois
   disso, só quem tem a senha consegue **salvar** mudanças naquele slot (ver
   continua liberado). Bom pra evitar edição acidental.
5. **Mestre:** a tela `/portraits` (ex.: `https://sua-url/portraits`) mostra os
   retratos dos personagens — deixe numa segunda tela ou compartilhe na live.

> O campo **"Servidor da API"** nas configurações do site só é usado pela versão
> desktop do app. Para o site no navegador ele é ignorado — a interface sempre fala
> com o servidor que a serviu.

---

## Mantendo seu fork atualizado

Quando este projeto original receber novidades, atualize o seu fork:

- **Pelo site do GitHub:** na página do seu fork, clique em **Sync fork** →
  **Update branch**.
- **Pelo terminal:**

  ```bash
  git remote add upstream https://github.com/DONO-ORIGINAL/ProjetoHexatombe.git
  git fetch upstream
  git merge upstream/main
  git push
  ```

Se você usou a **Opção A**, o host detecta o push e **redeploya sozinho**.

---

## E o GitHub Pages?

**Não dá para hospedar este projeto no GitHub Pages.** O Pages só serve arquivos
estáticos — ele não roda Node.js e não conecta em banco de dados. Como as fichas
são salvas por um servidor Node + MongoDB, um deploy só-estático teria a tela
aparecendo mas **nada carregaria nem salvaria**.

Além disso, este repositório traz apenas o site **já compilado** (`dist/`), sem o
código-fonte do frontend, então também não dá pra reconfigurar a interface para
falar com um backend separado hospedado em outro lugar.

O papel do GitHub aqui é ser o **lugar do seu fork**, de onde os serviços da
**Opção A** puxam o código e fazem o deploy automático.

---

## Problemas comuns (FAQ)

**Abri o site mas as fichas não salvam / ficam "carregando".**
O servidor não está conseguindo falar com o MongoDB. Confira:
- O `MONGODB_URI` está certo (usuário, senha, e `/fichaop` antes do `?`).
- No Atlas, **Network Access** libera `0.0.0.0/0`.
- A senha do usuário do banco não tem caracteres que quebram a URL (`@`, `:`, `/`).
  Se tiver, gere uma senha só com letras e números.
- Teste `SUA-URL/api/healthz` — se responder `{"status":"ok"}`, o servidor está de
  pé e o problema é só a conexão com o banco.

**`npm start` reclama de "Cannot find module 'dotenv'".**
Você esqueceu o `npm install` antes.

**O deploy no Render fica "Deploy failed" ou o serviço reinicia sem parar.**
Quase sempre é `MONGODB_URI` ausente ou inválido. Veja os **Logs** no painel do
Render. Confirme também que o **Start Command** é `npm start`.

**Meus amigos não conseguem abrir `http://meu-ip:8080`.**
Firewall do seu sistema bloqueando a porta, ou vocês não estão na mesma rede.
Para acesso pela internet, use um **túnel** (seção B.5) ou a **Opção A**.

**O link do Render demora muito pra abrir.**
Plano Free "dorme" após inatividade. Abra o link ~1 min antes da sessão, ou use
Railway/Fly/Koyeb, ou um plano pago do próprio Render.

**Dois jogadores usaram o mesmo slot e um sobrescreveu o outro.**
Combinem os números antes e/ou ponham **senha** em cada slot.
