# Guia Completo: rsync - Cópia, Sincronização e Backup de Arquivos no Linux
---

O **`rsync`** (remote sync) é uma ferramenta de linha de comando extremamente poderosa e versátil no Linux, usada para copiar e sincronizar arquivos e diretórios de forma eficiente. Sua principal vantagem é o algoritmo de "delta-transfer", que copia apenas as diferenças entre os arquivos, tornando-o ideal para backups incrementais, sincronização de dados e transferências de arquivos otimizadas, tanto localmente quanto remotamente.

## O que é o rsync?
---

`rsync` é um utilitário que permite a sincronização de arquivos entre duas localizações, sejam elas locais ou remotas (via SSH ou daemon rsync). Ele é projetado para minimizar a transferência de dados, o que o torna muito mais rápido que `cp` ou `scp` em muitas situações, especialmente quando apenas pequenas partes de arquivos grandes foram alteradas.

### Principais Características:

* **Algoritmo de Delta-Transfer:** Transfere apenas os blocos de arquivos que foram alterados, economizando largura de banda e tempo.
* **Eficiência:** Rápido e otimizado para grandes quantidades de dados ou arquivos muito grandes.
* **Flexibilidade:** Pode ser usado para cópias locais, sincronização remota (push/pull), e como cliente/servidor rsync.
* **Preservação de Atributos:** Capaz de preservar permissões, proprietários, grupos, timestamps e links simbólicos.
* **Resumível:** Se uma transferência for interrompida, pode ser retomada.
* **Exclusão de Arquivos:** Pode ser configurado para excluir arquivos do destino que não existem mais na origem.
* **Compressão:** Pode compactar dados durante a transferência para economizar largura de banda.

## Sintaxe Básica do `rsync`
---

A sintaxe geral do `rsync` é muito semelhante à de `cp` ou `scp`:

```bash
rsync [OPÇÕES] ORIGEM DESTINO
```

* **`ORIGEM`**: O arquivo ou diretório que você deseja copiar/sincronizar.
* **`DESTINO`**: O local para onde você deseja copiar/sincronizar os arquivos.

## Opções Comuns do `rsync`
---

O `rsync` possui muitas opções. Aqui estão as mais importantes e frequentemente usadas:

* **`-a` (archive):** Modo arquivo. Esta é a opção mais comum e **altamente recomendada** para a maioria dos backups e sincronizações. É uma combinação de outras opções: `-rlptgoD`.
    * `-r`: recursivo (copia diretórios e seus conteúdos).
    * `-l`: copia links simbólicos como links simbólicos.
    * `-p`: preserva permissões.
    * `-t`: preserva timestamps (hora de modificação).
    * `-g`: preserva grupo.
    * `-o`: preserva proprietário.
    * `-D`: preserva dispositivos e arquivos especiais.
* **`-v` (verbose):** Aumenta a verbosidade, mostrando os arquivos que estão sendo transferidos.
* **`-z` (compress):** Compacta os dados durante a transferência. Útil para conexões lentas.
* **`-h` (human-readable):** Formata os números de forma legível por humanos (ex: 1.5G em vez de 1500000000).
* **`--progress`:** Mostra o progresso da transferência de cada arquivo.
* **`--delete`:** Exclui arquivos no destino que não existem mais na origem. **Use com extrema cautela**, pois isso pode causar perda de dados se usado incorretamente.
* **`--exclude=PADRÃO`:** Exclui arquivos ou diretórios que correspondem a um padrão.
* **`--include=PADRÃO`:** Inclui arquivos ou diretórios que correspondem a um padrão (usado em conjunto com `--exclude`).
* **`--dry-run` ou `-n`:** Simula a operação sem fazer nenhuma alteração real. **Altamente recomendado** para testar comandos `rsync` complexos antes de executá-los de verdade.

## Exemplos de Uso do `rsync`
---

### 1. Copiar Arquivos e Diretórios Localmente

Como um `cp` melhorado, preservando atributos:

```bash
rsync -avh --progress /home/user/Documents/ /mnt/backup/Documents/
```
* Copia o conteúdo de `Documents/` para `/mnt/backup/Documents/`.
* A barra final `/` em `Documents/` na origem é importante: se você omitir, ele criará `/mnt/backup/Documents/Documents/`.

### 2. Sincronizar Diretórios Localmente (Backup Incremental)

Mantém o destino idêntico à origem:

```bash
rsync -avh --delete --progress /home/user/MyData/ /mnt/backup/MyData/
```
* Qualquer arquivo deletado em `MyData/` na origem será deletado em `/mnt/backup/MyData/`.
* Apenas arquivos novos ou modificados serão copiados.

### 3. Sincronizar com um Servidor Remoto (via SSH)

`rsync` usa SSH por padrão para transferências remotas, garantindo segurança.

* **Copiar arquivos para um servidor remoto:**
    ```bash
    rsync -avzh --progress /home/user/MyProject/ user@remoteserver:/var/www/html/MyProject/
    ```
    * `-z`: Compacta a transferência (útil em redes lentas).
    * `user@remoteserver`: Nome de usuário e endereço IP/hostname do servidor remoto.
    * Você será solicitado pela senha SSH.

* **Puxar arquivos de um servidor remoto para o local:**
    ```bash
    rsync -avzh --progress user@remoteserver:/var/www/html/MyBackup/ /home/user/local_backup/
    ```

### 4. Excluir Arquivos/Diretórios Específicos

```bash
# Excluir a pasta 'cache' e todos os arquivos '.tmp'
rsync -avh --progress --exclude 'cache/' --exclude '*.tmp' /var/www/html/ /mnt/backup/web_data/
```

### 5. Fazer Backup com Sincronização Diária (Cron Job)

Você pode agendar o `rsync` para fazer backups automaticamente.

1.  **Crie um script de backup (ex: `daily_backup.sh`):**
    ```bash
    #!/bin/bash
    LOG_FILE="/var/log/backup_mysite.log"
    SOURCE_DIR="/var/www/html/mysite/"
    DEST_DIR="/mnt/backup/web_data/"

    echo "--- Backup iniciado em $(date) ---" >> $LOG_FILE
    rsync -avh --delete --stats --log-file=$LOG_FILE $SOURCE_DIR $DEST_DIR
    echo "--- Backup finalizado em $(date) ---" >> $LOG_FILE
    echo " " >> $LOG_FILE
    ```
    * `--stats`: Mostra um resumo das estatísticas de transferência.
    * `--log-file`: Salva a saída do rsync em um arquivo de log.

2.  **Torne o script executável:**
    ```bash
    chmod +x daily_backup.sh
    ```

3.  **Adicione-o ao `cron` (como root para backups de sistema):**
    ```bash
    sudo crontab -e
    ```
    Adicione a seguinte linha para executar o backup todo dia à 01:00 AM:
    ```cron
    0 1 * * * /path/to/your/daily_backup.sh
    ```

## Considerações Finais
---

* **Atenção à barra final (`/`):** A presença ou ausência da barra final na **ORIGEM** é crucial.
    * `rsync -a /origem/pasta` /destino` (copia a pasta `pasta` para dentro de `/destino`, resultando em `/destino/pasta`).
    * `rsync -a /origem/pasta/` /destino` (copia o *conteúdo* de `pasta` para dentro de `/destino`, resultando em arquivos diretamente em `/destino`).
* **Modo `dry-run`:** Sempre use `--dry-run` (`-n`) para testar seus comandos `rsync` antes de executá-los, especialmente quando usando `--delete`.
* **Backup Remoto com Chaves SSH:** Para automação de backups remotos, configure o SSH com autenticação por chave (sem senha) para evitar a necessidade de digitar a senha a cada execução.
* **Permissões:** `rsync -a` é geralmente suficiente para preservar permissões. Se precisar de controle mais fino, ajuste as opções `chmod`, `chown`, `chgrp`.

`rsync` é uma ferramenta indispensável para qualquer administrador de sistema ou usuário avançado de Linux, proporcionando um controle eficiente e flexível sobre a cópia e sincronização de arquivos.