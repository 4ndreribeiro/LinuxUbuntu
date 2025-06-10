# Guia Completo: Servidor de Arquivos com Samba no Linux
---

O **Samba** é uma implementação de software livre do protocolo **SMB/CIFS** (Server Message Block/Common Internet File System), que permite que sistemas Linux e Unix compartilhem arquivos e impressoras com sistemas Windows e outros dispositivos que suportam SMB. É uma ferramenta essencial para a interoperabilidade em redes mistas.

## O que é Samba?
---

Em termos simples, o Samba permite que seu servidor Linux funcione como um servidor de arquivos compatível com o Windows. Isso significa que você pode configurar pastas compartilhadas no seu sistema Linux e acessá-las facilmente a partir de computadores Windows (ou até mesmo outros Linux, macOS, etc.) como se fossem pastas compartilhadas em uma rede Windows.

### Principais Casos de Uso:

* **Compartilhamento de Arquivos:** Principalmente usado para criar pastas compartilhadas na rede para que vários usuários possam acessar e colaborar em documentos.
* **Servidor de Impressão:** Compartilhar impressoras conectadas ao servidor Linux com clientes na rede.
* **Autenticação:** Pode atuar como um controlador de domínio primário (PDC) ou controlador de domínio de backup (BDC) em ambientes Windows (embora o Active Directory tenha superado em grande parte esse uso).
* **Integração com Active Directory:** Pode se juntar a um domínio Active Directory existente.

## Configurando um Servidor Samba no Linux
---

Vamos configurar um compartilhamento de arquivos básico usando o `Samba`.

### 1. Instalação do Samba

No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install samba samba-common-bin
```

No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install samba samba-client samba-common
```

Após a instalação, os serviços necessários serão iniciados.

### 2. Criação do Diretório Compartilhado

Crie um diretório que você deseja compartilhar. É uma boa prática definir permissões adequadas para este diretório.

```bash
sudo mkdir -p /srv/samba/meus_arquivos_compartilhados
sudo chown -R nobody:nogroup /srv/samba/meus_arquivos_compartilhados
sudo chmod -R 777 /srv/samba/meus_arquivos_compartilhados
```
* `nobody:nogroup`: Define o proprietário como um usuário/grupo sem privilégios para o Samba lidar com as permissões de acesso através de suas próprias regras.
* `777`: Permissões amplas para testes iniciais. Você deve ajustar isso depois para algo mais restritivo, como `770` com um grupo específico, para maior segurança.

### 3. Configuração do Samba (`smb.conf`)

O principal arquivo de configuração do Samba é `/etc/samba/smb.conf`. **Sempre faça um backup antes de editar.**

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo nano /etc/samba/smb.conf
```

Role para o final do arquivo ou adicione a seguinte seção para o seu compartilhamento:

```ini
[global]
   workgroup = WORKGROUP        ; Substitua pelo seu grupo de trabalho da rede
   security = user              ; Modo de segurança: 'user' é o mais comum e seguro

[meus_compartilhados]
   comment = Meus arquivos de projeto
   path = /srv/samba/meus_arquivos_compartilhados
   browsable = yes              ; Permite que o compartilhamento seja visível na rede
   read only = no               ; Permite escrita no compartilhamento
   create mask = 0755           ; Permissões para novos arquivos
   directory mask = 0755        ; Permissões para novos diretórios
   valid users = @samba_users   ; Limita acesso a membros do grupo 'samba_users' (criaremos abaixo)
   writable = yes               ; Permite que usuários válidos escrevam
   guest ok = no                ; Não permite acesso de convidado (anônimo)
```

**Explicação das Opções:**

* **`[global]`**: Seção para configurações globais do servidor Samba.
    * `workgroup`: O nome do grupo de trabalho da sua rede Windows.
    * `security = user`: Os usuários devem se autenticar com um nome de usuário e senha do Samba.

* **`[meus_compartilhados]`**: O nome da sua pasta compartilhada (será visível na rede).
    * `comment`: Uma descrição para o compartilhamento.
    * `path`: O caminho absoluto para o diretório que você está compartilhando no seu sistema Linux.
    * `browsable = yes`: Permite que o compartilhamento seja listado quando os usuários navegam pela rede.
    * `read only = no`: Permite que usuários escrevam no compartilhamento (se tiverem as permissões de escrita).
    * `create mask` / `directory mask`: Definem as permissões padrão para arquivos e diretórios criados no compartilhamento. `0755` significa que o proprietário tem controle total, e o grupo/outros têm leitura e execução.
    * `valid users = @samba_users`: **Altamente recomendado.** Restringe o acesso a usuários que são membros do grupo de sistema `samba_users`. Vamos criar este grupo e adicionar usuários a ele.
    * `writable = yes`: Permite operações de escrita.
    * `guest ok = no`: Desativa o acesso anônimo para este compartilhamento.

Após as modificações, salve o arquivo e reinicie o serviço Samba:

```bash
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd # Para iniciar automaticamente
sudo systemctl status smbd nmbd
```

### 4. Criar Usuários Samba

Usuários que acessarão o compartilhamento Samba precisam existir como usuários no sistema Linux e ter uma senha Samba.

1.  **Criar um grupo para usuários Samba (se estiver usando `valid users = @samba_users`):**
    ```bash
    sudo groupadd samba_users
    ```

2.  **Criar um usuário Linux (se ainda não existir) e adicioná-lo ao grupo `samba_users`:**
    ```bash
    sudo useradd -M -s /usr/sbin/nologin sambauser1 ; Não cria home, não permite login shell
    sudo usermod -aG samba_users sambauser1 ; Adiciona ao grupo samba_users
    ```
    * `-M`: Não cria um diretório home para o usuário (útil se o usuário só vai acessar Samba).
    * `-s /usr/sbin/nologin`: Impede que o usuário faça login via SSH ou terminal (aumenta a segurança para usuários apenas de Samba).

3.  **Adicionar o usuário ao banco de dados de senhas do Samba:**
    ```bash
    sudo smbpasswd -a sambauser1
    ```
    Você será solicitado a digitar uma senha para o usuário Samba. **Esta senha pode ser diferente da senha do sistema Linux.**

    * Para desabilitar um usuário Samba: `sudo smbpasswd -d sambauser1`
    * Para habilitar um usuário Samba: `sudo smbpasswd -e sambauser1`
    * Para excluir um usuário Samba: `sudo smbpasswd -x sambauser1`

### 5. Configuração de Firewall

Você precisa abrir as portas do Samba no firewall do seu Linux.

#### Usando UFW (Ubuntu/Debian):

```bash
sudo ufw allow samba
sudo ufw enable         # Habilitar UFW se não estiver ativo
sudo ufw status verbose # Verificar regras
```
* `sudo ufw allow samba` abrirá as portas TCP/UDP 137, 138, 139 e 445, que são as portas padrão para Samba.

#### Usando Firewalld (Fedora/CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload                              # Recarregar firewall
sudo firewall-cmd --list-all                            # Verificar regras
```

## Acessando Compartilhamentos Samba de um Cliente Linux
---

Você pode acessar compartilhamentos Samba de um cliente Linux usando o gerenciador de arquivos ou a linha de comando.

### 1. Via Gerenciador de Arquivos (GNOME Files/Nautilus, Dolphin, Thunar)

1.  Abra seu gerenciador de arquivos.
2.  Geralmente, há uma opção "Outros Locais" ou "Navegar na Rede".
3.  No campo de endereço, digite: `smb://<endereco_ip_ou_hostname_do_servidor>/<nome_do_compartilhamento>`
    * Exemplo: `smb://192.168.1.100/meus_compartilhados`
4.  Você será solicitado a digitar o nome de usuário e a senha do Samba.

### 2. Via Linha de Comando (`smbclient` ou `mount`)

#### Usando `smbclient` (Cliente de linha de comando para Samba):

```bash
sudo apt install smbclient # Instalar se necessário

smbclient //192.168.1.100/meus_compartilhados -U sambauser1
```
Você será solicitado pela senha do usuário Samba. Uma vez conectado, você pode usar comandos como `ls`, `cd`, `get`, `put`.

#### Montando o Compartilhamento:

Você pode montar o compartilhamento Samba como um diretório local no seu sistema Linux.

1.  **Instalar `cifs-utils`:**
    ```bash
    sudo apt install cifs-utils # Ubuntu/Debian
    sudo dnf install cifs-utils # Fedora/CentOS/RHEL
    ```
2.  **Criar um ponto de montagem:**
    ```bash
    sudo mkdir /mnt/samba_share
    ```
3.  **Montar o compartilhamento manualmente:**
    ```bash
    sudo mount -t cifs //192.168.1.100/meus_compartilhados /mnt/samba_share -o username=sambauser1,uid=$(id -u),gid=$(id -g),noperm
    ```
    * `-t cifs`: Especifica o tipo de sistema de arquivos como CIFS (Samba).
    * `username=sambauser1`: O usuário Samba para autenticação.
    * `uid=$(id -u),gid=$(id -g)`: Define o proprietário dos arquivos montados para o seu usuário atual (muito útil).
    * `noperm`: Desabilita a verificação de permissões Unix no cliente (o Samba lida com isso).
    Você será solicitado a digitar a senha do usuário Samba.

4.  **Montagem Persistente (com `/etc/fstab`):**
    Para montar o compartilhamento automaticamente na inicialização, adicione uma entrada ao `/etc/fstab`.

    **Cuidado:** Armazenar senhas diretamente no `/etc/fstab` não é seguro. É melhor usar um arquivo de credenciais.

    Crie um arquivo de credenciais (ex: `/home/seu_usuario/.smbcredentials`) e defina permissões restritivas:
    ```bash
    nano /home/seu_usuario/.smbcredentials
    ```
    Conteúdo do arquivo:
    ```
    username=sambauser1
    password=sua_senha_samba
    ```
    Defina permissões restritivas para o arquivo:
    ```bash
    sudo chmod 600 /home/seu_usuario/.smbcredentials
    ```

    Adicione a seguinte linha ao `/etc/fstab` (substitua IP, nome do compartilhamento e usuário):
    ```bash
    //192.168.1.100/meus_compartilhados /mnt/samba_share cifs credentials=/home/seu_usuario/.smbcredentials,uid=$(id -u),gid=$(id -g),noperm,nofail,x-systemd.automount 0 0
    ```
    * `credentials=/home/seu_usuario/.smbcredentials`: Aponta para o arquivo de credenciais.
    * `nofail`: O sistema inicializa mesmo se o compartilhamento não puder ser montado.
    * `x-systemd.automount`: Monta o compartilhamento somente quando ele é acessado (útil para economizar recursos).

    Teste a montagem:
    ```bash
    sudo mount -a
    ```

O Samba é uma ferramenta poderosa para a colaboração em redes mistas, permitindo que você aproveite a flexibilidade do Linux para gerenciar recursos compartilhados.
