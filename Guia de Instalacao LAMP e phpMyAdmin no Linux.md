# Guia Completo: Instalação do LAMP e phpMyAdmin no Linux
---

O **LAMP** é um acrônimo para um stack de software de código aberto composto por **L**inux, **A**pache, **M**ySQL/MariaDB e **P**HP. É a plataforma mais popular para hospedar aplicações web dinâmicas e sites. Adicionalmente, o **phpMyAdmin** é uma ferramenta gráfica baseada na web para gerenciar bancos de dados MySQL/MariaDB.

Este guia fornecerá os passos para instalar e configurar o stack LAMP completo e o phpMyAdmin em distribuições Linux comuns.

## O que é o Stack LAMP?
---

* **Linux:** O sistema operacional subjacente.
* **Apache:** O servidor web (HTTP) que serve os arquivos do site para os navegadores.
* **MySQL/MariaDB:** O sistema de gerenciamento de banco de dados relacional que armazena os dados do site. (MariaDB é um fork compatível e frequentemente preferido ao MySQL).
* **PHP:** A linguagem de script do lado do servidor que processa o conteúdo dinâmico do site.

## 1. Instalação do Apache (Servidor Web)
---

O Apache HTTP Server é o componente do servidor web do stack LAMP.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```

### No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Testar Apache:
Abra seu navegador web e digite o endereço IP do seu servidor (ou `localhost` se estiver na mesma máquina). Você deverá ver a página padrão do Apache ("It works!" ou "Apache2 Ubuntu Default Page").

## 2. Instalação do MySQL/MariaDB (Banco de Dados)
---

O MySQL ou MariaDB é o sistema de gerenciamento de banco de dados.

### No Ubuntu/Debian e derivados (MariaDB recomendado):

```bash
sudo apt install mariadb-server mariadb-client
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
* **Para MySQL (alternativa ao MariaDB):**
    ```bash
    sudo apt install mysql-server mysql-client
    sudo systemctl start mysql
    sudo systemctl enable mysql
    ```

### No Fedora/CentOS/RHEL e derivados (MariaDB recomendado):

```bash
sudo dnf install mariadb-server mariadb
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
* **Para MySQL (alternativa ao MariaDB):**
    ```bash
    sudo dnf install mysql-server
    sudo systemctl start mysqld
    sudo systemctl enable mysqld
    ```

### Configuração de Segurança do Banco de Dados:
É crucial executar o script de segurança para proteger sua instalação do banco de dados.

```bash
sudo mysql_secure_installation
```
Este script irá guiá-lo através de:
* Definir/mudar a senha do usuário `root` do banco de dados.
* Remover usuários anônimos.
* Desativar o login remoto do `root`.
* Remover o banco de dados de teste.
* Recarregar as tabelas de privilégios.

## 3. Instalação do PHP (Linguagem de Script)
---

O PHP é a linguagem de programação que o Apache usará para processar páginas dinâmicas. Você precisará do módulo do Apache para PHP e extensões comuns.

### No Ubuntu/Debian e derivados:

```bash
sudo apt install php libapache2-mod-php php-mysql php-cli php-json php-curl php-gd php-mbstring php-xml php-zip
sudo systemctl restart apache2
```
* `php-mysql`: Extensão necessária para o PHP se conectar ao MySQL/MariaDB.
* Outras extensões são comuns e úteis para a maioria das aplicações web.

### No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install php php-mysqlnd php-cli php-json php-curl php-gd php-mbstring php-xml php-zip
sudo systemctl restart httpd
```
* `php-mysqlnd`: É o driver MySQL para PHP no DNF.

### Testar PHP:
Crie um arquivo de teste no diretório raiz do seu servidor web.

**No Ubuntu/Debian:**
```bash
sudo nano /var/www/html/info.php
```
**No Fedora/CentOS/RHEL:**
```bash
sudo nano /var/www/html/info.php
```
Adicione o seguinte conteúdo ao arquivo `info.php`:

```php
<?php
phpinfo();
?>
```
Salve e feche o arquivo. Abra seu navegador e navegue para `http://<seu_ip_do_servidor>/info.php`. Você deverá ver uma página com informações detalhadas sobre a instalação do PHP. Após verificar, é **altamente recomendável excluir este arquivo** por motivos de segurança: `sudo rm /var/www/html/info.php`.

## 4. Instalação do phpMyAdmin (Gerenciamento de BD)
---

O phpMyAdmin é uma interface web para gerenciar seus bancos de dados MySQL/MariaDB.

### No Ubuntu/Debian e derivados:

```bash
sudo apt install phpmyadmin
```
Durante a instalação, você será perguntado:
* **Servidor web para reconfigurar:** Selecione `apache2` e pressione `Espaço`, depois `Enter`.
* **Configurar banco de dados para phpmyadmin com dbconfig-common:** Selecione `Yes` (Sim).
* Você será solicitado a definir uma senha para o usuário `phpmyadmin` do banco de dados.

### No Fedora/CentOS/RHEL e derivados:

A instalação do phpMyAdmin via DNF não faz a integração automática com o Apache como no Debian/Ubuntu. Você precisará fazer algumas configurações manuais.

```bash
sudo dnf install phpmyadmin
```

**Configurar Apache para phpMyAdmin (Fedora/CentOS/RHEL):**
1.  **Edite o arquivo de configuração do phpMyAdmin para Apache:**
    ```bash
    sudo nano /etc/httpd/conf.d/phpMyAdmin.conf
    ```
2.  Dentro deste arquivo, localize as linhas `Require ip 127.0.0.1` e `Require ip ::1`. Se você quer acessar o phpMyAdmin de outros IPs na sua rede local, adicione uma linha para sua rede ou comente-as e adicione `Require all granted` (para redes de teste). **Para produção, restrinja o acesso por IP.**

    **Exemplo para permitir acesso da rede local (192.168.1.0/24):**
    ```apache
    <Directory /usr/share/phpMyAdmin/>
        AddDefaultCharset UTF-8

        <IfModule mod_authz_core.c>
            # Apache 2.4
            Require ip 127.0.0.1
            Require ip ::1
            Require ip 192.168.1.0/24  # Adicione esta linha para sua rede local
        </IfModule>
        # ...
    </Directory>
    ```
3.  **Reinicie o Apache:**
    ```bash
    sudo systemctl restart httpd
    ```

### Acessar phpMyAdmin:
Abra seu navegador e navegue para `http://<seu_ip_do_servidor>/phpmyadmin`.
Você será solicitado a fazer login. Use o usuário `root` do banco de dados e a senha que você definiu durante `mysql_secure_installation`, ou o usuário `phpmyadmin` com a senha que você configurou durante a instalação do pacote.

## 5. Configuração de Firewall
---

Para permitir acesso externo ao seu servidor web (Apache) e phpMyAdmin, você precisa abrir as portas no seu firewall.

### Usando UFW (Ubuntu/Debian):

```bash
sudo ufw allow 'Apache Full' # Abre portas 80 (HTTP) e 443 (HTTPS)
sudo ufw enable             # Habilitar UFW se não estiver ativo
sudo ufw status verbose     # Verificar regras
```
* Se você quiser apenas HTTP: `sudo ufw allow 'Apache'` (porta 80).

### Usando Firewalld (Fedora/CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload                              # Recarregar firewall
sudo firewall-cmd --list-all                            # Verificar regras
```

Com o stack LAMP e o phpMyAdmin instalados e configurados, você tem uma base sólida para desenvolver e hospedar suas aplicações web no Linux!