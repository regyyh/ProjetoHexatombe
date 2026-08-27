# Ficha OP — Hexatombe

> Este é um conteúdo não oficial, publicado sob a
> [Licença da Comunidade de Ordem Paranormal](https://ordemparanormal.com.br/licenca).
> Projeto de fã, sem fins lucrativos. **Não oficial. Não canônico.**

Ficha de personagem online para as suas mesas de **Ordem Paranormal**.

Cada ficha fica guardada num **slot numerado** e pode ter uma **senha**. Todo mundo
que abrir o mesmo endereço enxerga os mesmos slots — é assim que a mesa inteira
compartilha as fichas. Também tem uma página `/portraits` com os retratos dos
personagens, ótima pra deixar numa segunda tela ou na live.

> Este repositório já vem com o site **compilado** (pasta `dist/`). Você não precisa
> "buildar" nada — só rodar o servidor e apontar pra um banco de dados MongoDB.

> **Projeto original:** [@regyyh](https://github.com/regyyh) (Discord: `regyyh`) —
> <https://github.com/regyyh/ProjetoHexatombe>

---

## Como é o app

| Tela inicial (seleção de fichas) | No celular |
|---|---|
| ![Tela inicial](docs/tela-inicial.png) | ![No celular](docs/celular.png) |

Cada linha é um **slot** (`ficha1`, `ficha2`, …) com o nome do personagem, a data
da última alteração e um cadeado quando a ficha tem **senha**. Os botões de baixo
criam uma **Nova Ficha** ou abrem a página de **Retratos**.

### Dentro da ficha

A ficha é dividida em abas. À esquerda ficam sempre o retrato, atributos, **PV**
(vida), **PD** (pontos de determinação/esforço), sanidade, defesa e resistências.

| Combate | Perícias |
|---|---|
| ![Aba Combate](docs/ficha-combate.png) | ![Aba Perícias](docs/ficha-pericias.png) |

| Rituais | Inventário |
|---|---|
| ![Aba Rituais](docs/ficha-rituais.png) | ![Aba Inventário](docs/ficha-inventario.png) |

- **Combate:** armas e ataques (dano, crítico, alcance, tipo de ação).
- **Perícias:** treino + bônus de cada perícia, com o total calculado.
- **Poderes / Rituais:** habilidades e rituais, com círculo, elemento e custo em PE.
- **Inventário:** itens, carga atual, patente e categorias.
- **Descrição:** história e anotações livres do personagem.

Tudo é salvo automaticamente no servidor (o indicador **"Salvando…"** aparece no
topo). No celular o mesmo layout vira uma coluna só, com um seletor de aba no topo:

| Topo da ficha | Aba Combate | Aba Rituais |
|---|---|---|
| ![Ficha no celular](docs/ficha-mobile-topo.png) | ![Combate no celular](docs/ficha-mobile-combate.png) | ![Rituais no celular](docs/ficha-mobile-rituais.png) |

### Página de Retratos (`/portraits`) — para lives e gravações

![Página de retratos](docs/retratos.png)

Mostra os retratos dos personagens em molduras hexagonais com a vida atual. Feita
para entrar como **fonte de navegador no OBS** (fundo branco, sem interface). Veja
a seção [Uso em lives / gravações](#uso-em-lives--gravações).

---

## Índice

1. [Como funciona (leia antes)](#como-funciona-leia-antes)
2. [Passo 0 — Banco de dados grátis (MongoDB Atlas)](#passo-0--banco-de-dados-grátis-mongodb-atlas)
3. [Opção A — Hospedar de graça a partir do seu fork (recomendado)](#opção-a--hospedar-de-graça-a-partir-do-seu-fork-recomendado)
4. [Opção B — Rodar no seu PC e jogar com os amigos](#opção-b--rodar-no-seu-pc-e-jogar-com-os-amigos)
5. [Opção C — Docker (self-host completo, banco incluso)](#opção-c--docker-self-host-completo-banco-incluso)
6. [Variáveis de ambiente](#variáveis-de-ambiente)
7. [Usando na mesa](#usando-na-mesa)
8. [Uso em lives / gravações](#uso-em-lives--gravações)
9. [Mantendo seu fork atualizado](#mantendo-seu-fork-atualizado)
10. [E o GitHub Pages?](#e-o-github-pages)
11. [Problemas comuns (FAQ)](#problemas-comuns-faq)
12. [Licença](#licença)
13. [Créditos](#créditos)

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

## Uso em lives / gravações

O projeto tem uma página feita pra aparecer na transmissão: **`/portraits`**
(ex.: `https://sua-url/portraits`). Ela mostra só os retratos dos personagens e a
vida atual, com **fundo branco** e sem nenhuma interface — pensada para virar
_overlay_ no OBS.

### Colocando no OBS Studio

1. Deixe a sua ficha no ar por qualquer uma das opções deste tutorial.
2. No OBS: **Fontes** → **+** → **Navegador**.
3. Em **URL**, coloque `https://sua-url/portraits`.
4. Largura/altura a gosto (ex.: `1920 × 1080`).
5. Para deixar o fundo **transparente** em vez de branco, cole isto no campo
   **CSS personalizado** da fonte:

   ```css
   body { background: transparent !important; margin: 0; overflow: hidden; }
   ```

6. Os retratos e a vida se atualizam sozinhos conforme a mesa edita as fichas — não
   precisa ficar recarregando a fonte.

### Se você postar a gravação ou fizer live

Se for divulgar o vídeo, o stream ou uma versão modificada, dá uma força pro autor:

- **Marque [@regyyh](https://github.com/regyyh)** (Discord: `regyyh`).
- **Deixe o link do repositório original** na descrição:
  <https://github.com/regyyh/ProjetoHexatombe>
- Deixe visível o **selo "Não Oficial / Não Canônico"** (em [`licenca/`](licenca/))
  ou o aviso da [Licença da Comunidade](#licença) — pode ser como imagem fixa no
  layout do OBS, num canto do overlay.

O projeto é gratuito; esse crédito é o que mantém a coisa viva. 🙏

> **Privacidade:** a página `/portraits` e as fichas ficam acessíveis para qualquer
> pessoa que tiver o link. Se não quiser que a plateia mexa nas fichas durante a
> live, ponha **senha** em cada slot (ver [Usando na mesa](#usando-na-mesa)) e evite
> mostrar a URL na tela.

---

## Mantendo seu fork atualizado

Quando este projeto original receber novidades, atualize o seu fork:

- **Pelo site do GitHub:** na página do seu fork, clique em **Sync fork** →
  **Update branch**.
- **Pelo terminal:**

  ```bash
  git remote add upstream https://github.com/regyyh/ProjetoHexatombe.git
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

---

## Licença

Este repositório tem **duas camadas**:

| O quê | Licença |
|---|---|
| **Código** (servidor, build, scripts) | [MIT](LICENSE) — © 2026 [@regyyh](https://github.com/regyyh) |
| **Ordem Paranormal** (universo, nomes e símbolos dos elementos, estética, termos de sistema) + **todas as imagens e fontes** do site | [Licença da Comunidade de Ordem Paranormal](https://ordemparanormal.com.br/licenca) — obra de Rafael "Cellbit" Lange / e artistas da comunidade (ver tabela abaixo) |

A licença MIT vale **só para o código**. As artes e o material de Ordem Paranormal
**não** são MIT.

**O que a Licença da Comunidade exige de qualquer fork/publicação:**

- Mostrar o **selo "Não Oficial / Não Canônico"** (mín. 10% da largura, 100% de
  opacidade) **ou** o texto
  *"Este é um conteúdo não oficial, publicado sob a Licença da Comunidade de Ordem Paranormal"*
  logo abaixo do título. Selo em [`licenca/`](licenca/).
- Uso **não comercial** (não cobrar pelo acesso).
- Não sugerir que é oficial ou aprovado pela Jambô / Cellbit.
- **Não reproduzir imagens dos livros oficiais.**

> ⚠️ **Para quem forka:** o `dist/public/` embute símbolos oficiais dos elementos,
> o diagrama de atributos e molduras dos *Arquivos Secretos*. Isso esbarra no
> "não reproduzir imagens oficiais". O mais seguro é **trocar essas artes pelas
> suas** e manter o selo visível na interface (o `dist/` aqui é build fechado — dá
> pra sobrepor o selo pelo OBS, mas o certo é rebuildar a partir do fonte com o
> selo dentro do app).

---

## Créditos

**Código / projeto:** [@regyyh](https://github.com/regyyh) (Discord: `regyyh`) —
repositório original: <https://github.com/regyyh/ProjetoHexatombe>

**Ordem Paranormal** — universo, os 5 elementos e seus símbolos, estética e termos
de sistema (NEX, PV, PD, PE, Perícias, Rituais, DT…): criação de **Rafael "Cellbit"
Lange**, © 2022, todos os direitos reservados. RPG publicado pela **Jambô Editora**.
Usado sob a [Licença da Comunidade](https://ordemparanormal.com.br/licenca).

**Equipes de arte oficiais** (de onde vêm os símbolos, o diagrama de atributos e as molduras):

- *Ordem Paranormal RPG — Livro de Regras*: arte interna de Akila Gabriel, Danilo
  "Orenjiro" Murakami, Dan Ramos, Kael e Jottah Designer; logo, projeto gráfico e
  diagramação de **Dan Ramos**.
- *Arquivos Secretos* (suplementos oficiais — origem das **molduras**): direção de
  arte de **Dan Ramos**; ilustrações de Dan Ramos, Gabriel Feitosa, Heitor Aquino,
  Janio Garcia, Lara "Rarinha", Marina "Nina" Theisen, Vic Terra e Vinícius
  "Oceanin" Rafael.

### Tabela de imagens do site

> Boa parte das artes foi **coletada do Twitter/X pela comunidade** e a procedência
> individual não foi registrada na época. As linhas marcadas `⚠️ a preencher`
> precisam do link/@ de quem publicou — se você reconhece ou é o autor, abra uma
> [issue](https://github.com/regyyh/ProjetoHexatombe/issues) para receber crédito
> ou pedir remoção.

| Arquivo | O que é | Fonte | Autor / crédito |
|---|---|---|---|
| `dist/public/sangue.jpg` | Símbolo do elemento Sangue | Twitter/X (comunidade) | ⚠️ a preencher — símbolo © Rafael Lange |
| `dist/public/morte.jpg` | Símbolo do elemento Morte | Twitter/X (comunidade) | ⚠️ a preencher — símbolo © Rafael Lange |
| `dist/public/conhecimento.jpg` | Símbolo do elemento Conhecimento | Twitter/X (comunidade) | ⚠️ a preencher — símbolo © Rafael Lange |
| `dist/public/energia.jpg` | Símbolo do elemento Energia | Twitter/X (comunidade) | ⚠️ a preencher — símbolo © Rafael Lange |
| `dist/public/medo.jpg` | Símbolo do elemento Medo | Twitter/X (comunidade) | ⚠️ a preencher — símbolo © Rafael Lange |
| `dist/public/atributos.png` | Diagrama hexagonal "ATRIBUTOS" | Identidade visual oficial do RPG | Projeto gráfico: Dan Ramos / Jambô — © Rafael Lange |
| `dist/public/favicon.jpg`, `favicon.png` | Ícone da aba (símbolo de Sangue) | Oficial | © Rafael Lange |
| `dist/public/opengraph.jpg` | Miniatura de link (Open Graph) | (https://fonts.google.com/) | Fonts-google|
| `dist/public/assets/moldura-sangue.png` | Moldura hexagonal (Sangue) | *Arquivos Secretos* (extra oficial) — Jambô Editora | Equipe de arte de *Arquivos Secretos* (dir.: Dan Ramos) |
| `dist/public/assets/moldura-morte.png` | Moldura hexagonal (Morte) | *Arquivos Secretos* — Jambô Editora | Equipe de arte de *Arquivos Secretos* |
| `dist/public/assets/moldura-conhecimento.png` | Moldura hexagonal (Conhecimento) | *Arquivos Secretos* — Jambô Editora | Equipe de arte de *Arquivos Secretos* |
| `dist/public/assets/moldura-energia.png` | Moldura hexagonal (Energia) | *Arquivos Secretos* — Jambô Editora | Equipe de arte de *Arquivos Secretos* |
| `dist/public/assets/fundope.png` | Textura de fumaça (fundo) | ⚠️ a preencher | ⚠️ a preencher |
| `dist/public/assets/portrait-bg.png` | Fundo esfumaçado dos retratos | ⚠️ a preencher | ⚠️ a preencher |
| Retratos em `/portraits` (vêm do banco) | Arte de cada personagem | Enviados pelo mestre / jogadores | Crédito é de quem sobe a imagem |
| `licenca/selo-op-comunidade-*.png` | Selo da Licença da Comunidade | Jambô / Cellbit ([ordemparanormal.com.br/licenca](https://ordemparanormal.com.br/licenca)) | © Rafael Lange |
| `docs/*.png` | Capturas de tela deste README | Interface do próprio app | Este repositório |

**Fontes tipográficas:** Utilizadas fonts pela https://fonts.google.com/).


Se for usar em live, gravação ou fork público: mantenha o crédito ao
[@regyyh](https://github.com/regyyh), o link do repositório original e o
selo/aviso da Licença da Comunidade.

![Selo Não Oficial da Licença da Comunidade de Ordem Paranormal](licenca/selo-op-comunidade-preto.png)
