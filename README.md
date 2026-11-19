# Legendarr

[![CI](https://github.com/Agelinena/legendarr/actions/workflows/ci.yml/badge.svg)](https://github.com/Agelinena/legendarr/actions/workflows/ci.yml)

Legendas em português para a biblioteca do Radarr e do Sonarr, sem serviço de fora: o que o Bazarr não acha, a IA local (Ollama) traduz. Licença: [AGPL-3.0](LICENSE).

## O que faz

| Serviço | O quê |
|---|---|
| `worker` | Varre `filmes/` e `series/`; pede a legenda ao Bazarr e espera a sincronização dele; se não houver, traduz pelo Ollama a legenda em inglês (embutida ou externa); refina a sincronia por conteúdo (embeddings) e audita legendas incompletas. Também confere se o áudio do idioma original existe (senão apaga e pede de novo ao Radarr/Sonarr) e remove downloads travados. |
| `web` | Painel: legendas por título, traduções, estatísticas, logs e ações manuais (traduzir, buscar no Bazarr, alinhar com ALASS). **Sem login.** |
| `optimizer` | Converte para HEVC os vídeos que ainda não são (NVENC 10 bits, depois 8 bits, depois x265 por software), para ocupar menos espaço. Áudio e legendas são copiados. |

As legendas geradas por IA saem com um rótulo na faixa (`AI_SUBTITLE_LABEL`, ex.: "Português - AI"), para o Jellyfin mostrar a diferença.

## Precisa de

- **Radarr, Sonarr e Bazarr** já funcionando, com as chaves de API.
- **Ollama** com os modelos: `ollama pull translategemma:4b` (tradução) e `ollama pull bge-m3` (refino de sincronia).
- **GPU NVIDIA** com o nvidia-container-toolkit (worker e optimizer).
- A **mesma pasta de mídia** do Radarr e do Sonarr, com `filmes/` e `series/`.

## Subir

```bash
git clone https://github.com/Agelinena/legendarr.git && cd legendarr
cp .env.example .env        # MEDIA_PATH, chaves do Bazarr, Radarr e Sonarr, endereço do Ollama
docker compose up -d --build
docker compose logs -f worker
```

**O painel não tem login.** Ele pode disparar traduções e apagar legendas: não o exponha direto. Ponha na frente um proxy com autenticação (forward-auth do Authentik, Authelia, oauth2-proxy) ou deixe a porta só na rede local.

### Com o Bazarr, o Radarr, o Sonarr e o Ollama em containers

Os endereços padrão (`http://bazarr:6767`, `http://radarr:7878`, `http://sonarr:8989`, `http://ollama:11434`) funcionam quando o Legendarr está na mesma rede Docker que eles. Um arquivo por cima do compose resolve:

```yaml
# legendarr.local.yml — use com COMPOSE_FILE=docker-compose.yml:legendarr.local.yml no .env
services:
  web:
    networks: [legendarr, midia]
  worker:
    networks: [legendarr, midia]
networks:
  midia:
    external: true
    name: nome-da-rede-dos-arr   # docker network ls
```

## Atualizar

```bash
git pull && docker compose up -d --build
```

## Testes

```bash
cd worker && pip install -r requirements.txt pytest && python -m pytest tests
cd web && pip install -r requirements.txt pytest && python -m pytest tests
```

## Limitações conhecidas

- Os três containers rodam como root (o `PUID`/`PGID` ainda não é aplicado).
- O optimizer guarda `requires_redownload.txt` dentro do container, fora de volume.
