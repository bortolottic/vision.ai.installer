# Vision.AI — Guia de Instalação

| Data | Autor | Descrição |
| ---- | ----- | --------- |
| 20/10/2026 | CVB | Criação do projeto de instalação (deploy Swarm, 14 serviços) |

---

## O que é o Vision.AI?

O **Vision.AI** é uma plataforma de **visão computacional para câmeras**: captura o
vídeo das câmeras, roda modelos de detecção sobre os quadros, aplica regras e
dispara notificações (central na tela e webhook assinado). Roda de forma
containerizada com Docker. Este repositório é o **instalador oficial**, que
automatiza o download e a implantação.

> **Para quem não é técnico:** pense neste instalador como um assistente que faz
> tudo sozinho. Você segue os passos abaixo, cola um único comando no terminal e
> ele baixa a versão correta, descompacta e coloca tudo no ar.

### 📖 Já instalou? A documentação de uso fica aqui

**[Manual do Vision.AI](https://bortolottic.github.io/vision.ai.installer/)** — em
**português, inglês e espanhol**:

- **Manual das telas** — o que cada tela faz e as decisões que ela pede.
- **Referência de operação** — serviços, portas, backup e solução de problemas.

Este README continua sendo o guia de **instalação**. A mesma documentação é
servida pela sua instalação em `/docs/` — útil quando a máquina não tem internet.

---

## Antes de instalar

### Requisitos da máquina

O Vision.AI sobe **quinze serviços** em conjunto (a plataforma, os bancos, as
filas, o armazém de objetos, o gateway de vídeo e os workers de captura,
inferência e regras). Para funcionar com folga:

| Recurso | Mínimo | Recomendado |
| ------- | ------ | ----------- |
| Memória RAM | 8 GB | 16 GB |
| Processador | 4 núcleos | 8 núcleos |
| Espaço em disco | 40 GB livres | 100 GB livres |

> **GPU (opcional, mas recomendada para produção):** a inferência e o treino
> rodam em CPU por padrão — funcionam, mas devagar. Para detecção em tempo real
> com várias câmeras, use um nó com placa NVIDIA e instale com `--gpu` (ver
> [Rodar na GPU](#rodar-na-gpu)). Exige o **NVIDIA Container Toolkit**.

### Portas que precisam estar livres

| Porta | Para que serve |
| ----- | -------------- |
| **9080** | Endereço da plataforma no navegador |
| 8189/UDP | WebRTC — vídeo ao vivo chega ao navegador por aqui |
| 3433 | Banco de dados TimescaleDB (publicado só para inspeção; pode ser fechado) |
| 443 | HTTPS, quando a instalação é feita com `--ssl` |

Para verificar se uma porta está ocupada (Linux, macOS ou WSL2):

```bash
sudo lsof -i :9080
```

Se o comando não devolver nada, a porta está livre.

---

## Escolha o seu sistema operacional

- [Instalação no Linux](#instalação-no-linux)
- [Instalação no Windows](#instalação-no-windows)
- [Instalação no macOS](#instalação-no-macos)

---

## Instalação no Linux

### Passo 1 — Abra o terminal

- **Ubuntu / Debian / Mint:** `Ctrl + Alt + T`
- **Fedora / RHEL:** botão direito na área de trabalho → "Abrir terminal"
- **Qualquer distribuição:** procure por "Terminal" no menu de aplicativos

### Passo 2 — Instale o Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io curl
sudo systemctl enable docker --now
sudo usermod -aG docker $USER
```

> Fedora/RHEL: troque `apt` por `dnf`.

Feche e reabra o terminal (para aplicar o grupo docker) e confira:

```bash
docker --version
```

### Passo 3 — Ative o Docker Swarm

```bash
docker swarm init
```

Se aparecer `Swarm initialized`, está pronto. Um erro dizendo que já existe um
swarm pode ser ignorado.

### Passo 4 — Execute o instalador

```bash
curl -fsSL https://raw.githubusercontent.com/bortolottic/vision.ai.installer/refs/heads/main/install.sh | bash
```

Aguarde até aparecer:

```
======================================
 Deploy finalizado com sucesso
======================================
```

### Passo 5 — Verifique os serviços

```bash
docker service ls
```

Compare com a tabela de [serviços que devem estar rodando](#serviços-que-devem-estar-rodando)
e siga para [Depois de instalar](#depois-de-instalar).

---

## Instalação no Windows

No Windows, o Vision.AI roda dentro do **WSL2** com o **Docker Desktop**.

### Passo 1 — Verifique a versão do Windows

WSL2 e Docker Desktop exigem **Windows 10 versão 2004 ou superior** (ou Windows
11). Pressione `Win + R`, digite `winver` e confira.

### Passo 2 — Ative o WSL2

Abra o **PowerShell como administrador** e rode:

```powershell
wsl --install
```

Reinicie quando pedido e crie usuário/senha do Linux.

### Passo 3 — Instale o Docker Desktop

Baixe de `https://www.docker.com/products/docker-desktop/`, instale com a opção
**"Use WSL 2 instead of Hyper-V"** marcada, e abra o Docker Desktop. Em
**Settings → Resources → WSL Integration**, ative a integração com o Ubuntu e
clique em **Apply & Restart**.

### Passo 4 — Ative o Swarm e execute o instalador

Abra o **Ubuntu (WSL)** pelo menu Iniciar e rode:

```bash
docker swarm init
curl -fsSL https://raw.githubusercontent.com/bortolottic/vision.ai.installer/refs/heads/main/install.sh | bash
```

### Passo 5 — Verifique os serviços

```bash
docker service ls
```

---

## Instalação no macOS

O Docker Desktop para macOS roda nativamente (Intel e Apple Silicon).

> **Nota sobre GPU:** o `--gpu` vale só para placas NVIDIA em Linux/WSL2. No Mac,
> a inferência roda em CPU.

### Passo 1 — Abra o Terminal

`Cmd + Espaço` → digite `Terminal` → Enter.

### Passo 2 — Instale o Homebrew e o Docker Desktop

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install --cask docker
```

Abra o Docker Desktop pelo Launchpad e aguarde o ícone da baleia ficar estável
na barra de menus.

### Passo 3 — Ative o Swarm e execute o instalador

```bash
docker swarm init
curl -fsSL https://raw.githubusercontent.com/bortolottic/vision.ai.installer/refs/heads/main/install.sh | bash
```

### Passo 4 — Verifique os serviços

```bash
docker service ls
```

---

## Depois de instalar

### Passo 1 — Acesse a plataforma

```
http://<endereço-da-máquina>:9080
```

No próprio computador, use `http://localhost:9080`.

### Passo 2 — Os segredos já foram gerados para você

Diferente de outros instaladores, **você não precisa trocar senhas à mão**. Na
primeira instalação, o `bootstrap-secrets.sh` gera valores aleatórios para
`TOKEN_SECRET`, `VISION_SECRET_KEY`, `VISION_FEEDBACK_SECRET`, a senha do banco,
a do RabbitMQ, a chave do MinIO e a senha da porta de controle do gateway de
mídia — e grava tudo em `/tmp/vision-ai/vision-ai/.env.production` (com
permissão `600`).

> **Faça backup desse arquivo.** Esses valores existem só nele. E guarde-o:
> trocar `VISION_SECRET_KEY` depois que há câmeras cadastradas torna as senhas
> delas ilegíveis.

### Passo 3 — Ajuste o endereço público

Se você instalou em um servidor (e não no próprio computador), edite
`/tmp/vision-ai/vision-ai/.env.production` e ajuste:

```
VISION_PUBLIC_URL=http://<endereço-da-máquina>:9080
# IP/host por onde o navegador alcança o UDP do WebRTC (vídeo ao vivo):
VISION_WEBRTC_HOST=<endereço-da-máquina>
```

Depois suba a stack de novo a partir da mesma pasta:

```bash
cd /tmp/vision-ai/vision-ai && ./deploy_production.sh
```

Sem `VISION_PUBLIC_URL` correto, o link de qualificação das notificações aponta
para `localhost`; sem `VISION_WEBRTC_HOST`, o vídeo ao vivo só funciona na
própria máquina.

---

### Serviços que devem estar rodando

O `docker service ls` deve listar os serviços abaixo com as réplicas completas:

| Serviço | Réplicas | O que faz |
| ------- | -------- | --------- |
| `vision-ai_nginx` | 1/1 | Porta de entrada (atende a 9080) |
| `vision-ai_hub` | 2/2 | A tela da plataforma |
| `vision-ai_api` | 1/1 | O motor da aplicação (roda as migrações na partida) |
| `vision-ai_ingest` | 1/1 | Captura o vídeo das câmeras |
| `vision-ai_infer` | 1/1 | Inferência (detecção) |
| `vision-ai_rules` | 1/1 | Motor de regras |
| `vision-ai_notify` | 1/1 | Entrega das notificações |
| `vision-ai_aggregator` | 1/1 | Persiste a detecção (histórico) |
| `vision-ai_trainer` | 1/1 | Worker de treino |
| `vision-ai_mcp` | 1/1 | Servidor MCP para agentes |
| `vision-ai_media` | 1/1 | Gateway de vídeo ao vivo (MediaMTX) |
| `vision-ai_db` | 1/1 | Banco TimescaleDB |
| `vision-ai_redis` | 1/1 | Cache, frames e leases |
| `vision-ai_broker` | 1/1 | Filas (RabbitMQ) |
| `vision-ai_objects` | 1/1 | Armazém de objetos (MinIO) |

> O `vision-ai_broker-init` aparece e some: ele aplica o teto das filas e sai.

Um serviço parado em `0/1` por mais de alguns minutos indica problema:

```bash
docker stack ps vision-ai --no-trunc
```

A coluna **ERROR** mostra a causa — normalmente porta ocupada, memória
insuficiente ou falha ao baixar a imagem.

---

## Rodar na GPU

Para a inferência e o treino usarem uma placa NVIDIA:

1. Instale o **NVIDIA Container Toolkit** e confirme:
   ```bash
   docker run --rm --gpus all nvidia/cuda:12.6.0-base-ubuntu24.04 nvidia-smi
   ```
2. No `/etc/docker/daemon.json` do nó de GPU, anuncie o recurso e reinicie o Docker:
   ```json
   { "default-runtime": "nvidia", "node-generic-resources": [ "NVIDIA-GPU=0" ] }
   ```
3. Suba com `--gpu` a partir da pasta de instalação:
   ```bash
   cd /tmp/vision-ai/vision-ai && . deploy/deploy.sh --gpu vision-ai .env.production
   ```

Com duas placas, aponte `VISION_TRAINER_DEVICE=cuda:1` no `.env.production` para
o treino não roubar quadro da inferência.

---

## Instalação com HTTPS

O deploy padrão sobe em HTTP na porta 9080. Para publicar com certificado:

```bash
cd /tmp/vision-ai/vision-ai
./deploy/scripts/prepare-ssl.sh /caminho/do/certificado.pfx
. deploy/deploy.sh --ssl vision-ai .env.production
```

A plataforma passa a atender em `https://<host>/` (porta 443); a 9080 redireciona
para HTTPS.

---

## Instalar uma versão específica

```bash
curl -fsSL https://raw.githubusercontent.com/bortolottic/vision.ai.installer/refs/heads/main/install.sh | bash -s V1.0.0.0
```

Troque `V1.0.0.0` pela versão desejada. Todas estão na página de
[Releases](https://github.com/bortolottic/vision.ai.installer/releases).

---

## O que o instalador faz (para técnicos)

1. **Detecção de versão** — consulta a API do GitHub (`/releases/latest`) se
   nenhuma versão for informada.
2. **Download** — baixa `vision-ai-<versão>.tar.gz` dos releases deste repositório.
3. **Extração** — descompacta em `/tmp/vision-ai`.
4. **Deploy** — roda `deploy_production.sh`, que chama `deploy/deploy.sh` com a
   stack `vision-ai` e o `.env.production`. Esse script: gera os segredos de uma
   instalação nova (`bootstrap-secrets.sh`), valida as variáveis obrigatórias e
   recusa segredos de fábrica, registra a config do nginx como *Docker config* e
   sobe a stack via `docker stack deploy`.

```
GitHub Releases
      │
      ▼
install.sh ─► baixa .tar.gz ─► extrai ─► deploy_production.sh ─► deploy/deploy.sh ─► Docker Swarm
```

**Imagens no DockerHub:** `visionai-{api,ingest,infer,trainer,rules,notify,aggregator,mcp,hub}`
(e `visionai-{infer,trainer}:<versão>-gpu`). Os serviços de base usam imagens
oficiais: `timescale/timescaledb`, `redis`, `rabbitmq`, `quay.io/minio/minio` e
`bluenviron/mediamtx`.

**Onde ficam os dados:** em volumes do Docker (`db_data`, `broker_data`,
`object_data`, `model_cache`, `exports_data`). Removê-los apaga tudo.

---

## Solução de problemas

| Problema | Possível causa | Solução |
| -------- | -------------- | ------- |
| `curl ou wget não encontrado` | Nenhum instalado | `sudo apt install -y curl` |
| Erro ao baixar o pacote | Versão inexistente | Veja as [releases](https://github.com/bortolottic/vision.ai.installer/releases) |
| `Docker not found` | Docker fora do PATH | Instale o Docker e confirme que está rodando |
| `permission denied` no Docker | Usuário fora do grupo docker | `sudo usermod -aG docker $USER` e reabra o terminal |
| `This node is already part of a swarm` | Swarm já iniciado | Pode ignorar |
| `Variáveis obrigatórias faltando` | `.env.production` com variável vazia | Preencha a variável citada e rode `./deploy_production.sh` |
| `segredos de fábrica ainda em .env` | Arquivo editado à mão | Gere um valor novo: `openssl rand -hex 32` |
| Termina com sucesso mas a tela não abre | Um serviço não subiu | `docker stack ps vision-ai --no-trunc` e leia ERROR |
| Serviço em `0/1` com erro de porta | Porta ocupada | Veja [Portas](#portas-que-precisam-estar-livres) e libere |
| Serviço reiniciando sem parar | Memória insuficiente | 8 GB de RAM no mínimo |
| Vídeo ao vivo não abre em servidor | `VISION_WEBRTC_HOST` vazio | Ajuste-o no `.env.production` e suba de novo |
| GPU ignorada mesmo com `--gpu` | Toolkit/daemon não configurados | Veja [Rodar na GPU](#rodar-na-gpu) |
| Docker Desktop não inicia (Windows) | Virtualização desativada na BIOS | Ative VT-x/AMD-V na BIOS |

---

## Suporte

Em caso de dúvidas, entre em contato com a equipe responsável pelo projeto.
