# 🔐 AI-Hardened — Private RAG Stack (100% Offline)

<p align="center">
  <img src="https://img.shields.io/badge/status-production--ready-00ff88?style=for-the-badge" />
  <img src="https://img.shields.io/badge/data%20egress-ZERO-ff0000?style=for-the-badge" />
  <img src="https://img.shields.io/badge/encryption-AES--256--XTS-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/compliance-LGPD%20%7C%20ISO27001%20%7C%20SOC2-orange?style=for-the-badge" />
</p>

> **Infraestrutura de IA Generativa com isolamento total de rede, armazenamento vetorial criptografado e auditoria forense — projetada para setores Jurídico, Bancário e Governamental.**

---

## 🎯 O Problema que Resolvemos

| Cenário | Solução Tradicional | AI-Hardened |
|---|---|---|
| Análise de contratos sigilosos | Enviado para OpenAI/Azure ☠️ | Processado localmente ✅ |
| Documentos de clientes bancários | Nuvem pública com PII ☠️ | Volume LUKS isolado ✅ |
| Dados de investigação criminal | API de terceiros ☠️ | Air-gap + auditoria ✅ |
| Segredos de estado | SaaS com logging ☠️ | Zero egress garantido ✅ |

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                    PERÍMETRO ISOLADO (air-gap)                   │
│                                                                   │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────────────────┐ │
│  │   CLIENTE   │    │   API LAYER  │    │   OLLAMA ENGINE     │ │
│  │  (Browser)  │───▶│  FastAPI +   │───▶│  Llama 3 / Phi-3   │ │
│  │  localhost  │    │  rate-limit  │    │  Mistral / Gemma    │ │
│  └─────────────┘    └──────┬───────┘    └─────────────────────┘ │
│                             │                                     │
│                    ┌────────▼────────┐                           │
│                    │  VECTOR STORE   │                           │
│                    │  ChromaDB +     │                           │
│                    │  LangChain RAG  │                           │
│                    └────────┬────────┘                           │
│                             │                                     │
│              ┌──────────────▼──────────────┐                    │
│              │     LUKS ENCRYPTED VOLUME    │                    │
│              │  /dev/mapper/ai_vault        │                    │
│              │  AES-256-XTS + SHA-512       │                    │
│              │  Auto-lock on suspend/halt   │                    │
│              └─────────────────────────────┘                    │
│                                                                   │
│  🚫 ZERO conexões externas — iptables DROP por padrão           │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Quick Start

### Pré-requisitos

```bash
# Ubuntu 22.04+ / Debian 12+
sudo apt install -y ollama cryptsetup python3-pip docker.io

# Verificar isolamento de rede (deve retornar 0 conexões externas)
./scripts/verify_isolation.sh
```

### 1. Criar Volume Criptografado

```bash
sudo ./scripts/setup_luks_vault.sh --size 50G --keyfile /etc/ai-hardened/vault.key
```

### 2. Subir Stack Completo

```bash
cp configs/env.example .env
docker-compose up -d
```

### 3. Ingerir Documentos

```bash
python3 src/rag/ingest.py --path /mnt/ai_vault/docs/ --collection juridico
```

### 4. Consultar

```bash
curl -X POST http://localhost:8080/api/query \
  -H "Content-Type: application/json" \
  -d '{"question": "Quais são as cláusulas de rescisão do contrato X?", "collection": "juridico"}'
```

---

## 🔒 Garantias de Segurança

### Isolamento de Rede

O sistema opera com **política de firewall padrão DROP** em todas as interfaces externas. Apenas tráfego loopback (`lo`) é permitido.

```bash
# Regras aplicadas automaticamente pelo setup
iptables -P OUTPUT DROP
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
# Nenhuma outra regra de saída é adicionada
```

**Verificação auditável:**
```bash
./scripts/verify_isolation.sh
# Expected output:
# [✓] No DNS queries to external resolvers
# [✓] No outbound TCP/UDP connections
# [✓] Ollama bound to 127.0.0.1 only
# [✓] ChromaDB not exposed externally
```

### Criptografia em Repouso

| Camada | Algoritmo | Detalhe |
|---|---|---|
| Volume LUKS | AES-256-XTS | Padrão NIST SP 800-38E |
| Hash do header | SHA-512 | Resistente a colisão |
| KDF | PBKDF2 / Argon2id | Anti-brute-force |
| Auto-lock | systemd trigger | Lock em suspend/shutdown |

### Auditoria e Rastreabilidade

Cada query ao sistema gera um log imutável:

```json
{
  "timestamp": "2026-04-12T14:32:11Z",
  "user_hash": "sha256:a3f...",
  "query_hash": "sha256:b7c...",
  "collection": "juridico",
  "model": "llama3:8b",
  "tokens_used": 847,
  "egress_bytes": 0,
  "vault_sealed": false
}
```

---

## 📋 Modelos Suportados

| Modelo | Tamanho | VRAM | Uso Recomendado |
|---|---|---|---|
| `llama3:8b` | 4.7GB | 8GB | Uso geral, jurídico |
| `llama3:70b` | 40GB | 48GB+ | Análise densa, bancário |
| `phi3:mini` | 2.3GB | 4GB | Edge, dispositivos limitados |
| `mistral:7b` | 4.1GB | 6GB | Multilíngue, governamental |
| `gemma2:9b` | 5.5GB | 8GB | Código, compliance técnico |

---

## 🏛️ Casos de Uso por Setor

### ⚖️ Jurídico
- Análise de contratos sem vazar conteúdo para terceiros
- Busca semântica em acervos de jurisprudência internos
- Due diligence em M&A com documentos sob NDA

### 🏦 Bancário
- Análise de documentos KYC/AML localmente
- Chatbot interno para compliance sem expor PII
- Auditoria de contratos de crédito

### 🏛️ Governamental
- Análise de documentos classificados
- Suporte à decisão em investigações
- Processamento de dados sob sigilo fiscal (Lei 9.784/99)

---

## 📁 Estrutura do Repositório

```
AI-Hardened/
├── README.md                   ← Este arquivo
├── docker-compose.yml          ← Stack completo
├── configs/
│   ├── env.example             ← Variáveis de ambiente
│   ├── iptables.rules          ← Firewall de isolamento
│   ├── ollama.conf             ← Ollama com bind localhost
│   └── chromadb.yaml           ← ChromaDB persistente
├── scripts/
│   ├── setup_luks_vault.sh     ← Criação do volume LUKS
│   ├── mount_vault.sh          ← Mount autenticado
│   ├── unmount_vault.sh        ← Seal + lock
│   └── verify_isolation.sh     ← Auditoria de isolamento
├── src/
│   ├── rag/
│   │   ├── ingest.py           ← Ingestão de documentos
│   │   ├── query.py            ← Motor de busca semântica
│   │   └── embeddings.py       ← Embeddings locais (nomic)
│   ├── api/
│   │   ├── main.py             ← FastAPI server
│   │   ├── auth.py             ← JWT local
│   │   └── audit.py            ← Log imutável
│   └── ui/
│       └── index.html          ← Interface web local
├── docs/
│   ├── SECURITY.md             ← Modelo de ameaças
│   ├── ARCHITECTURE.md         ← Diagrama detalhado
│   ├── COMPLIANCE.md           ← LGPD / ISO 27001
│   └── THREAT_MODEL.md         ← STRIDE analysis
└── tests/
    ├── test_isolation.py       ← Testes de egress
    └── test_encryption.py      ← Testes de criptografia
```

---

## 🛡️ Modelo de Ameaças (Resumo)

Ver [`docs/THREAT_MODEL.md`](docs/THREAT_MODEL.md) para análise STRIDE completa.

| Ameaça | Mitigação |
|---|---|
| Exfiltração via modelo | Ollama bind 127.0.0.1, iptables DROP outbound |
| Acesso físico ao disco | LUKS AES-256-XTS, auto-lock em shutdown |
| Comprometimento da supply chain | Hashes SHA-256 verificados de modelos |
| Insider threat | Logs imutáveis com hash de usuário |
| Memory scraping | Vault desmontado quando inativo |

---

## 🔧 Contribuindo

Este projeto é mantido pela [**SELOCK**](https://selock.net) — especialistas em segurança de dispositivos móveis, hardening e compliance.

Para contribuições, abra uma issue com:
1. Descrição do vetor de ataque ou melhoria
2. Evidência técnica / PoC
3. Proposta de mitigação

---

## 📜 Licença

MIT License — veja [LICENSE](LICENSE)

---

<p align="center">
  <strong>Construído por <a href="https://selock.net">SELOCK</a> · Salvador, Bahia · Brasil</strong><br/>
  <em>IA para setores que não podem errar.</em>
</p>
