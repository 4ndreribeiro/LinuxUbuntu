# Guia Completo: Instalação do MongoDB no Linux
---

O **MongoDB** é um popular banco de dados NoSQL baseado em documentos, conhecido por sua flexibilidade, escalabilidade e alto desempenho. Ele armazena dados em documentos JSON (ou BSON, uma versão binária do JSON) com esquemas flexíveis, o que o torna ideal para aplicações modernas que exigem agilidade e manipulação de dados não estruturados ou semi-estruturados.

Este guia detalha como instalar o MongoDB Community Edition em diferentes distribuições Linux e como iniciar e gerenciar o serviço.

## O que é o MongoDB?
---

Diferente de bancos de dados relacionais (SQL) que usam tabelas e linhas, o MongoDB armazena dados em coleções e documentos. Cada documento pode ter sua própria estrutura, permitindo que os desenvolvedores evoluam seus modelos de dados sem a necessidade de migrações complexas.

### Principais Características:

* **Orientado a Documentos:** Armazena dados em documentos BSON (um formato binário de JSON), o que é natural para desenvolvedores que trabalham com JSON em suas aplicações.
* **Esquema Flexível:** Não exige um esquema rígido. Documentos na mesma coleção podem ter diferentes campos e estruturas.
* **Escalabilidade Horizontal:** Projetado para escalar horizontalmente através de *sharding* (distribuição de dados em múltiplos servidores).
* **Alta Disponibilidade:** Suporta conjuntos de réplicas (*replica sets*) para failover automático e redundância de dados.
* **Consultas Ricas:** Oferece uma linguagem de consulta poderosa e flexível para agregação de dados e consultas complexas.
* **Índices:** Suporta vários tipos de índices para otimizar o desempenho das consultas.

## Pré-requisitos
---

* **Sistema Linux:** Uma distribuição Linux moderna (Ubuntu, Debian, Fedora, CentOS, RHEL).
* **Acesso Root (sudo):** Você precisará de privilégios de superusuário para instalar e configurar o MongoDB.
* **Conexão com a Internet:** Para baixar os pacotes do repositório oficial do MongoDB.

## 1. Instalação do MongoDB Community Edition no Linux
---

É altamente recomendado instalar o MongoDB a partir dos repositórios oficiais do MongoDB, pois eles fornecem as versões mais recentes e estáveis.

### 1.1. Para Ubuntu/Debian e derivados

Siga estes passos para instalar o MongoDB no Ubuntu 22.04 (Jammy Jellyfish). Para outras versões, consulte a documentação oficial do MongoDB, pois as URLs e chaves podem mudar.

1.  **Importe a chave pública do GPG do MongoDB:**
    ```bash
    curl -fsSL [https://www.mongodb.org/static/pgp/server-6.0.asc](https://www.mongodb.org/static/pgp/server-6.0.asc) | \
       sudo gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb-6.0.asc > /dev/null
    ```
    *Para MongoDB 7.0, use `server-7.0.asc` na URL.*

2.  **Crie um arquivo de lista para o repositório MongoDB:**
    Para Ubuntu 22.04:
    ```bash
    echo "deb [ arch=amd64,arm64 ] [https://repo.mongodb.org/apt/ubuntu](https://repo.mongodb.org/apt/ubuntu) jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
    ```
    *Se você estiver usando Ubuntu 20.04 (Focal Fossa), substitua `jammy` por `focal`.*
    *Para MongoDB 7.0, use `7.0` na URL e `mongodb-org-7.0.list` no nome do arquivo.*

3.  **Atualize o índice de pacotes:**
    ```bash
    sudo apt update
    ```

4.  **Instale o pacote `mongodb-org`:**
    ```bash
    sudo apt install -y mongodb-org
    ```
    Isso instalará os seguintes pacotes:
    * `mongodb-org-server`: O daemon do servidor MongoDB (`mongod`).
    * `mongodb-org-mongos`: O daemon do servidor `mongos` (para sharding).
    * `mongodb-org-shell`: O shell interativo do MongoDB (`mongosh`).
    * `mongodb-org-tools`: Ferramentas de importação/exportação e outras utilidades.

### 1.2. Para Fedora/CentOS/RHEL e derivados

Siga estes passos para instalar o MongoDB no Fedora 38. Para outras versões, consulte a documentação oficial do MongoDB.

1.  **Crie um arquivo de repositório para o MongoDB:**
    ```bash
    sudo nano /etc/yum.repos.d/mongodb-org-6.0.repo
    ```
    Adicione o seguinte conteúdo (para Fedora 38, MongoDB 6.0):
    ```ini
    [mongodb-org-6.0]
    name=MongoDB Repository
    baseurl=[https://repo.mongodb.org/yum/redhat/8/mongodb-org/6.0/x86_64/](https://repo.mongodb.org/yum/redhat/8/mongodb-org/6.0/x86_64/)
    gpgcheck=1
    enabled=1
    gpgkey=[https://www.mongodb.org/static/pgp/server-6.0.asc](https://www.mongodb.org/static/pgp/server-6.0.asc)
    ```
    *Ajuste a `baseurl` e `gpgkey` conforme sua versão do Fedora/RHEL e a versão do MongoDB que você deseja instalar (ex: `7.0/x86_64` para MongoDB 7.0).*
    *Para RHEL/CentOS 8, use `redhat/8` na `baseurl`. Para RHEL/CentOS 7, use `redhat/7`.*

2.  **Instale o pacote `mongodb-org`:**
    ```bash
    sudo dnf install -y mongodb-org
    # ou para CentOS/RHEL 7: sudo yum install -y mongodb-org
    ```

## 2. Iniciar e Gerenciar o Serviço MongoDB
---

Após a instalação, o serviço MongoDB (`mongod`) pode não ser iniciado automaticamente. Você precisará iniciá-lo e habilitá-lo para iniciar com o sistema.

1.  **Iniciar o serviço MongoDB:**
    ```bash
    sudo systemctl start mongod
    ```

2.  **Verificar o status do serviço:**
    ```bash
    sudo systemctl status mongod
    ```
    Você deve ver `active (running)`.

3.  **Habilitar o MongoDB para iniciar automaticamente na inicialização:**
    ```bash
    sudo systemctl enable mongod
    ```

4.  **Parar o serviço MongoDB:**
    ```bash
    sudo systemctl stop mongod
    ```

5.  **Reiniciar o serviço MongoDB:**
    ```bash
    sudo systemctl restart mongod
    ```

## 3. Configuração de Firewall
---

Se você tem um firewall ativo (como UFW no Ubuntu/Debian ou Firewalld no Fedora/CentOS/RHEL), você precisa permitir o tráfego na porta padrão do MongoDB, que é a **porta 27017**.

### Usando UFW (Ubuntu/Debian):

```bash
sudo ufw allow 27017/tcp
sudo ufw reload             # Recarregar as regras do UFW
sudo ufw status verbose     # Verificar regras
```

### Usando Firewalld (Fedora/CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-port=27017/tcp
sudo firewall-cmd --reload                              # Recarregar firewall
sudo firewall-cmd --list-all                            # Verificar regras
```

## 4. Testando o MongoDB
---

Depois que o serviço MongoDB estiver em execução e as portas do firewall estiverem abertas, você pode se conectar ao banco de dados usando o shell `mongosh`.

1.  **Inicie o shell `mongosh`:**
    ```bash
    mongosh
    ```
    Você verá um prompt `test>` (ou o nome do banco de dados padrão).

2.  **Verificar o status do servidor (no shell `mongosh`):**
    ```javascript
    db.runCommand( { serverStatus: 1 } )
    ```
    Isso mostrará muitas informações sobre o servidor MongoDB.

3.  **Criar e Inserir um Documento de Teste:**
    ```javascript
    use mydatabase; // Seleciona ou cria um banco de dados
    db.mycollection.insertOne({ name: "Test User", age: 30, city: "Sampleton" });
    ```

4.  **Consultar o Documento:**
    ```javascript
    db.mycollection.find();
    ```

5.  **Sair do shell `mongosh`:**
    ```javascript
    exit
    ```

## 5. Arquivos e Diretórios Importantes
---

* **Arquivo de Configuração:** `/etc/mongod.conf`
    * Aqui você pode alterar configurações como o caminho dos dados, caminho dos logs, porta, segurança e vinculação de IP (por padrão, o MongoDB escuta apenas em `127.0.0.1`).
    * **Para permitir conexões de outros IPs**, você precisa modificar a linha `bindIp` para incluir o IP do seu servidor ou `0.0.0.0` para todas as interfaces (use com cautela e configure a autenticação!).
        ```yaml
        # file: /etc/mongod.conf
        net:
          port: 27017
          bindIp: 127.0.0.1,192.168.1.100 # Adicione seu IP aqui para acesso externo
        ```
        Após alterar, reinicie o `mongod`.

* **Diretório de Dados:** `/var/lib/mongodb`
* **Diretório de Logs:** `/var/log/mongodb/mongod.log`

## Considerações Finais
---

* **Segurança:** Para ambientes de produção, **NUNCA** execute o MongoDB sem autenticação. Habilite a autenticação baseada em usuário e senha no `mongod.conf` (`security.authorization: enabled`) e crie usuários com funções apropriadas.
* **Atualizações:** Mantenha seu MongoDB atualizado para obter as últimas funcionalidades e patches de segurança. Você pode usar os comandos `apt update && apt upgrade` ou `dnf update` normalmente após configurar o repositório oficial.
* **Monitoramento:** Monitore o desempenho do seu servidor MongoDB usando ferramentas de monitoramento para garantir seu bom funcionamento.

Com o MongoDB instalado e configurado, você está pronto para começar a desenvolver suas aplicações com um banco de dados NoSQL poderoso e flexível no Linux!
