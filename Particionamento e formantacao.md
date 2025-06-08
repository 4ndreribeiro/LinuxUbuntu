# Guia Completo: Particionamento, Formatação e Montagem de Discos no Linux
---

Gerenciar discos no Linux envolve três etapas principais: **particionamento**, **formatação** e **montagem**. Este guia detalha cada uma dessas etapas, fornecendo os comandos essenciais para você configurar novos discos ou gerenciar os existentes no seu sistema Linux.

## 1. Particionamento de Disco
---

O particionamento divide um disco físico em uma ou mais seções lógicas, chamadas partições. Cada partição pode ser tratada como um dispositivo de armazenamento independente. As ferramentas mais comuns para particionamento são `fdisk` (para tabelas de partição MBR) e `gdisk` (para tabelas de partição GPT, que são mais modernas e recomendadas para discos maiores que 2TB e sistemas mais novos).

**Cuidado:** Operações de particionamento podem resultar em perda de dados se não forem feitas corretamente. Certifique-se de fazer um backup de dados importantes antes de prosseguir.

### 1.1. Identificar o Disco

Primeiro, você precisa identificar o nome do disco que deseja particionar. Use o comando `lsblk` ou `fdisk -l`.

```bash
sudo lsblk
# ou
sudo fdisk -l
```

A saída mostrará seus discos (ex: `/dev/sda`, `/dev/sdb`, `/dev/nvme0n1`).

### 1.2. Usando `fdisk` (para MBR - Legado)

`fdisk` é usado para discos com tabelas de partição **MBR (Master Boot Record)**.

```bash
sudo fdisk /dev/sdb # Substitua /dev/sdb pelo seu disco
```

Dentro do `fdisk`, você pode usar os seguintes comandos:
* `m`: Mostra o menu de ajuda.
* `p`: Imprime a tabela de partições atual.
* `n`: Cria uma nova partição.
* `d`: Exclui uma partição.
* `w`: Salva as alterações e sai.
* `q`: Sai sem salvar as alterações.

**Exemplo de criação de partição primária:**
1.  Digite `n` para nova partição.
2.  Escolha `p` para partição primária (ou `e` para estendida).
3.  Pressione `Enter` para o número da partição (padrão é o próximo disponível).
4.  Pressione `Enter` para o primeiro setor (padrão é o início do espaço livre).
5.  Especifique o tamanho da partição (ex: `+10G` para 10GB, `+100M` para 100MB, ou pressione `Enter` para usar todo o espaço restante).
6.  Digite `w` para salvar e sair.

### 1.3. Usando `gdisk` (para GPT - Moderno)

`gdisk` é recomendado para discos com tabelas de partição **GPT (GUID Partition Table)**.

```bash
sudo gdisk /dev/sdb # Substitua /dev/sdb pelo seu disco
```

Dentro do `gdisk`, os comandos são semelhantes ao `fdisk`:
* `?`: Mostra o menu de ajuda.
* `p`: Imprime a tabela de partições atual.
* `n`: Cria uma nova partição.
* `d`: Exclui uma partição.
* `w`: Salva as alterações e sai.
* `q`: Sai sem salvar as alterações.

**Exemplo de criação de partição:**
1.  Digite `n` para nova partição.
2.  Pressione `Enter` para o número da partição (padrão é o próximo disponível).
3.  Pressione `Enter` para o primeiro setor (padrão é o início do espaço livre).
4.  Especifique o tamanho da partição (ex: `+10G`, `+100M`, ou `Enter` para usar todo o espaço restante).
5.  Pressione `Enter` para o tipo de partição (padrão é Linux filesystem).
6.  Digite `w` para salvar e sair.

## 2. Formatação de Disco
---

Após particionar o disco, você precisa formatar as partições com um sistema de arquivos. O sistema de arquivos organiza os dados na partição, tornando-a utilizável pelo sistema operacional.

### 2.1. Criar Sistema de Arquivos

Use o comando `mkfs` (make filesystem) para formatar a partição. Os sistemas de arquivos mais comuns no Linux são `ext4` (o padrão), `xfs` e `btrfs`.

```bash
# Para ext4 (comum para partições de dados e sistema)
sudo mkfs.ext4 /dev/sdb1 # Substitua /dev/sdb1 pela sua nova partição

# Para xfs (bom para grandes arquivos e servidores)
sudo mkfs.xfs /dev/sdb1

# Para btrfs (moderno, com recursos como snapshots)
sudo mkfs.btrfs /dev/sdb1
```

## 3. Montagem de Disco
---

Para que o sistema Linux possa acessar os dados em uma partição formatada, ela precisa ser "montada" em um diretório dentro da hierarquia do sistema de arquivos.

### 3.1. Montagem Temporária

Você pode montar uma partição manualmente para acesso imediato.

1.  **Crie um ponto de montagem (um diretório vazio):**
    ```bash
    sudo mkdir /mnt/meudisco
    ```
2.  **Monte a partição:**
    ```bash
    sudo mount /dev/sdb1 /mnt/meudisco # Substitua pela sua partição e ponto de montagem
    ```
Agora, o conteúdo da partição `/dev/sdb1` estará acessível em `/mnt/meudisco`.

### 3.2. Desmontar uma Partição

Para desmontar uma partição, use `umount`. **Certifique-se de não estar dentro do diretório montado ao tentar desmontá-lo.**

```bash
sudo umount /mnt/meudisco
```

### 3.3. Montagem Persistente (com `/etc/fstab`)

Para que a partição seja montada automaticamente a cada inicialização do sistema, você precisa adicioná-la ao arquivo `/etc/fstab`.

1.  **Obtenha o UUID da sua partição:** O UUID (Universally Unique Identifier) é a maneira mais segura de referenciar uma partição, pois não muda se o nome do dispositivo (`/dev/sdb1`) for alterado.
    ```bash
    sudo blkid /dev/sdb1 # Substitua pela sua partição
    ```
    Anote o UUID. Exemplo: `UUID="a1b2c3d4-e5f6-7890-1234-56789abcdef0"`

2.  **Edite o arquivo `/etc/fstab`:**
    ```bash
    sudo nano /etc/fstab # Use seu editor de texto preferido
    ```
3.  **Adicione uma nova linha no final do arquivo no seguinte formato:**

    ```
    UUID=<seu_UUID> <ponto_de_montagem> <tipo_filesystem> <opcoes> <dump> <pass>
    ```

    **Exemplo para uma partição `ext4`:**
    ```
    UUID=a1b2c3d4-e5f6-7890-1234-56789abcdef0 /mnt/meudisco ext4 defaults 0 2
    ```
    * **`<seu_UUID>`:** O UUID obtido com `blkid`.
    * **`<ponto_de_montagem>`:** O diretório onde a partição será montada (ex: `/mnt/meudisco`).
    * **`<tipo_filesystem>`:** O tipo de sistema de arquivos (ex: `ext4`, `xfs`).
    * **`<opcoes>`:** Opções de montagem (ex: `defaults` é um bom ponto de partida e inclui `rw, suid, dev, exec, auto, nouser, async`). Outras opções comuns: `noatime` (melhora o desempenho), `nofail` (o sistema inicializa mesmo se a partição não montar).
    * **`<dump>`:** `0` (não fazer backup com `dump`).
    * **`<pass>`:** `0` (não verificar a integridade do sistema de arquivos na inicialização), `1` (partição root), `2` (outras partições). Para partições de dados, geralmente use `2`.

4.  **Salve o arquivo e saia do editor.**

5.  **Teste a nova entrada no `fstab`:** Para garantir que não há erros antes de reiniciar, tente montar todas as entradas do `fstab`.

    ```bash
    sudo mount -a
    ```
    Se não houver erros, a partição foi montada com sucesso e será montada automaticamente na próxima inicialização.

## Exemplo de Fluxo de Trabalho Completo
---

Vamos imaginar que você conectou um novo disco e ele apareceu como `/dev/sdb`. Você quer criar uma única partição `ext4` de 50GB e montá-la permanentemente em `/data`.

1.  **Particionar (usando `gdisk` para GPT):**
    ```bash
    sudo gdisk /dev/sdb
    # Dentro do gdisk:
    # n (nova partição)
    # <Enter> (número da partição padrão)
    # <Enter> (primeiro setor padrão)
    # +50G (tamanho de 50GB)
    # <Enter> (tipo de partição padrão)
    # w (salvar e sair)
    # y (confirmar)
    ```
2.  **Formatar a nova partição:**
    ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```
3.  **Criar ponto de montagem:**
    ```bash
    sudo mkdir /data
    ```
4.  **Obter UUID:**
    ```bash
    sudo blkid /dev/sdb1
    # Exemplo de saída: /dev/sdb1: UUID="1234abcd-efgh-5678-ijkl-9012mnopqrst" TYPE="ext4"
    ```
5.  **Adicionar ao `/etc/fstab`:**
    ```bash
    sudo nano /etc/fstab
    # Adicione a linha (substituindo o UUID):
    # UUID=1234abcd-efgh-5678-ijkl-9012mnopqrst /data ext4 defaults 0 2
    ```
6.  **Testar montagem:**
    ```bash
    sudo mount -a
    ```
7.  **Verificar se está montado:**
    ```bash
    df -h /data
    ```

Com estas etapas, você pode gerenciar com eficácia o armazenamento de discos no seu sistema Linux.
