# 🐍 MADSON COBRA CLIPPER

**Captura de lives por link → IA encontra os melhores momentos → gera os clipes → publica.**

Cole o link de uma transmissão ao vivo (YouTube Live, Twitch ou Kick) e o sistema faz todo o resto:
valida o link, confirma que está AO VIVO, captura a transmissão, analisa áudio/vídeo em tempo real,
identifica os melhores momentos com pontuação de qualidade, renderiza os clipes com legendas,
títulos e descrições, e deixa tudo pronto para publicação.

---

##✨ O que foi implementado

| Módulo | Status |
|---|---|
| Tela inicial **COLE O LINK DA SUA LIVE** com detecção automática de plataforma | ✅ |
| Painel de monitoramento por transmissão (streamer, miniatura, tempo, status, momentos, clipes, progresso) | ✅ |
| 8 indicadores de status (Aguardando → Validando → Capturando → Analisando → Identificando → Gerando → Finalizado / Erro) | ✅ |
| Adaptadores independentes **YouTube / Twitch / Kick** (yt-dlp) | ✅ |
| Validação real de link + verificação de AO VIVO + identificação do canal | ✅ |
| Captura real de lives com yt-dlp (`--live-from-start` no YouTube) | ✅ |
| Análise automática de momentos (cortes de cena + picos de energia de áudio) via ffmpeg | ✅ |
| Pontuação de qualidade por momento (0–100) | ✅ |
| Geração de clipes em MP4 + miniatura + legenda `.srt` + pacote de publicação `.txt` | ✅ |
| Processamento durante a transmissão (momentos e clipes surgem ao vivo) | ✅ |
| Finalização automática quando a live termina ( ou ao clicar em **Encerrar live** ) | ✅ |
| Gerenciamento de várias lives simultâneas com fila (capacidade configurável) | ✅ |
| Pausar / Retomar / Cancelar / Encerrar cada análise | ✅ |
| Upload de gravação local (quando a plataforma bloqueia a captura) | ✅ |
| Modo demonstração (transmissão simulada processada de verdade pelo pipeline) | ✅ |
| Publicação preparada (simulada) para YouTube / Twitch / Kick + download dos arquivos | ✅ |
| Tempo real via WebSocket (progresso, momentos e clipes sem recarregar) | ✅ |

---

## 📋 Requisitos

- **Node.js 18+** (testado com 20)
- **ffmpeg / ffprobe**
- **yt-dlp** (captura e validação das plataformas)
- **Python 3** (apenas para o *modo demonstração*)

Instalação rápida (Debian/Ubuntu):

```bash
sudo apt-get install -y ffmpeg python3
pip3 install --user yt-dlp
```

## 🚀 Como rodar

```bash
cd madson-cobra-clipper
npm install
npm start
```

Abra **http://localhost:3000**

Variáveis de ambiente (opcionais):

| Variável | Padrão | Descrição |
|---|---|---|
| `PORT` | `3000` | Porta do servidor |
| `CAPACITY` | `3` | Análises simultâneas (fila automática acima disso) |
| `YTDLP_BIN` | auto | Caminho do yt-dlp |
| `FFMPEG_BIN` / `FFPROBE_BIN` | `ffmpeg` / `ffprobe` | Binários do ffmpeg |

---

## ▶️ Como usar

1. **Cole o link da live** — o sistema identifica a plataforma sozinho (YouTube, Twitch ou Kick).
2. Clique em **INICIAR ANÁLISE DA LIVE**.
3. Acompanhe o painel: status, tempo analisado, momentos encontrados e clipes gerados em tempo real.
4. Quando a transmissão terminar (ou você clicar em **Encerrar live**), a IA finaliza: seleciona os
   melhores momentos, remove trechos inadequados, renderiza os clipes, gera legendas/títulos/descrições
   e publica conforme suas configurações.
5. Baixe os clipes: botão **⬇ Baixar MP4** em cada clipe, ou **⬇ Baixar tudo** no card para
   levar um ZIP com todos os clipes (MP4 + legenda `.srt` + pacote `.txt` + miniatura).

Se a plataforma **não permitir a captura direta** (login, membros, privacidade, geo-bloqueio), o sistema
exibe o motivo claro e oferece **enviar uma gravação local** — que é analisada com o mesmo pipeline.

## 🧪 Modo demonstração

Sem um link ao vivo à mão? Clique em **▶ Ver demonstração**. O sistema gera uma transmissão simulada
(vídeo + áudio sintetizados em `tools/make_demo_stream.py`) e a processa **de verdade**: análise,
cortes, miniaturas, legendas e pacote — tudo real, só o conteúdo da live que é sintético.

---

## 🏗️ Arquitetura

```
server.js                  HTTP + WebSocket + upload (Express, ws, multer)
src/
  config.js                configurações e diretórios
  status.js                status, labels e meta das plataformas
  jobs.js                  JobManager: fila, capacidade, broadcast em tempo real
  pipeline.js              máquina de estados: validar → capturar → analisar → gerar → finalizar
  adapters/
    youtube.js             adaptador YouTube (watch/live/shorts/youtu.be)
    twitch.js              adaptador Twitch
    kick.js                adaptador Kick
    probe.js               sonda yt-dlp + mensagens de erro amigáveis
  capture.js               captura real (yt-dlp) e geração da transmissão demo
  analysis.js              análise ffmpeg: cenas + silêncios + picos de áudio
  render.js                corte dos clipes, miniaturas, legendas, títulos/descrições
  publish.js               publicação preparada (simulada)
tools/
  make_demo_stream.py      sintetizador da transmissão de demonstração
public/
  index.html · styles.css · app.js   dashboard (tempo real via WebSocket)
media/                     capturas, clipes, miniaturas e uploads (servidos em /media)
data/                      sidecars do modo demo
```

### Fluxo

```
COLAR LINK → INICIAR ANÁLISE → IA ENCONTRA OS MELHORES MOMENTOS → GERA OS CORTES → PUBLICA
```

## 🌐 Publicando (para ser usado de verdade)

O site já funciona **agora** no preview da Arena:

```
https://3000-{sandboxId}.e2b.app
```

Esse preview é ideal para testar, mas vive enquanto o sandbox ficar ligado. Para uso
contínuo (link fixo, sempre no ar), publique em uma nuvem. As opções abaixo já estão
com arquivos de configuração prontos neste repositório.

### Opção A — Render (mais fácil, plano gratuito)

1. Crie conta em `render.com` (pode ser com GitHub).
2. Suba este código para um repositório (GitHub/GitLab).
3. No Render: **New → Blueprint → selecione o repositório**.
4. Ele lê o `render.yaml` e cria o serviço com Docker + volume para os clipes.

Plano gratuito "dorme" após inatividade (~30s para acordar) — ótimo para começar.

### Opção B — Fly.io (gratuito com conta ativa)

```bash
fly apps create madson-cobra-clipper
fly volumes create media --region gru --size 2
fly deploy
```

### Opção C — VPS própria (Docker Compose)

```bash
git clone <seu-repo> && cd madson-cobra-clipper
docker compose up -d --build
```

Pronto: `http://IP-DA-VPS:3000` (use um domínio + HTTPS com Caddy/Traefik se quiser).

### Opção D — Railway

Crie o serviço a partir do repositório (ele detecta o `Dockerfile`); defina a
variável `PORT=3000`. Para não perder clipes, anexe um volume em `/app/media`.

### Observações importantes

- **Volume**: sempre monte `/app/media` (e `/app/data`) em disco persistente —
  sem isso, capturas e clipes somem a cada deploy/reinício.
- **Recursos**: análises usam ffmpeg/yt-dlp; 1 vCPU + 1 GB de RAM aguenta bem
  até 3 análises simultâneas (`CAPACITY=3`).
- **Domínio próprio**: todas as plataformas acima permitem adicionar domínio
  e HTTPS gerado automaticamente.
- **Atualizar**: após mudanças, refaça o deploy (git push + redeploy, ou
  `docker compose up -d --build`).

## ⚖️ Compatibilidade e legalidade

- Captura apenas de transmissões **públicas e autorizadas**, usando APIs/ferramentas oficiais
  e fontes disponibilizadas legalmente pela plataforma.
- **Não** contorna DRM, autenticação, restrições geográficas ou mecanismos de proteção.
- Quando a plataforma bloqueia, o motivo é informado e a alternativa é o **upload de gravação local**.
- A publicação é **simulada** nesta versão (as plataformas podem exigir OAuth/API oficial para upload
  automatizado); todos os arquivos ficam prontos para publicação manual.

## 🔌 API (resumo)

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/state` | estado de todas as análises |
| `POST` | `/api/jobs` | cria análise por link (`{url, maxClips, autoPublish, targets}`) ou demo (`{demo:true}`) |
| `POST` | `/api/jobs/local` | cria análise a partir de arquivo (`multipart: video`) |
| `POST` | `/api/jobs/:id/pause` · `/resume` · `/cancel` · `/end` | controles |
| `POST` | `/api/jobs/:id/publish` | publica os clipes |
| `GET` | `/api/jobs/:id/download` | ZIP com todos os clipes (MP4, legenda, pacote, miniatura) |
| `WS` | `/ws` | broadcast de estado em tempo real |
