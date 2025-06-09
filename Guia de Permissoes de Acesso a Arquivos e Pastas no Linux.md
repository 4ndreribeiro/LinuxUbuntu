# Guia Completo: Permissões de Acesso a Arquivos e Pastas pelo Terminal Linux
---

No Linux, o sistema de permissões de arquivos e pastas é uma parte fundamental da segurança e do controle de acesso. Ele determina quem pode ler, escrever ou executar um arquivo ou diretório. Compreender e gerenciar essas permissões é essencial para qualquer usuário ou administrador de sistema Linux.

## O que são Permissões de Arquivo no Linux?
---

Cada arquivo e diretório no Linux possui um conjunto de permissões que especifica os direitos de acesso para três categorias de usuários:

1.  **Proprietário (Owner):** O usuário que possui o arquivo ou diretório.
2.  **Grupo (Group):** O grupo ao qual o arquivo ou diretório pertence.
3.  **Outros (Others):** Todos os outros usuários no sistema que não são o proprietário nem membros do grupo.

Para cada uma dessas categorias, existem três tipos de permissões:

* **Leitura (`r` - Read):** Permite visualizar o conteúdo de um arquivo ou listar o conteúdo de um diretório.
* **Escrita (`w` - Write):** Permite modificar ou excluir um arquivo, ou criar, excluir e renomear arquivos dentro de um diretório.
* **Execução (`x` - Execute):** Permite executar um arquivo (se for um programa ou script) ou entrar (acessar) um diretório.

## Visualizando Permissões (`ls -l`)
---

O comando `ls -l` é usado para listar arquivos e diretórios em formato longo, mostrando suas permissões, proprietário, grupo, tamanho, data de modificação e nome.

**Exemplo de saída:**

```
drwxr-xr-x 2 user1 group1 4096 Jan  1 10:00 my_directory/
-rw-r--r-- 1 user1 group1 1024 Jan  1 10:05 my_file.txt
```

### Análise das Permissões:

A primeira coluna (ex: `drwxr-xr-x`) é a string de permissões, que consiste em 10 caracteres:

1.  **Primeiro Caractere:** Indica o tipo de arquivo:
    * `-`: Arquivo comum
    * `d`: Diretório
    * `l`: Link simbólico
    * `b`: Dispositivo de bloco
    * `c`: Dispositivo de caractere
    * `p`: Named pipe (FIFO)
    * `s`: Socket

2.  **Próximos 9 Caracteres:** São divididos em três grupos de três, representando as permissões para:
    * **Proprietário:** (caracteres 2, 3, 4) - `rwx` no exemplo.
    * **Grupo:** (caracteres 5, 6, 7) - `r-x` no exemplo.
    * **Outros:** (caracteres 8, 9, 10) - `r-x` no exemplo.

    * `r`: Leitura
    * `w`: Escrita
    * `x`: Execução
    * `-`: Permissão ausente

**Exemplo:**
* `drwxr-xr-x`:
    * `d`: É um diretório.
    * `rwx`: Proprietário tem leitura, escrita e execução.
    * `r-x`: Membros do grupo têm leitura e execução, mas não escrita.
    * `r-x`: Outros usuários têm leitura e execução, mas não escrita.

## Alterando Permissões (`chmod`)
---

O comando `chmod` (change mode) é usado para alterar as permissões de arquivos e diretórios. Ele pode ser usado em dois modos: simbólico (letras) ou numérico (octal).

### 1. Modo Simbólico (Letras)

Usa uma combinação de letras para adicionar (`+`), remover (`-`) ou definir (`=`) permissões.

* **Categorias:** `u` (proprietário), `g` (grupo), `o` (outros), `a` (todos).
* **Permissões:** `r` (leitura), `w` (escrita), `x` (execução).

**Exemplos:**
* **Adicionar permissão de execução ao proprietário:**
    ```bash
    chmod u+x my_script.sh
    ```
* **Remover permissão de escrita de outros:**
    ```bash
    chmod o-w my_file.txt
    ```
* **Adicionar leitura e escrita para o grupo:**
    ```bash
    chmod g+rw my_file.txt
    ```
* **Definir permissões específicas (somente leitura para todos):**
    ```bash
    chmod a=r my_file.txt
    ```
* **Permitir leitura, escrita e execução para o proprietário; leitura e execução para grupo e outros (padrão para diretórios):**
    ```bash
    chmod u=rwx,go=rx my_directory/
    ```

### 2. Modo Numérico (Octal)

É o método mais comum e poderoso, usando números octais para representar as permissões. Cada permissão tem um valor numérico:

* **r (leitura) = 4**
* **w (escrita) = 2**
* **x (execução) = 1**
* **- (nenhuma) = 0**

As permissões para cada categoria (proprietário, grupo, outros) são somadas para formar um dígito octal de 0 a 7.

| Permissão | Valor Octal |
| :-------- | :---------- |
| `---`     | 0           |
| `--x`     | 1           |
| `-w-`     | 2           |
| `-wx`     | 3           |
| `r--`     | 4           |
| `r-x`     | 5           |
| `rw-`     | 6           |
| `rwx`     | 7           |

Um comando `chmod` em modo numérico usa três dígitos: `<proprietário><grupo><outros>`.

**Exemplos:**

* **`755` (rwx r-x r-x):** Comum para diretórios e scripts executáveis.
    * Proprietário: 4+2+1 = 7
    * Grupo: 4+0+1 = 5
    * Outros: 4+0+1 = 5
    ```bash
    chmod 755 my_script.sh
    chmod 755 my_directory/
    ```
* **`644` (rw- r-- r--):** Comum para arquivos de texto.
    * Proprietário: 4+2+0 = 6
    * Grupo: 4+0+0 = 4
    * Outros: 4+0+0 = 4
    ```bash
    chmod 644 my_file.txt
    ```
* **`777` (rwx rwx rwx):** Permissões totais para todos (cuidado, pode ser um risco de segurança!).
    ```bash
    chmod 777 public_share/
    ```

### Opção Recursiva (`-R`)

Para aplicar permissões a um diretório e todo o seu conteúdo (subdiretórios e arquivos), use a opção `-R` (recursivo).

```bash
chmod -R 755 my_project_folder/
```

## Alterando Proprietário e Grupo (`chown`, `chgrp`)
---

Além das permissões, você pode alterar o proprietário e o grupo de um arquivo ou diretório.

### 1. Alterar Proprietário (`chown`)

O comando `chown` (change owner) é usado para mudar o proprietário de um arquivo ou diretório. Requer privilégios de root.

```bash
sudo chown new_owner my_file.txt
sudo chown -R new_owner my_directory/ # Recursivo
```

### 2. Alterar Grupo (`chgrp`)

O comando `chgrp` (change group) é usado para mudar o grupo de um arquivo ou diretório.

```bash
sudo chgrp new_group my_file.txt
sudo chgrp -R new_group my_directory/ # Recursivo
```

### 3. Alterar Proprietário e Grupo Simultaneamente (`chown user:group`)

Você pode usar `chown` para alterar ambos proprietário e grupo ao mesmo tempo, separando-os por dois pontos (`:`).

```bash
sudo chown new_owner:new_group my_file.txt
sudo chown -R new_owner:new_group my_directory/ # Recursivo
```

## Permissões Especiais
---

Além das permissões `rwx` básicas, existem algumas permissões especiais que afetam o comportamento de arquivos e diretórios:

* **SetUID (Set User ID - `s` no lugar de `x` do proprietário):**
    * Em um arquivo executável, faz com que ele seja executado com as permissões do proprietário do arquivo, não do usuário que o executa.
    * Ex: `passwd` (permite que usuários comuns alterem suas senhas, embora o arquivo `passwd` seja de root).
    * Numérico: Adicione `4` antes dos três dígitos. Ex: `chmod 4755 my_program`

* **SetGID (Set Group ID - `s` no lugar de `x` do grupo):**
    * Em um arquivo executável, faz com que ele seja executado com as permissões do grupo do arquivo.
    * Em um diretório, faz com que os novos arquivos criados dentro dele herdem o grupo do diretório, não o grupo primário do usuário que o criou.
    * Numérico: Adicione `2` antes dos três dígitos. Ex: `chmod 2775 shared_folder`

* **Sticky Bit (`t` no lugar de `x` de outros):**
    * Aplicado a diretórios, impede que usuários excluam ou renomeiem arquivos que não são de sua propriedade, mesmo que tenham permissão de escrita no diretório.
    * Comum em diretórios como `/tmp`.
    * Numérico: Adicione `1` antes dos três dígitos. Ex: `chmod 1777 /tmp`

**Exemplo de visualização de permissões especiais:**

```
-rwsr-xr-x 1 root root 61608 May 11 2023 /usr/bin/passwd (SetUID)
drwxrwsr-x 2 user1 shared_group 4096 Jan  1 10:00 my_shared_folder/ (SetGID em diretório)
drwxrwxrwt 2 root root 4096 Apr 20 15:30 /tmp (Sticky Bit)
```

Compreender e aplicar corretamente as permissões de acesso é um pilar da segurança e da administração eficaz de sistemas Linux.
