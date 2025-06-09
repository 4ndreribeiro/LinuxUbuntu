# Guia Completo: FTP no Linux - Servidores e Clientes
---

O **FTP** (File Transfer Protocol) é um protocolo padrão de rede usado para a transferência de arquivos entre um cliente e um servidor em uma rede de computadores. Embora tecnologias mais seguras como SFTP e SCP sejam frequentemente preferidas hoje em dia, o FTP ainda é amplamente utilizado em muitos cenários.

Este guia aborda como configurar um servidor FTP (vsftpd) e como usar clientes FTP no Linux.

## O que é FTP?
---

O FTP opera em um modelo cliente-servidor, onde o cliente inicia uma conexão com o servidor FTP para enviar ou receber arquivos. Ele usa duas portas TCP:

* **Porta 21 (Porta de Comando):** Usada para enviar comandos do cliente para o servidor e receber respostas (autenticação, listagem de diretórios, etc.).
* **Porta 20 (Porta de Dados):** Usada para a transferência de dados reais. Esta porta é usada no modo ativo de conexão. No modo passivo, o cliente e o servidor negociam portas aleatórias para a transferência de dados.

### Modos de Conexão:

* **Modo Ativo:** O cliente envia a porta de dados que ele abrirá para o servidor, e o servidor se conecta a essa porta. Pode ter problemas com firewalls no lado do cliente.
* **Modo Passivo:** O cliente solicita ao servidor uma porta para conexão de dados. O servidor abre uma porta e informa ao cliente, que então se conecta a ela. Mais amigável para firewalls.

## Configurando um Servidor FTP (vsftpd) no Linux
---

O **vsftpd** (Very Secure FTP Daemon) é um servidor FTP leve, estável e seguro, amplamente utilizado em distribuições Linux.

### 1. Instalação do vsftpd

No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install vsftpd
```

No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install vsftpd
```

### 2. Configuração do vsftpd

O principal arquivo de configuração do vsftpd é `/etc/vsftpd.conf`. Sempre faça um backup antes de editar.

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
sudo nano /etc/vsftpd.conf
```

A seguir, algumas configurações comuns e seus significados. Descomente (`#` remova) ou modifique as linhas conforme necessário:

* **Permitir usuários anônimos (desativado por padrão, não recomendado para segurança):**
    ```
    anonymous_enable=YES
    ```
* **Permitir upload de arquivos por usuários anônimos (APENAS se `anonymous_enable=YES`):**
    ```
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    ```
* **Permitir login local de usuários do sistema:**
    ```
    local_enable=YES
    ```
* **Permitir operações de escrita (upload, criação de pastas) para usuários locais:**
    ```
    write_enable=YES
    ```
* **Restringir usuários ao seu diretório home (chroot jail - Altamente recomendado para segurança):**
    ```
    chroot_local_user=YES
    ```
    * **Importante:** Se você estiver com problemas ao usar `chroot_local_user=YES` (ex: erro `500 OOPS: vsftpd: refusing to run with writable root inside chroot()`), você pode precisar adicionar:
        ```
        allow_writeable_chroot=YES
        ```
        Ou garantir que o diretório home dos usuários (onde eles são chrootados) não seja gravável por eles. Uma alternativa é criar um subdiretório gravável dentro do home do usuário.

* **Desativar o modo ativo (geralmente melhor usar passivo):**
    ```
    connect_from_port_20=NO
    ```
* **Definir um intervalo de portas para o modo passivo (essencial para firewalls):**
    ```
    pasv_enable=YES
    pasv_min_port=40000
    pasv_max_port=50000
    ```
    (Ajuste o intervalo conforme sua necessidade, mas mantenha-o suficientemente grande).

* **Configurar banner de boas-vindas:**
    ```
    ftpd_banner=Bem-vindo ao meu servidor FTP!
    ```
* **Configurações de segurança adicionais:**
    * Para listar usuários que NÃO podem fazer login:
        ```
        userlist_enable=YES
        userlist_deny=YES
        userlist_file=/etc/vsftpd.deny_users
        ```
        Crie o arquivo `/etc/vsftpd.deny_users` e adicione os nomes de usuário linha por linha.
    * Para listar usuários que SÓ podem fazer login:
        ```
        userlist_enable=YES
        userlist_deny=NO
        userlist_file=/etc/vsftpd.allow_users
        ```
        Crie o arquivo `/etc/vsftpd.allow_users` e adicione os nomes de usuário permitidos linha por linha.

Após as modificações, salve o arquivo e reinicie o serviço vsftpd:

```bash
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd # Para iniciar automaticamente na inicialização
sudo systemctl status vsftpd # Verificar o status
```

### 3. Configuração de Firewall (UFW/Firewalld)

Você precisa abrir as portas FTP no firewall do seu Linux.

#### Usando UFW (Ubuntu/Debian):

```bash
sudo ufw allow 20/tcp   # Porta de dados (modo ativo)
sudo ufw allow 21/tcp   # Porta de comando
sudo ufw allow 40000:50000/tcp # Portas para o modo passivo (intervalo definido no vsftpd.conf)
sudo ufw enable         # Habilitar UFW se não estiver ativo
sudo ufw status verbose # Verificar regras
```

#### Usando Firewalld (Fedora/CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-port=20/tcp
sudo firewall-cmd --permanent --add-port=21/tcp
sudo firewall-cmd --permanent --add-port=40000-50000/tcp # Portas para o modo passivo
sudo firewall-cmd --permanent --add-service=ftp        # Adiciona regras FTP padrão
sudo firewall-cmd --reload                              # Recarregar firewall
sudo firewall-cmd --list-all                            # Verificar regras
```

### 4. Gerenciando Usuários FTP

Usuários locais do sistema podem fazer login no FTP se `local_enable=YES` e `userlist_enable` estiver configurado para permiti-los.

* Para criar um novo usuário para FTP (se você não quiser usar um usuário existente):
    ```bash
    sudo adduser nome_usuario_ftp
    sudo passwd nome_usuario_ftp # Definir senha
    ```
* Para desabilitar o shell para um usuário (se ele for apenas para FTP):
    ```bash
    sudo usermod -s /usr/sbin/nologin nome_usuario_ftp
    ```
    (Note: `nologin` impede o login via SSH, mas não afeta o FTP.)

## Usando Clientes FTP no Linux
---

Existem várias maneiras de acessar um servidor FTP a partir do Linux:

### 1. Cliente de Linha de Comando (ftp)

O cliente `ftp` é um utilitário básico de linha de comando.

```bash
ftp <endereco_ip_ou_hostname_do_servidor>
# Exemplo: ftp 192.168.1.100 ou ftp meuserver.com
```

Após conectar, você será solicitado por `Username` e `Password`.
Comandos úteis dentro do cliente `ftp`:
* `ls`: Listar arquivos no diretório atual do servidor.
* `cd <diretorio>`: Mudar de diretório no servidor.
* `lcd <diretorio_local>`: Mudar de diretório local.
* `get <arquivo_remoto>`: Baixar um arquivo do servidor.
* `put <arquivo_local>`: Enviar um arquivo para o servidor.
* `mget <arquivos>`: Baixar múltiplos arquivos.
* `mput <arquivos>`: Enviar múltiplos arquivos.
* `binary`: Mudar para modo de transferência binária (para arquivos não-texto).
* `ascii`: Mudar para modo de transferência ASCII (para arquivos de texto).
* `bye` ou `quit`: Sair do cliente FTP.

### 2. Cliente Gráfico (FileZilla)

O FileZilla é um cliente FTP/FTPS/SFTP multiplataforma popular e fácil de usar.

Instalação (Ubuntu/Debian):

```bash
sudo apt install filezilla
```

Uso:
1.  Abra o FileZilla.
2.  Na barra de Conexão Rápida (Quickconnect), insira:
    * **Host:** `ftp://<endereco_ip_ou_hostname>` (ou deixe sem `ftp://` para detecção automática)
    * **Username:** Nome de usuário FTP
    * **Password:** Senha FTP
    * **Port:** `21` (ou a porta de comando personalizada, se houver)
3.  Clique em "Quickconnect".
4.  A interface mostrará seus arquivos locais à esquerda e os arquivos do servidor à direita. Você pode arrastar e soltar arquivos para transferir.

### 3. Usando `wget` ou `curl` (para downloads/uploads simples)

Para downloads de arquivos via FTP sem interface interativa:

```bash
# Download
wget ftp://username:password@server_ip/caminho/do/arquivo.txt

# Upload (com curl)
curl -u username:password -T /caminho/local/arquivo_para_upload.txt ftp://server_ip/caminho/remoto/
```

O FTP continua sendo uma ferramenta útil para transferência de arquivos, especialmente em redes locais ou quando a segurança não é a principal preocupação. Para maior segurança, considere usar SFTP ou SCP.
