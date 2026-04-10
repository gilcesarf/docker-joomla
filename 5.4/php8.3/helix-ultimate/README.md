# Joomla Helix Ultimate Quickstart (PHP 8.3 + Apache)

Imagem Docker baseada em `php:8.3-apache` com extensões PHP usuais do Joomla, empacotando o **quickstart** do template [Helix Ultimate](https://www.joomshaper.com/documentation/helix-framework/helix-ultimate-quckstart-installation) em vez do pacote “vanilla” do núcleo Joomla.

## Pré-requisitos

- Docker e Docker Compose (plugin `docker compose`)
- Ficheiro `.env` nesta pasta (criar a partir do modelo): `cp .env.example .env` e editar; `HELIX_QUICKSTART_URL` é obrigatório para o build

## Variáveis de build da imagem

| Variável | Obrigatório | Descrição |
|----------|-------------|-----------|
| `HELIX_QUICKSTART_URL` | Sim | URL HTTPS que devolve o ficheiro `.zip` do quickstart (ex.: link de download JoomShaper). |
| `HELIX_QUICKSTART_SHA512` | Não | Soma SHA-512 do zip; se preenchida, o build valida o ficheiro após o download. |

## Criar a imagem

### Com Docker Compose (recomendado)

Na pasta `5.4/php8.3/helix-ultimate/`:

```bash
docker compose build
```

O Compose lê o `.env` e passa os *build args* ao `Dockerfile`. Para forçar reconstrução completa:

```bash
docker compose build --no-cache
```

### Com `docker build` manual

Executar a partir desta pasta (onde estão o `Dockerfile`, `docker-entrypoint.sh` e `makedb.php`):

```bash
docker build \
  --build-arg HELIX_QUICKSTART_URL="https://www.joomshaper.com/downloads/template/helixultimate/2-quickstart-pack-joomla-5" \
  -t joomla-helix-ultimate:5.4-php8.3 \
  .
```

Opcionalmente acrescentar `--build-arg HELIX_QUICKSTART_SHA512=<hash>`.

## Uso local com Docker Compose

1. Se ainda não existir `.env`: `cp .env.example .env` e ajustar (em especial `MYSQL_ROOT_PASSWORD`; URL do quickstart, portas, `JOOMLA_DB_NAME`, `JOOMLA_HOST_APP_DIR`, `MYSQL_HOST_DATA_DIR` se necessário).
2. Subir os serviços:

```bash
docker compose up -d
```

3. Abrir o site no browser: **http://localhost:8080** (ou a porta definida em `JOOMLA_HTTP_PORT`).
4. Seguir o assistente de instalação do Joomla (passos descritos na documentação JoomShaper em *Installing procedure*). O contentor cria a base MySQL se ainda não existir; credenciais vêm de `JOOMLA_DB_*` e `MYSQL_ROOT_PASSWORD` no `.env`.

### Serviços e volumes

| Serviço | Descrição |
|---------|-----------|
| `joomla-mysql` | MySQL 8.4; dados persistentes na pasta do host definida por `MYSQL_HOST_DATA_DIR` (predefinição `./database`). |
| `joomla-app` | Apache + PHP; ficheiros do site na pasta do host `JOOMLA_HOST_APP_DIR` (predefinição `./app`), montada em `/var/www/html`. |

Na primeira execução, com a pasta da app vazia, o *entrypoint* copia o quickstart de `/usr/src/joomla` para `/var/www/html`.

O `.dockerignore` ignora por defeito `app/` e `database/` para não inflar o contexto de build; se mudares as pastas no `.env`, acrescenta os mesmos caminhos (relativos) ao `.dockerignore`.

### Parar e remover

```bash
docker compose down
```

Para apagar também os volumes nomeados (não remove por defeito as pastas `./app` e `./database` no disco):

```bash
docker compose down -v
```

## Ficheiros relevantes

- `Dockerfile` — construção da imagem e extração do quickstart
- `docker-compose.yml` — MySQL + aplicação Joomla
- `.dockerignore` — exclui do contexto de build `app/`, `database/`, `.env`, `.git`
- `.env.example` — modelo de variáveis (versionado); copiar para `.env` local
- `.env` — criado por ti a partir do exemplo; ignorado pelo Git e usado pelo Compose

## Referências

- [Quickstart Installation | Helix Ultimate (JoomShaper)](https://www.joomshaper.com/documentation/helix-framework/helix-ultimate-quckstart-installation)
- [Requisitos técnicos Joomla](https://downloads.joomla.org/technical-requirements)
