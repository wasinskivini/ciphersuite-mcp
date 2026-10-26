# @viniciuswasinski/ciphersuite-mcp

Servidor [MCP (Model Context Protocol)](https://modelcontextprotocol.io) construído do zero que expõe ferramentas de **criptografia e descriptografia AES-256-CBC**, um resource descrevendo o algoritmo e um prompt pronto para uso — tudo utilizável diretamente no Copilot Chat do VS Code (ou em qualquer cliente MCP).

> Projeto desenvolvido no **Módulo 03 — MCP na Prática** da pós em Engenharia de Software com IA Aplicada.

---

## O que o servidor oferece

| Tipo | Nome | Descrição |
|---|---|---|
| 🔧 Tool | `encrypt_message` | Criptografa uma mensagem em texto puro usando uma passphrase |
| 🔧 Tool | `decrypt_message` | Descriptografa uma mensagem gerada por `encrypt_message`, usando a mesma passphrase |
| 📄 Resource | `encryption://info` | Retorna detalhes do algoritmo, da derivação de chave e do formato de saída |
| 💬 Prompt | `encrypt_message_prompt` | Prompt pré-montado que pede ao agente para criptografar uma mensagem |

As tools retornam tanto `content` (texto) quanto `structuredContent` (validado por `outputSchema` com Zod). Em caso de falha, retornam `isError: true` com uma mensagem explicativa em vez de derrubar o servidor.

### Como a criptografia funciona

- **Algoritmo:** AES-256-CBC (`node:crypto`)
- **Derivação de chave:** `scrypt(passphrase, saltFixo, 32)` — o usuário informa qualquer passphrase e o servidor deriva uma chave forte de 32 bytes
- **Formato de saída:** `<IV em hex>:<ciphertext em hex>` — guarde a string completa para conseguir descriptografar depois
- **IV aleatório:** um IV de 16 bytes é gerado a cada chamada, então a mesma mensagem criptografada duas vezes produz saídas diferentes

> ⚠️ Projeto com fins didáticos: o salt é fixo no código e não há autenticação do ciphertext (MAC/AEAD). Não use em produção para dados sensíveis.

---

## Pré-requisitos

- **Node.js v24+** (veja `engines` no `package.json`)

## Instalação

```bash
npm install
```

Não há etapa de build — o servidor roda TypeScript diretamente graças ao suporte nativo de type stripping do Node.js.

---

## Usando no VS Code

O projeto já inclui [.vscode/mcp.json](.vscode/mcp.json):

```json
{
  "servers": {
    "ciphersuite-mcp": {
      "command": "node",
      "args": ["--experimental-strip-types", "src/index.ts"]
    }
  }
}
```

Para usar em outro workspace, troque `src/index.ts` pelo caminho absoluto do arquivo.

1. Abra esta pasta no VS Code e recarregue a janela (`Cmd+Shift+P` → **Developer: Reload Window**).
2. Inicie o servidor `ciphersuite-mcp` pela lista de MCP servers.
3. No Copilot Chat (modo Agent), experimente:

```
Criptografe a mensagem "Olá, mundo!" usando a passphrase "minha-chave-secreta"
```

```
Descriptografe esta mensagem: <iv>:<ciphertext> usando a passphrase "minha-chave-secreta"
```

```
Mostre o resource encryption://info
```

---

## MCP Inspector

Para explorar e testar tools, resources e prompts por uma interface web:

```bash
npm run mcp:inspect
```

Referência: <https://modelcontextprotocol.io/docs/tools/inspector>

---

## Testes

Os testes de integração sobem o servidor via stdio e se conectam com um `Client` do SDK MCP ([tests/helpers.ts](tests/helpers.ts)).

```bash
npm test          # roda todos os testes
npm run test:dev  # modo watch com debugger
```

Cobertura atual ([tests/mcp.test.ts](tests/mcp.test.ts)):

- Criptografar uma mensagem
- Criptografar e depois descriptografar com a mesma passphrase
- Listar o resource `encryption://info`
- Obter o `encrypt_message_prompt` no formato esperado

---

## Estrutura do projeto

```
src/
  index.ts     # Ponto de entrada — conecta o servidor ao transporte stdio
  mcp.ts       # Registro das tools, resource e prompt
  service.ts   # Lógica de criptografia (AES-256-CBC + scrypt)
tests/
  helpers.ts   # Cria o client MCP de teste
  mcp.test.ts  # Testes de integração
.vscode/
  mcp.json     # Configuração do servidor para o VS Code
```

## Scripts disponíveis

| Script | Descrição |
|---|---|
| `npm start` | Inicia o servidor (usado pelos clientes MCP) |
| `npm run dev` | Inicia com file-watch e inspector do Node.js |
| `npm test` | Roda todos os testes |
| `npm run test:dev` | Roda os testes em modo watch |
| `npm run mcp:inspect` | Abre a UI do MCP Inspector |

## Stack

- [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) — servidor e client MCP
- [`zod`](https://zod.dev) — schemas de input/output
- `node:crypto` e `node:test` — criptografia e testes sem dependências extras
