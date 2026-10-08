# 🔐 Demo — Pass\_Vault

Mode
Stack
Tests
Lint

## 🎯 Sobre o projeto

O Pass\_Vault é um gerenciador de senhas desenvolvido em Python e utilizado pelo terminal. Armazena credenciais em um cofre criptografado, protegido por uma senha mestra.

O projeto explora derivação de chaves, criptografia autenticada e proteção de arquivos para o armazenamento seguro de credenciais.

***

## 🛠️ Stack e arquitetura

| Tecnologia | Aplicação |
|---|---|
| **Python** | Implementação da ferramenta |
| **Typer** | Comandos e argumentos do CLI |
| **Rich** | Apresentação dos dados no terminal |
| **Argon2id** | Derivação da chave a partir da senha mestra |
| **AES-256-GCM** | Criptografia autenticada do cofre |

### 🔄 Fluxo principal

1. O usuário executa um comando e informa a senha mestra.
2. O Argon2id deriva a chave utilizando o salt e os parâmetros do cofre.
3. As entradas são descriptografadas e carregadas em memória.
4. O comando consulta ou modifica os dados.
5. Quando necessário, o cofre é criptografado e salvo novamente.

A escrita atômica reduz o risco de corrupção durante o salvamento. O bloqueio de arquivo coordena operações concorrentes.

***

## ⚙️ Funcionalidades implementadas

Além dos comandos originais, implementei:

| Comando ou campo | Funcionalidade |
|---|---|
| `pv search <substring>` | Busca entradas pelo nome, sem diferenciar maiúsculas e minúsculas |
| `pv count` | Imprime somente a quantidade de entradas |
| `last_used_at` | Registra e salva o horário da última consulta pelo comando `get` |
| `pv get <name>` | Exibe a senha mascarada por padrão |
| `pv get <name> --show` ou `-s` | Exibe a senha quando solicitado |
| `pv export <path>` | Exporta as entradas para JSON após confirmação |
| `pv import <path>` | Valida e importa entradas, ignorando nomes existentes |

***

## 💻 Demonstração prática

> \[\!NOTE\]
> A demonstração será realizada em ambiente próprio, utilizando um cofre separado e credenciais fictícias.

### 🔎 Gerenciamento e consulta

Demonstrar a criação do cofre e a inclusão de uma entrada com `init` e `add`.

Em seguida, executar:

- `pv list`
- `pv search GiT`
- `pv count`
- `pv get github`
- `pv get github --show`
- `pv get github -s`

Explicar como a busca funciona, por que a saída do `count` facilita seu uso em scripts e como o `get` atualiza o último uso.

Mostrar a diferença entre a senha mascarada e a senha exibida com as flags.

### 📦 Exportação e importação

Executar:

- `pv export ./pv-export.json`
- `stat -c '%a' ./pv-export.json`
- `pv import ./pv-export.json`

Mostrar o aviso de exportação e a confirmação exigida. A permissão esperada do arquivo é **`600`**.

Demonstrar também:

- Recusa da confirmação, sem criar o arquivo.
- Recusa de sobrescrita de um arquivo existente.
- Importação de entradas novas em um cofre de demonstração.
- Entradas duplicadas sendo ignoradas.
- Rejeição de JSON inválido sem alterar o cofre.

***

## 🧠 Decisões técnicas

### 🔑 Derivação de chaves e criptografia

O Argon2id deriva uma chave a partir da senha mestra, utilizando salt e parâmetros de custo. Esse custo dificulta tentativas de adivinhação da senha.

O AES-256-GCM protege a confidencialidade e permite detectar alterações no conteúdo criptografado.

### Senha mascarada por padrão

O comando `get` exige `--show` ou `-s` para exibir a senha. Essa escolha reduz sua exposição durante apresentações e compartilhamento de tela.

### 🕒 Registro do último uso

Como `Entry` é imutável, utilizei `dataclasses.replace()` para criar uma cópia com `last_used_at` atualizado. Essa cópia substitui a entrada anterior e o cofre é salvo.

Entradas antigas, sem esse campo, continuam sendo carregadas com um valor vazio.

> \[\!NOTE\]
> Esse registro acrescenta informações sobre o uso das credenciais: alguém com acesso ao conteúdo descriptografado pode identificar a entrada consultada mais recentemente.

### 📤 Exportação em texto simples

A exportação facilita a transferência das entradas, mas coloca as senhas em texto simples fora do cofre.

Por isso, exige confirmação e cria o arquivo com `os.open`, utilizando permissão `0600`. Também impede sobrescrita de arquivos existentes.

> \[\!IMPORTANT\]
> A permissão restringe o acesso ao arquivo; ela não criptografa seu conteúdo.

### 📥 Validação antes da importação

O comando verifica a estrutura do JSON e reconstrói as entradas com `Entry.from_dict()` antes de modificar o cofre.

Nomes existentes são ignorados para preservar as credenciais cadastradas. As novas entradas são salvas ao final da operação.

***

## 🧪 Validação e resultados

### ✅ Testes automatizados

**Comando executado:** `just test`.

**Resultado:** **69 testes passaram, sem falhas.**

A suíte cobre criptografia, geração de senhas e operações do cofre, incluindo senha mestra incorreta, entradas duplicadas, permissões de arquivo e escrita atômica.

> \[\!NOTE\]
> Esse resultado não comprova, sozinho, todos os novos comandos do CLI.

### 📋 Análise estática

**Comando executado:** `just lint`.

A verificação foi repetida após a integração das correções na `main`.

| Ferramenta | Resultado |
|---|---|
| **Ruff** | Todas as verificações passaram |
| **Pylint** | Nota `10.00/10` |
| **Mypy** | Nenhum problema encontrado em 7 arquivos |

A verificação foi realizada no commit `63208ef`, presente na `main` local e em `origin/main`.

### 🔍 Verificações adicionais

Os comportamentos de senha oculta por padrão, `--show` e `-s` foram verificados com dados fictícios e chamadas do CLI por `CliRunner`.

Também foi verificada a leitura de entradas sem `last_used_at` e a serialização do novo campo.

***

## 📚 Conclusões e aprendizado

O projeto aprofundou meu entendimento sobre derivação de chaves, criptografia autenticada e persistência de credenciais.

Os desafios mostraram que o comportamento da ferramenta também participa da segurança: ocultar senhas, validar arquivos antes de alterar o cofre e controlar permissões complementam a proteção criptográfica.

Como melhoria futura, pretendo ampliar os testes dos novos comandos e estudar a avaliação de força de senhas.

***

## 🎬 Vídeo da demonstração

**Link:** (https://youtu.be/jOALEhWe9vs)
