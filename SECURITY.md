# Política de Segurança — AI-Hardened

## Versões Suportadas

| Versão | Suporte de Segurança |
|--------|---------------------|
| 1.x    | ✅ Ativo             |

## Reportando Vulnerabilidades

### Para vulnerabilidades críticas
Não abra uma issue pública. Envie um e-mail para:

**security@selock.net**

Inclua:
- Descrição detalhada do vetor de ataque
- Passos para reprodução
- Impacto potencial
- Sugestão de mitigação (se houver)

Respondemos em até **48 horas** com confirmação de recebimento e estimativa de prazo para correção.

### Para melhorias de hardening não críticas
Abra uma issue usando o template **Security Vulnerability Report**.

## Escopo

Estão **dentro do escopo**:
- Vazamento de dados para redes externas (falha do objetivo principal)
- Bypass de autenticação JWT
- Adulteração não detectada do audit log
- Escape de container Docker para o host
- Vulnerabilidades no vault LUKS

Estão **fora do escopo**:
- Vulnerabilidades em versões antigas de dependências sem PoC exploitável
- Ataques que requerem acesso físico ao servidor (mitigados pelo LUKS)
- Ataques de engenharia social

## Créditos

Reportadores de vulnerabilidades válidas serão creditados no CHANGELOG (com permissão).
