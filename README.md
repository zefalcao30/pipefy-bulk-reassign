# Pipefy — Transferência em massa de responsável

Reatribui o **responsável (assignee)** de **todos os cards abertos**, em **todos os pipes**, de um usuário para outro — via API GraphQL do Pipefy.

Pensado para o cenário clássico: alguém sai do time e deixa dezenas de cards espalhados. Em vez de reatribuir um por um na interface, o script varre a conta inteira e faz a troca de forma **reproduzível e auditável**, com um modo de simulação antes de aplicar qualquer alteração.

## Por que existe

Fazer isso na mão significa abrir cada pipe, filtrar por responsável, abrir cada card e trocar o assignee — uma tarde inteira de cliques e uma boa chance de esquecer algum. Este notebook resolve em minutos.

## Recursos

- **Modo de simulação (`DRY_RUN`)** — lista tudo o que *seria* alterado antes de tocar em qualquer card.
- **Prévia com contagem** — tabela por pipe (quantos cards) e total geral, sem alterar nada.
- **Preserva co-responsáveis** — a mutation `updateCard` substitui a lista inteira de responsáveis; o script remove apenas o antigo e mantém os demais.
- **Só cards abertos** — cards já concluídos (`done = true`) são ignorados.
- **Respeita o rate limit** — paginação de 50 em 50, pausas entre chamadas e *backoff* exponencial em 429.

## Pré-requisitos

- Python 3.9+
- Um [token de API do Pipefy](https://api-docs.pipefy.com/) (Configurações da conta → Tokens de API). O token herda suas permissões: você só enxerga e altera pipes aos quais tem acesso.

## Instalação

```bash
git clone https://github.com/<seu-usuario>/pipefy-bulk-reassign.git
cd pipefy-bulk-reassign
pip install -r requirements.txt
```

## Configuração do token

O token **nunca** fica escrito no notebook. Ele é lido da variável de ambiente `PIPEFY_TOKEN`.

Opção A — variável de ambiente direta:

```bash
export PIPEFY_TOKEN="seu_token_aqui"      # Linux/macOS
# setx PIPEFY_TOKEN "seu_token_aqui"      # Windows (PowerShell/cmd)
```

Opção B — arquivo `.env` (mais prático para notebooks):

```bash
cp .env.example .env
# edite o .env e cole seu token
```

O `.env` já está no `.gitignore` e não é comitado.

## Uso

Abra `pipefy_card_transfer.ipynb` e rode as células na ordem:

1. **Conexão** — carrega o token e valida.
2. **Descobrir IDs** — lista os usuários (id, nome, e-mail).
3. **Listar pipes** — mostra o que será varrido.
4. **Configuração** — preencha `OLD_USER_ID` (sai) e `NEW_USER_ID` (entra). Deixe `DRY_RUN = True`.
5. **Prévia** — tabela com a contagem de cards por pipe e o total.
6. **Executar** — com `DRY_RUN = True`, apenas simula. Revise a saída, mude para `DRY_RUN = False` e rode de novo para aplicar.

Para restringir a pipes específicos, preencha `PIPE_IDS_FILTER` com a lista de IDs.

## Detalhe técnico importante

A mutation `updateCard` com `assignee_ids` **substitui a lista inteira** de responsáveis do card — não é adicionar/remover. Se um card tem dois responsáveis e você envia só o novo, o outro é removido. Por isso o script sempre reconstrói a lista a partir da atual: `[responsáveis exceto o antigo] + novo`.

## Aviso

Este script altera dados de produção no seu Pipefy. Use sempre o `DRY_RUN` antes de aplicar e comece testando com `PIPE_IDS_FILTER` em um único pipe. O autor não se responsabiliza por alterações indevidas. Sem afiliação com o Pipefy.

## Licença

[MIT](LICENSE).
