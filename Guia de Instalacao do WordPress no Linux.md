# Guia Completo: Instalação do WordPress no Linux
---

O **WordPress** é o sistema de gerenciamento de conteúdo (CMS - Content Management System) mais popular do mundo, utilizado para criar e gerenciar uma vasta gama de sites, desde blogs pessoais a complexos portais corporativos e e-commerce. Escrito em PHP, o WordPress requer um servidor web (como Apache ou Nginx) e um banco de dados (MySQL ou MariaDB).

Este guia detalha os passos para instalar o WordPress em um servidor Linux com um stack LAMP (Linux, Apache, MySQL/MariaDB, PHP) já configurado.

## Pré-requisitos
---

Antes de instalar o WordPress, certifique-se de ter um servidor LAMP funcional e configurado. Se você seguiu guias anteriores, seu servidor já deve estar pronto.

O WordPress geralmente requer as seguintes extensões PHP (além das básicas que já devem estar instaladas):
* `php-curl`
* `php-gd`
* `php-mbstring`
* `php-xml`
* `php-zip`
* `php-imagick` (recomendado para manipulação de imagens)
* `php-intl` (recomendado para algumas funcionalidades)
* `php-fpm` (se estiver usando Nginx, mas para Apache com `mod_php` não é estritamente necessário)

Se precisar instalar extensões adicionais:

```bash
# Para Ubuntu/Debian e derivados:
sudo apt update
sudo apt install php-curl php-gd php-mbstring php-xml php-zip php-imagick php-intl
sudo systemctl restart apache2 # Ou nginx se estiver usando

# Para Fedora/CentOS/RHEL e derivados:
sudo dnf install php-curl php-gd php-mbstring php-xml php-zip php-imagick php-intl
sudo systemctl restart httpd # Ou nginx se estiver usando
```

## 1. Baixar o WordPress
---

Primeiro, acesse o diretório raiz do seu servidor web e baixe o pacote de instalação do WordPress.

1.  **Acesse o diretório raiz do seu servidor web Apache:**
    * Para Ubuntu/Debian: `/var/www/html/`
    * Para Fedora/CentOS/RHEL: `/var/www/html/`

    ```bash
    cd /var/www/html/
    ```

2.  **Baixe o pacote WordPress (substitua a URL pela versão mais recente e estável):**
    ```bash
    sudo wget [https://wordpress.org/latest.zip](https://wordpress.org/latest.zip)
    ```
    *Verifique sempre o site oficial do WordPress ([wordpress.org](https://wordpress.org)) para a versão mais recente.*

3.  **Descompacte o pacote:**
    ```bash
    sudo unzip latest.zip
    ```
    Isso criará um diretório `wordpress` dentro de `/var/www/html/` contendo todos os arquivos do WordPress.

4.  **Mova os arquivos para o diretório raiz do servidor (opcional, mas comum):**
    Para que o WordPress seja acessível diretamente pelo seu IP/domínio (ex: `http://seusite.com/` em vez de `http://seusite.com/wordpress/`), mova os arquivos para o diretório raiz.

    ```bash
    sudo mv wordpress/* .
    sudo rm -r wordpress
    sudo rm latest.zip
    ```

## 2. Criar um Banco de Dados para o WordPress
---

O WordPress precisa de um banco de dados MySQL ou MariaDB para armazenar todo o seu conteúdo (posts, páginas, configurações, usuários, etc.).

1.  **Acesse o console do MySQL/MariaDB como usuário `root` (do banco de dados):**
    ```bash
    sudo mysql -u root -p
    ```
    Você será solicitado a digitar a senha do usuário `root` do seu banco de dados.

2.  **Crie o banco de dados para o WordPress (ex: `wordpress_db`):**
    ```sql
    CREATE DATABASE wordpress_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    ```

3.  **Crie um usuário para o banco de dados (ex: `wordpress_user`) e defina uma senha forte:**
    ```sql
    CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'SUA_SENHA_FORTE_DO_BD';
    ```
    *Substitua `SUA_SENHA_FORTE_DO_BD` por uma senha segura e exclusiva para o banco de dados.*

4.  **Conceda todos os privilégios ao novo usuário no banco de dados do WordPress:**
    ```sql
    GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
    ```

5.  **Recarregue os privilégios e saia:**
    ```sql
    FLUSH PRIVILEGES;
    EXIT;
    ```

## 3. Configurar Permissões de Arquivo
---

As permissões corretas são cruciais para a segurança e o funcionamento do WordPress. O servidor web (Apache) precisa ter permissão de escrita em certos diretórios para que o WordPress possa gerenciar plugins, temas, uploads de mídia, etc.

1.  **Defina o proprietário e o grupo dos arquivos do WordPress para o usuário do servidor web:**
    * No Ubuntu/Debian: `www-data`
    * No Fedora/CentOS/RHEL: `apache`

    ```bash
    sudo chown -R www-data:www-data /var/www/html/ # Para Ubuntu/Debian
    # ou
    sudo chown -R apache:apache /var/www/html/    # Para Fedora/CentOS/RHEL
    ```

2.  **Defina as permissões de diretórios e arquivos:**
    * Diretórios devem ter permissão `755` (rwxr-xr-x).
    * Arquivos devem ter permissão `644` (rw-r--r--).

    ```bash
    sudo find /var/www/html/ -type d -exec chmod 755 {} \;
    sudo find /var/www/html/ -type f -exec chmod 644 {} \;
    ```
    *Este passo é muito importante para a segurança e funcionalidade.*

## 4. Executar o Instalador Web do WordPress
---

Agora você pode acessar o instalador do WordPress através do seu navegador web.

1.  Abra seu navegador e navegue para o endereço IP do seu servidor (ou `localhost`):
    ```
    http://<seu_ip_do_servidor>/
    ```
2.  A página de boas-vindas do WordPress será carregada.
3.  **Selecione o idioma** e clique em "Continuar".
4.  Na tela "Bem-vindo", clique em "Vamos lá!".
5.  **Configuração do Banco de Dados:**
    * **Nome do Banco de Dados:** `wordpress_db` (ou o que você criou)
    * **Nome de Usuário:** `wordpress_user` (ou o que você criou)
    * **Senha:** `SUA_SENHA_FORTE_DO_BD`
    * **Host do Banco de Dados:** `localhost`
    * **Prefixo da Tabela:** `wp_` (padrão, pode ser mudado se você tiver múltiplas instalações no mesmo BD)
    * Clique em "Enviar".
6.  O WordPress tentará se conectar ao banco de dados. Se a conexão for bem-sucedida, você verá a mensagem "Tudo certo, companheiro!". Clique em "Rodar a instalação".
7.  **Informações do Site:**
    * **Título do Site:** O nome do seu site.
    * **Nome de Usuário:** O nome de usuário para o administrador do WordPress (crie um novo, não use "admin" por segurança).
    * **Senha:** Crie uma senha forte para o administrador do WordPress.
    * **Seu E-mail:** O endereço de e-mail do administrador.
    * **Visibilidade nos Mecanismos de Busca:** Marque esta opção se não quiser que seu site seja indexado por mecanismos de busca (útil para sites em desenvolvimento).
    * Clique em "Instalar WordPress".

## 5. Configuração do Firewall (se necessário)
---

Se você tem um firewall ativo (como UFW ou Firewalld), certifique-se de que as portas 80 (HTTP) e 443 (HTTPS, se usar SSL/TLS) estejam abertas para permitir o acesso ao seu site WordPress.

### Usando UFW (Ubuntu/Debian):
```bash
sudo ufw allow 'Apache Full' # Abre portas 80 (HTTP) e 443 (HTTPS)
sudo ufw enable             # Habilitar UFW se não estiver ativo
sudo ufw reload             # Recarregar as regras do UFW
sudo ufw status verbose     # Verificar regras
```

### Usando Firewalld (Fedora/CentOS/RHEL):
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload                              # Recarregar firewall
sudo firewall-cmd --list-all                            # Verificar regras
```

Parabéns! Seu site WordPress agora deve estar instalado e funcionando no seu servidor Linux. Você pode acessar o painel de administração em `http://<seu_ip_do_servidor>/wp-admin`.
