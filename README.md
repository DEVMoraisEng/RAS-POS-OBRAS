# RAS-POS-OBRA

Reunião de Alinhamento Semanal do Departamento de Pós-Obra e Reparos — Morais Engenharia e Construção.

Mesma arquitetura do RAS Financeiro / RAS Projetos: Python lê o Notion via GitHub Actions, publica JSON estático em `dist/`, e o `index.html` consome esse JSON. Nenhum token aparece no navegador.

**Banco de dados:** ATIVIDADES RAS Pós-Obra — `3bfc5ab5-32d3-801b-8a4a-c5b83b8b608b`

**Setores (pessoas):** Paula, Caio

## Passos para colocar no ar

### 1. Criar o repositório

Suba estes arquivos em um repositório novo (`RAS-POS-OBRA`), branch `main`. Confirme que os workflows ficaram em `.github/workflows/`.

### 2. Secret

Em *Settings → Secrets and variables → Actions*, criar `NOTION_TOKEN` com o token da integração (pode ser o mesmo já usado nos outros RAS, desde que a integração tenha acesso a este banco também).

### 3. Compartilhar o banco com a integração

No Notion, abrir o banco ATIVIDADES RAS Pós-Obra → `···` → *Conexões* → adicionar a integração.

### 4. Conferir as opções do Notion

```bash
export NOTION_TOKEN="..."
python3 verificar_status.py
```

O site espera os Status: **A Fazer, Em Andamento, Pendente, Concluído, Continuidade da Semana Anterior**. Se a coluna Status for do tipo `status` (não `select`), todas essas opções precisam existir no Notion com esse texto exato.

### 5. Primeiro fetch

*Actions → RAS Pós-Obra - atualizar dados → Run workflow.* Gera `dist/data_atividades.json`.

### 6. GitHub Pages

*Settings → Pages → Source: **Deploy from a branch*** → Branch `main`, pasta `/ (root)` → Save. (Se o Source já estiver em "GitHub Actions" ou o campo Branch estiver em "None", o link dá 404 até você corrigir isso.)

### 7. Escrita no Notion (Apps Script)

`Code.gs` já está neste repositório. Passos:

1. script.google.com → *Novo projeto* → colar o conteúdo de `Code.gs`.
2. Propriedades do script:

| Propriedade | Valor |
|---|---|
| `NOTION_TOKEN` | o token da integração |
| `RAS_POSOBRA_DB_ID` | `3bfc5ab532d3801b8a4ac5b83b8b608b` |
| `GITHUB_REPO` | `DEVMoraisEng/RAS-POS-OBRA` |
| `GITHUB_TOKEN` | Personal Access Token do GitHub com escopo `repo` |

3. **PEOPLE_IDS** — ainda vazio no `Code.gs`. Preencha o Responsável de pelo menos uma linha por pessoa (Paula, Caio) direto no Notion, rode `extrair_ids_responsavel.py` (duplo clique — pede o token na tela, não fecha sozinho), e cole o bloco `PEOPLE_IDS` impresso no `Code.gs`.

4. Implantar → Nova implantação → tipo **App da Web** → Executar como **Eu** → Quem pode acessar: **Qualquer pessoa**.

   ⚠️ **Não escolha "Somente eu"** — com acesso restrito, o `fetch` do navegador esbarra numa tela de login do Google em vez de rodar o `doGet`, e as gravações falham sem erro claro.

5. Copie a URL `/exec` e cole em `const WRITE_ENDPOINT = "";` no `index.html`. Suba o arquivo atualizado.

6. Toda vez que editar o `Code.gs` depois disso: *Gerenciar implantações → lápis → Nova versão* (nunca "Nova implantação").

7. Testar: rode `testeConexao` no editor e depois `testeUpdateReal` (edite o `idReal` no código pra um ID real, pego do `dist/data_atividades.json` publicado).

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | O site (single file) |
| `Code.gs` | Apps Script — proxy de escrita para o Notion |
| `fetch_ras.py` | Lê o Notion → gera `dist/data_atividades.json` |
| `rollover_semana.py` | Domingo à noite: empurra o não concluído pra semana nova |
| `verificar_status.py` | Compara as opções do site com as do Notion |
| `extrair_ids_responsavel.py` | Extrai os IDs de usuário do Notion pro `PEOPLE_IDS` |
