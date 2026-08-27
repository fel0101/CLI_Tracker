# CLI_Tracker
Projeto de aprendizado, feito em python para aprendizado inicial das funções e sintaxe da linguagem.
 Tarefa CLI

Uma ferramenta de linha de comando (CLI) simples, feita em Python puro, para gerenciar tarefas do dia a dia: adicionar, listar, atualizar, marcar como em andamento/concluída e deletar. As tarefas são salvas em um arquivo `tasks.json` na mesma pasta do script.

Este projeto foi construído como exercício de aprendizado de Python, praticando: argumentos de linha de comando (`sys.argv`), leitura/escrita de arquivos JSON, manipulação de listas e dicionários, datas (`datetime`) e tratamento de erros (`try/except`).

## Requisitos

- Python instalado (versão 3.x)
- Nenhuma biblioteca externa é necessária — o projeto usa apenas módulos nativos do Python (`sys`, `json`, `os`, `datetime`)

## Como rodar

Abra o terminal na pasta onde está o arquivo `tarefa_cli.py` e execute:

```bash
python tarefa_cli.py <comando> [argumentos]
```

Se nenhum comando for informado, o programa avisa:
```
Nenhum comando fornecido.
```

## Comandos disponíveis

### Adicionar uma tarefa
```bash
python tarefa_cli.py adicionar "Comprar mantimentos"
```
Cria uma nova tarefa com status inicial `todo`. Saída esperada:
```
Tarefa adicionada com sucesso: (Id: 1)
```

### Listar tarefas
```bash
python tarefa_cli.py listar
```
Lista todas as tarefas cadastradas.

Também é possível filtrar por status:
```bash
python tarefa_cli.py listar todo
python tarefa_cli.py listar in-progress
python tarefa_cli.py listar done
```

### Marcar tarefa como "em andamento"
```bash
python tarefa_cli.py mark-in-progress 1
```
Altera o status da tarefa de ID informado para `in-progress`.

### Marcar tarefa como "concluída"
```bash
python tarefa_cli.py mark-done 1
```
Altera o status da tarefa de ID informado para `done`.

### Atualizar a descrição de uma tarefa
```bash
python tarefa_cli.py atualizar 1 "Comprar mantimentos e cozinhar o jantar"
```
Substitui a descrição da tarefa de ID informado pela nova descrição.

### Deletar uma tarefa
```bash
python tarefa_cli.py deletar 1
```
Remove permanentemente a tarefa de ID informado.

## Estrutura de uma tarefa

Cada tarefa é armazenada como um objeto (dicionário) com estas propriedades:

| Campo | Descrição |
|---|---|
| `id` | Identificador numérico único, gerado automaticamente |
| `description` | Texto descrevendo a tarefa |
| `status` | Um de: `todo`, `in-progress`, `done` |
| `createdAt` | Data/hora de criação, no formato ISO 8601 |
| `updatedAt` | Data/hora da última atualização, no formato ISO 8601 |

Exemplo de `tasks.json`:
```json
[
  {
    "id": 1,
    "description": "Comprar mantimentos",
    "status": "todo",
    "createdAt": "2026-08-13T20:12:51.170390",
    "updatedAt": "2026-08-13T20:12:51.170390"
  }
]
```

## Tratamento de erros

O programa lida com situações inesperadas sem travar:
- Comando não reconhecido → avisa qual comando foi digitado e que não existe
- Faltou informar descrição/ID → avisa qual informação está faltando
- ID informado não é um número → avisa que o ID precisa ser um número inteiro
- ID informado não corresponde a nenhuma tarefa → avisa que a tarefa não foi encontrada

---

## Como o código funciona (por dentro)

Esta seção explica cada função do `tarefa_cli.py` e a lógica de controle de fluxo usada.

### Imports

```python
import sys        # lê os argumentos digitados no terminal (sys.argv)
import json        # converte entre texto JSON e listas/dicionários Python
import os          # verifica se o arquivo tasks.json existe
from datetime import datetime   # gera a data/hora atual para createdAt/updatedAt
```

### `carregar_tarefas()`
Lê o arquivo `tasks.json` e retorna a lista de tarefas.
- Usa `if os.path.exists("tasks.json")` para checar se o arquivo já existe.
- Se existir, abre com `with open(...)` e usa `json.load()` para transformar o texto JSON em uma lista Python.
- Se não existir (por exemplo, na primeira vez que o programa roda), retorna uma lista vazia `[]`, para o resto do código não quebrar.

### `salvar_tarefas(tarefas)`
Recebe a lista de tarefas e grava no arquivo `tasks.json`.
- Abre o arquivo em modo `"w"` (escrita) — que cria o arquivo automaticamente se ele não existir, ou sobrescreve se já existir.
- Usa `json.dump()` para converter a lista Python de volta em texto JSON e escrever no arquivo.

### `adicionar_tarefa(argumentos_extras)`
Cria uma nova tarefa e salva.
- `if len(argumentos_extras) == 0`: verifica se a descrição foi informada. Se não foi, avisa o usuário e sai da função com `return` (sem essa checagem, tentar acessar `argumentos_extras[0]` numa lista vazia quebraria o programa).
- Calcula o novo `id`: `1` se a lista de tarefas estiver vazia, ou o `id` da última tarefa + 1 caso contrário.
- Usa `datetime.now().isoformat()` para gerar a data/hora atual, usada tanto em `createdAt` quanto em `updatedAt`.
- Monta um dicionário com todos os campos da tarefa, adiciona na lista com `.append()`, e chama `salvar_tarefas()`.

### `listar_tarefas(argumentos_extras)`
Mostra as tarefas na tela, com filtro opcional por status.
- Se `argumentos_extras` tiver algum item, esse item vira o `filtro` (ex: `"done"`); senão, `filtro = None`.
- Um laço `for tarefa in tarefas` percorre cada tarefa da lista.
- A condição `if filtro is None or tarefa["status"] == filtro` decide o que imprimir: se não há filtro, mostra tudo; se há filtro, só mostra quando o status bate.

### `mark_in_progress(argumentos_extras)` e `mark_done(argumentos_extras)`
Alteram o status de uma tarefa específica (para `"in-progress"` ou `"done"`, respectivamente). A lógica das duas é idêntica, só muda o valor atribuído a `status`:
- Verifica se o ID foi informado (`len(argumentos_extras) == 0`).
- Usa `try / except ValueError` para converter o ID (texto) em número com `int(...)`. Se a conversão falhar (o usuário digitou algo que não é número), o `except` captura o erro e avisa, em vez de o programa travar.
- Percorre as tarefas com `for`, procurando a que tem o `id` igual ao informado.
- Ao encontrar: atualiza `status` e `updatedAt`, salva, avisa sucesso, e usa `return` para sair da função imediatamente (evitando continuar o `for` à toa).
- Se o `for` terminar sem encontrar nenhuma tarefa com esse `id`, a linha `print("não encontrada")` (fora do `for`) é executada.

### `atualizar_tarefa(argumentos_extras)`
Atualiza a descrição de uma tarefa.
- Verifica se **dois** argumentos foram informados (`len(argumentos_extras) < 2`): o ID e a nova descrição.
- Mesmo padrão de `try/except` para converter o ID.
- Percorre as tarefas, encontra pela `id`, atualiza `description` e `updatedAt`, salva e avisa sucesso.

### `deletar_tarefa(argumentos_extras)`
Remove uma tarefa da lista.
- Mesma checagem de argumento e conversão de ID das funções anteriores.
- Percorre as tarefas; ao encontrar a de `id` correspondente, usa `tarefas.remove(tarefa)` para removê-la da lista, salva o restante, e sai com `return`.

### Bloco principal (roteador de comandos)

```python
quantidade = len(sys.argv)

if quantidade == 1:
    print("Nenhum comando fornecido.")
else:
    comando = sys.argv[1]
    argumentos_extras = sys.argv[2:]

    if comando == "adicionar":
        adicionar_tarefa(argumentos_extras)
    elif comando == "listar":
        listar_tarefas(argumentos_extras)
    elif comando == "mark-in-progress":
        mark_in_progress(argumentos_extras)
    elif comando == "mark-done":
        mark_done(argumentos_extras)
    elif comando == "atualizar":
        atualizar_tarefa(argumentos_extras)
    elif comando == "deletar":
        deletar_tarefa(argumentos_extras)
    else:
        print(f"Comando '{comando}' não reconhecido")
```

Esta é a parte que **conecta tudo**: lê o que foi digitado no terminal (`sys.argv`), separa o comando (`sys.argv[1]`) do restante dos argumentos (`sys.argv[2:]`), e usa a cadeia `if / elif / else` para decidir qual função chamar. Cada `elif` é testado em ordem, até um bater — e se nenhum bater, cai no `else` final, avisando que o comando não existe.

## Possíveis melhorias futuras
- Adicionar testes automatizados
- Permitir editar o status junto com a descrição no comando `atualizar`
- Empacotar o script para ser chamado como `tarefa-cli` diretamente, sem precisar digitar `python` antes
