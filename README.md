# minhasContas

Ponto de entrada para trabalhar nos projetos do minhasContas juntos: subir o ambiente, rodar os testes e fazer o build com um comando só.

Este repositório guarda **só scripts e documentação**. O código mora nos dois projetos, que são clones **irmãos** desta pasta, cada um com o seu git e o seu remoto:

```
WORK/
├── minhasContas/    <- este repositório (scripts)
├── minhasContas-db/     <- API (Spring Boot)   github.com/RafaFBorges/minhasContas-DB
└── minhasContas-Web/    <- Web (Next.js)       github.com/RafaFBorges/minhasContas-Web
```

Commit, push, branch e histórico continuam dentro de cada projeto (`cd ../minhasContas-Web && git log`). Aqui nada é submódulo, então não existe ponteiro para atualizar depois de cada mudança.

## Primeira vez (inclusive em outra máquina)

Precisa de: git, JDK 17, Maven, Node 18+, Docker (com compose), curl e openssl.

```bash
git clone <url deste repositório> minhasContas
cd minhasContas
./dev setup      # clona os projetos que faltarem, cria o .env da API e instala as dependências do Web
./dev up         # banco (Docker), API e Web
```

O `setup` não sobrescreve nada que já existe, então pode ser rodado de novo. O `.env` da API é gerado com segredos novos (o login do Admin local fica em `ADMIN_USER` e `ADMIN_PASSWORD` dentro dele). Segredos nunca entram no git.

O que o `setup` **não** faz: trocar de branch. Os clones vêm na `main`; se você trabalha numa branch de feature, dê `git checkout` nela dentro do projeto.

## Comandos

| Comando | O que faz |
|---|---|
| `./dev setup` | clona o que faltar, cria o `.env` da API, instala dependências |
| `./dev doctor` | confere ferramentas, projetos, `.env` e portas |
| `./dev up [db] [api] [web]` | sobe os serviços (sem argumento, todos) |
| `./dev down [--keep-db]` | para tudo |
| `./dev status` | estado de db, api e web |
| `./dev logs api\|web\|db` | acompanha o log |
| `./dev reset-db` | apaga os dados do banco local (pede confirmação) |
| `./dev test [tipo] [alvo]` | roda testes (veja abaixo) |
| `./dev lint` | lint e typecheck do Web |
| `./dev build [api\|web\|all]` | build de cada projeto |
| `./dev git [status\|pull]` | branch, alterações e ahead/behind dos três repositórios |

Endereços locais: Web em http://localhost:3000, API em http://localhost:8080, Postgres na porta 5433.

### Como sobe

- **Banco:** o `docker-compose.yml` da API é a única definição do Postgres. O script usa só o serviço `db` dele, com o mesmo volume de sempre.
- **API:** `mvn spring-boot:run` na pasta da API (lê o `.env` dela).
- **Web:** `npm run dev` com `API_URL=http://localhost:8080/`.
- API e Web rodam em segundo plano; PIDs e logs ficam em `.run/` (ignorado pelo git).

## Testes por tipo

```bash
./dev test                     # tudo o que existe (unit + integration + static), sem e2e
./dev test unit api            # só unitários da API
./dev test integration api     # só integração da API
./dev test static web          # lint + typecheck do Web
```

| Tipo | Onde | O que é | Precisa de Docker |
|---|---|---|---|
| `unit` | API | classes sem Spring (Mockito e lógica pura). Rápidas | não |
| `integration` | API | `@SpringBootTest` com Testcontainers (Postgres 15) | sim |
| `static` | Web | `eslint` e `tsc --noEmit` | não |
| `e2e` | Web + API | **ainda não existe** (backlog: Playwright) | sim |

A separação entre `unit` e `integration` é automática: uma classe de teste é de integração se tiver `@SpringBootTest`, `@Testcontainers`, `@DataJpaTest` ou `@WebMvcTest`; as demais são unitárias. Se criar um teste novo, basta anotá-lo do jeito certo.

O resumo ao final vem dos relatórios do surefire (e não do código de saída do Maven), porque o build deste projeto pode "passar" sem executar nada quando há erro de compilação nos testes. Se nenhum teste rodar, o comando falha.

## Variáveis

Tudo tem padrão; troque por variável de ambiente se precisar:

| Variável | Padrão |
|---|---|
| `API_DIR` | `<pasta acima>/minhasContas-db` |
| `WEB_DIR` | `<pasta acima>/minhasContas-Web` |
| `API_REPO`, `WEB_REPO` | URLs SSH dos repositórios no GitHub |
| `API_PORT`, `WEB_PORT` | `8080`, `3000` |

## Adicionar um serviço novo (ex.: o coletor de logs)

Seguir o padrão dos dois existentes no script `dev`: variáveis `*_REPO`/`*_DIR`, uma linha no `setup`, uma função `up_*` e as entradas de `test`/`build`.
