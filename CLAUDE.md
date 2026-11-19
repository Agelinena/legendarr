# Regras para conversas neste repositório

Fale sempre em português.

## Antes

- `git pull`. Se houver mudança local que não é sua, pare e pergunte.
- Este repositório é **público**: nada de domínio, IP, e-mail ou segredo reais em código, docs ou exemplos. Use `example.com`.
- O que é de um servidor específico (rotas do proxy, redes externas, rótulos de backup) não entra aqui: fica num arquivo de ajuste no repositório do servidor, por cima do `docker-compose.yml`.

## Testes com Docker

Um teste não pode deixar nada para trás nem encostar no que está no ar.

- **Nunca num servidor de produção** (os `container_name` seriam os mesmos).
- **Projeto `teste-legendarr`:** `docker compose -p teste-legendarr ...`.
- **Dados em `~/.cache/bergatrix-teste/legendarr`**, pelas variáveis de pasta do `.env` (`APPDATA_PATH`, `CACHE_PATH`...). Nunca na pasta temporária do Claude (`/tmp/claude-*`): o Docker recria a pasta como root a cada boot e o Claude Code para de funcionar.
- **`restart: "no"`** nos serviços do teste (num arquivo de ajuste do teste, fora do git), **porta alta** (`WEB_PORT=18080`) e **segredos gerados na hora**.
- **GPU NVIDIA:** o worker precisa dela. Sem GPU na máquina de teste, teste o resto e diga que o worker não foi testado.

Exemplo:

```bash
mkdir -p ~/.cache/bergatrix-teste/legendarr
MEDIA_PATH=~/.cache/bergatrix-teste/legendarr/midia APPDATA_PATH=~/.cache/bergatrix-teste/legendarr CACHE_PATH=~/.cache/bergatrix-teste/legendarr/cache WEB_PORT=18080 \
  docker compose -p teste-legendarr up -d --build
```

## Depois

1. Limpar o teste:
   ```bash
   docker compose -p teste-legendarr down -v --remove-orphans --rmi local
   # a pasta pode ter arquivos do root: apague por dentro de um container
   docker run --rm -v ~/.cache/bergatrix-teste:/d alpine:3.20 rm -rf /d/legendarr
   docker image prune -f
   ```
   Confira com `docker ps -a` que não sobrou nada `teste-*`.
2. Atualize o README e os docs que a mudança deixou errados, no mesmo commit.
3. Diga o que foi e o que não foi testado.

## Commits

- Só com autorização de quem pediu a mudança.
- E-mail pessoal, nunca o do trabalho: confira `git config user.email` contra `git log -3 --format=%ae`.
- Nunca `.env`, chaves (`*.jks`, `key.properties`) nem dados de volume.
