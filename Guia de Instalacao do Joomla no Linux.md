# Guia Completo: Instalação do Joomla no Linux
---

O **Joomla!** é um sistema de gerenciamento de conteúdo (CMS - Content Management System) de código aberto, gratuito e amplamente utilizado para construir sites e aplicativos online robustos. Escrito em PHP, o Joomla requer um servidor web (como Apache) e um banco de dados (como MySQL/MariaDB).

Este guia detalha os passos para instalar o Joomla em um servidor Linux com um stack LAMP já configurado.

## O que é o Joomla?
---

Joomla é uma plataforma poderosa que permite a criação e gerenciamento de conteúdo web de forma eficiente, mesmo para usuários sem conhecimento aprofundado em programação. Ele é conhecido por sua flexibilidade, extensibilidade e uma vasta comunidade de desenvolvedores e usuários.

### Principais Características:

* **Gerenciamento de Conteúdo:** Ferramentas intuitivas para criar, editar e organizar artigos, categorias e menus.
* **Extensibilidade:** Milhares de extensões (componentes, módulos e plugins) disponíveis para adicionar funcionalidades.
* **Templates:** Grande variedade de templates para personalizar a aparência do site.
* **Multilíngue:** Suporte nativo para múltiplos idiomas.
* **Gerenciamento de Usuários:** Controle granular sobre permissões e níveis de acesso.
* **SEO Amigável:** Recursos embutidos para otimização de motores de busca.

## Pré-requisitos
---

Antes de instalar o Joomla, você deve ter um servidor LAMP (Linux, Apache, MySQL/MariaDB, PHP) funcional. Se você seguiu o guia anterior, seu servidor já deve estar pronto. Certifique-se de que o PHP e o MySQL/MariaDB estejam funcionando corretamente.

O Joomla geralmente requer as seguintes extensões PHP (que já devem estar instaladas ou você pode instalá-las):
* `php-mysql` (ou `php-mysqlnd` para Fedora/CentOS)
* `php-curl`
* `php-gd`
* `php-mbstring`
* `php-xml`
* `php-zip`
* `php-intl` (Recomendado para algumas funcionalidades)

Se precisar instalar `php-intl`:
```bash
# Para Ubuntu/Debian:
sudo apt install php-intl
sudo systemctl restart apache2

# Para Fedora/CentOS/RHEL:
sudo dnf install php-intl
sudo systemctl restart httpd
```

## 1. Baixar o Joomla
---

Acesse o site oficial do Joomla e baixe a versão mais recente do pacote de instalação.

1.  **Acesse o diretório raiz do seu servidor web Apache:**
    * Para Ubuntu/Debian: `/var/www/html/`
    * Para Fedora/CentOS/RHEL: `/var/www/html/`

    ```bash
    cd /var/www/html/
    ```

2.  **Baixe o pacote Joomla (substitua a URL pela versão mais recente):**
    ```bash
    sudo wget [https://downloads.joomla.org/cms/joomla4/4-4-5/Joomla_4.4.5-Stable-Full_Package.zip](https://downloads.joomla.org/cms/joomla4/4-4-5/Joomla_4.4.5-Stable-Full_Package.zip)
    ```
    *Verifique sempre o site oficial do Joomla para a versão mais recente.*

3.  **Descompacte o pacote:**
    ```bash
    sudo unzip Joomla_4.4.5-Stable-Full_Package.zip
    ```
    Isso extrairá os arquivos do Joomla diretamente para `/var/www/html/`.

4.  **Remova o arquivo zip:**
    ```bash
    sudo rm Joomla_4.4.5-Stable-Full_Package.zip
    ```

## 2. Criar um Banco de Dados para o Joomla
---

O Joomla precisa de um banco de dados MySQL/MariaDB para armazenar seu conteúdo.

1.  **Acesse o console do MySQL/MariaDB como usuário root:**
    ```bash
    sudo mysql -u root -p
    ```
    Você será solicitado a digitar a senha do usuário `root` do seu banco de dados.

2.  **Crie o banco de dados (ex: `joomla_db`):**
    ```sql
    CREATE DATABASE joomla_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    ```

3.  **Crie um usuário para o banco de dados (ex: `joomla_user`) e defina uma senha forte:**
    ```sql
    CREATE USER 'joomla_user'@'localhost' IDENTIFIED BY 'SUA_SENHA_FORTE';
    ```
    *Substitua `SUA_SENHA_FORTE` por uma senha segura.*

4.  **Conceda todos os privilégios ao novo usuário no banco de dados do Joomla:**
    ```sql
    GRANT ALL PRIVILEGES ON joomla_db.* TO 'joomla_user'@'localhost';
    ```

5.  **Recarregue os privilégios e saia:**
    ```sql
    FLUSH PRIVILEGES;
    EXIT;
    ```

## 3. Configurar Permissões de Arquivo
---

As permissões corretas são cruciais para a segurança e o funcionamento do Joomla. O servidor web (Apache) precisa ter permissão de escrita em certos diretórios.

1.  **Defina o proprietário e o grupo dos arquivos do Joomla para o usuário do servidor web:**
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
    *Este passo é muito importante para a segurança.*

## 4. Executar o Instalador Web do Joomla
---

Agora você pode acessar o instalador do Joomla através do seu navegador web.

1.  Abra seu navegador e navegue para o endereço IP do seu servidor (ou `localhost`):
    ```
    http://<seu_ip_do_servidor>/
    ```
2.  O instalador do Joomla será carregado. Siga os passos na tela:
    * **Configuração Principal:**
        * Nome do Site
        * Endereço de E-mail do Administrador
        * Nome de Usuário e Senha do Super Usuário (admin do Joomla)
    * **Configuração do Banco de Dados:**
        * Tipo de Banco de Dados: `MySQLi` (ou `MariaDB` se for o caso)
        * Nome do Host: `localhost`
        * Nome de Usuário: `joomla_user` (ou o que você criou)
        * Senha: `SUA_SENHA_FORTE` (a senha do usuário do banco de dados)
        * Nome do Banco de Dados: `joomla_db` (ou o que você criou)
        * Prefixo das Tabelas: (padrão, ou mude se tiver múltiplas instalações no mesmo BD)
    * **Instalação Final:**
        * Revise as configurações.
        * Clique em "Instalar".

3.  **Remova o diretório de instalação (CRUCIAL para segurança!):**
    Após a instalação, o Joomla solicitará que você remova a pasta `installation`. Isso é vital para a segurança.
    Você também pode fazer isso manualmente via terminal:
    ```bash
    sudo rm -rf /var/www/html/installation/
    ```

## 5. Configuração do Firewall (se necessário)
---

Se você tem um firewall ativo (como UFW ou Firewalld), certifique-se de que as portas 80 (HTTP) e 443 (HTTPS, se usar SSL/TLS) estejam abertas para permitir o acesso ao seu site Joomla.

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

Parabéns! Seu site Joomla agora deve estar instalado e funcionando no seu servidor Linux. Você pode acessar o painel de administração em `http://<seu_ip_do_servidor>/administrator`.
