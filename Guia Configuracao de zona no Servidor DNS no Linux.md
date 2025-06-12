# Guia Completo: Instalação e Configuração de um Servidor DNS no Linux
---

Um **servidor DNS** (Domain Name System) é uma peça fundamental da infraestrutura de rede, responsável por traduzir nomes de domínio legíveis por humanos (como `google.com`) em endereços IP numéricos (como `172.217.160.142`) e vice-versa. No Linux, o software mais comum e amplamente utilizado para essa finalidade é o **BIND** (Berkeley Internet Name Domain).

Este guia aborda a instalação e configuração básica de um servidor DNS autoritativo (que gerencia seus próprios domínios) ou recursivo (que resolve consultas para outros domínios) usando o BIND no Linux.

## O que é um Servidor DNS?
---

O DNS é como a "lista telefônica" da internet. Quando você digita um nome de domínio em seu navegador, seu computador envia uma consulta a um servidor DNS, que então retorna o endereço IP associado a esse nome. Sem o DNS, teríamos que memorizar endereços IP para cada site.

### Tipos de Servidores DNS:

* **Servidor Autoritativo:** Contém os registros DNS "originais" para um ou mais domínios (zonas). Ele responde a consultas sobre esses domínios diretamente de seus próprios dados.
* **Servidor Recursivo (Resolver):** Não armazena os registros originais, mas encaminha consultas para outros servidores DNS (autoritativos) e armazena os resultados em cache para consultas futuras.
* **Servidor de Cache:** Um tipo de servidor recursivo que se concentra em armazenar em cache os resultados das consultas para acelerar a resolução.

## 1. Instalação do BIND no Linux
---

O BIND está disponível nos repositórios da maioria das distribuições Linux.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc
```
* `bind9`: O pacote principal do servidor DNS.
* `bind9utils`: Utilitários úteis como `dig`, `nslookup`, `rndc`.
* `bind9-doc`: Documentação do BIND.

### No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install bind bind-utils
```
* `bind`: O pacote principal do servidor DNS (o daemon `named`).
* `bind-utils`: Utilitários como `dig`, `nslookup`.

Após a instalação, o serviço BIND (`named`) será iniciado automaticamente.

## 2. Configuração Básica do BIND (`named.conf`)
---

O arquivo de configuração principal do BIND é `/etc/bind/named.conf` (Ubuntu/Debian) ou `/etc/named.conf` (Fedora/CentOS/RHEL). **Sempre faça um backup antes de editar.**

```bash
# Para Ubuntu/Debian:
sudo cp /etc/bind/named.conf /etc/bind/named.conf.bak
sudo nano /etc/bind/named.conf

# Para Fedora/CentOS/RHEL:
sudo cp /etc/named.conf /etc/named.conf.bak
sudo nano /etc/named.conf
```

### Configuração de um Servidor DNS Recursivo (Caching/Forwarder):

Um servidor recursivo encaminha consultas para outros servidores DNS (geralmente os DNS do seu provedor de internet ou DNS públicos como Google DNS).

Edite o arquivo `/etc/named.conf` (ou `/etc/bind/named.conf`) e procure pela seção `options {}`.

```
options {
    directory "/var/cache/bind"; # Ou "/var/named" para RHEL/CentOS
    # ... outras opções ...

    # Ativar recursão para resolver consultas de clientes
    recursion yes;

    # Permitir que IPs específicos consultem recursivamente
    allow-recursion { 127.0.0.1; 192.168.1.0/24; }; # Seus clientes locais, sua rede

    # Encaminhar consultas não resolvidas localmente para servidores externos
    forwarders {
        8.8.8.8;  # Google Public DNS
        8.8.4.4;  # Google Public DNS
        # Ou os DNS do seu provedor de internet
    };

    # Listen em todas as interfaces ou em IPs específicos
    listen-on { any; }; # Escuta em todas as interfaces. Para maior segurança, defina IPs específicos.
    # Ex: listen-on { 127.0.0.1; 192.168.1.10; }; # Seu IP local e IP do servidor

    # Definir interface para consultas IPv6 (se necessário)
    listen-on-v6 { any; };
};
```

### Configuração de um Servidor DNS Autoritativo (para seus próprios domínios):

Além da configuração recursiva (se desejar que ele também resolva outros domínios), você precisará definir "zonas" para seus próprios domínios.

**Exemplo: Configurando uma zona para `meudominio.com`**

1.  **No `/etc/named.conf` (ou `/etc/bind/named.conf`), adicione a definição da zona:**

    ```
    zone "meudominio.com" IN {
        type master;                # Este servidor é o mestre para esta zona
        file "/etc/bind/db.meudominio.com"; # O arquivo de zona
        allow-update { none; };     # Não permitir atualizações dinâmicas (para iniciantes)
    };

    zone "1.168.192.in-addr.arpa" IN { # Zona reversa para a rede 192.168.1.x
        type master;
        file "/etc/bind/db.192";
        allow-update { none; };
    };
    ```
    * Ajuste o caminho do arquivo de zona conforme sua distribuição.

2.  **Crie o arquivo de zona para `meudominio.com`:**
    ```bash
    # Para Ubuntu/Debian:
    sudo nano /etc/bind/db.meudominio.com

    # Para Fedora/CentOS/RHEL:
    sudo nano /var/named/db.meudominio.com
    ```
    Conteúdo do arquivo `db.meudominio.com`:

    ```dns
    $TTL 604800
    @       IN      SOA     ns1.meudominio.com. admin.meudominio.com. (
                          2024010101 ; Serial (YYYYMMDDNN) - **INCREMENTE ESTE NÚMERO A CADA ALTERAÇÃO**
                          604800     ; Refresh (1 week)
                          86400      ; Retry (1 day)
                          2419200    ; Expire (4 weeks)
                          604800 )   ; Negative Cache TTL (1 week)
    ; Registros de Servidor de Nomes (NS Records)
    ; Apontam para os servidores DNS autoritativos para esta zona.
    @       IN      NS      ns1.meudominio.com.
    @       IN      NS      ns2.meudominio.com. ; Opcional: Se você tiver um segundo servidor DNS (NS secundário)

    ; Registros A (Address Records)
    ; Mapeiam um nome de host para um endereço IPv4.
    ns1     IN      A       192.168.1.100 ; IP do seu servidor DNS primário
    ns2     IN      A       192.168.1.101 ; IP do seu segundo servidor DNS (se houver)
    @       IN      A       192.168.1.100 ; O domínio principal (ex: meudominio.com) aponta para o IP do seu servidor web ou outro
    www     IN      A       192.168.1.100 ; Subdomínio 'www' aponta para o mesmo IP
    ftp     IN      A       192.168.1.100 ; Subdomínio 'ftp' aponta para o mesmo IP
    mail    IN      A       192.168.1.100 ; Subdomínio 'mail' aponta para o mesmo IP

    ; Registro MX (Mail Exchanger Record)
    ; Define o servidor de e-mail para o domínio, e sua prioridade (menor número = maior prioridade).
    @       IN      MX      10 mail.meudominio.com.

    ; Registros CNAME (Canonical Name Records)
    ; Criam um alias de um nome de host para outro nome de host.
    # blog    IN      CNAME   [www.meudominio.com](https://www.meudominio.com). ; Exemplo: 'blog' é um alias para 'www'
    ```
    * `SOA`: Start of Authority - Contém informações sobre o domínio e o servidor DNS primário. **`Serial` deve ser incrementado a cada alteração no arquivo de zona** (geralmente YYYYMMDDNN, onde NN é um contador diário).
    * `NS`: Name Server - Lista os servidores DNS autoritativos para o domínio.
    * `A`: Address - Mapeia um nome de host para um endereço IPv4.
    * `MX`: Mail Exchanger - Define o servidor de e-mail para o domínio.
    * `CNAME`: Canonical Name - Cria um alias de um nome de host para outro.

3.  **Crie o arquivo de zona reversa para a rede `192.168.1.0/24`:**
    ```bash
    # Para Ubuntu/Debian:
    sudo nano /etc/bind/db.192

    # Para Fedora/CentOS/RHEL:
    sudo nano /var/named/db.192
    ```
    Conteúdo do arquivo `db.192`:

    ```dns
    $TTL 604800
    @       IN      SOA     ns1.meudominio.com. admin.meudominio.com. (
                          2024010101 ; Serial
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          604800 )   ; Negative Cache TTL
    ; Name Servers
    @       IN      NS      ns1.meudominio.com.

    ; PTR Records (IP to Host mapping)
    ; Mapeiam um endereço IP para um nome de host (para resolução reversa).
    100     IN      PTR     ns1.meudominio.com. ; Para o IP 192.168.1.100 (host de nome de servidor)
    100     IN      PTR     meudominio.com.     ; Para o IP 192.168.1.100 (domínio principal)
    ```
    * `PTR`: Pointer - Mapeia um endereço IP para um nome de host (para resolução reversa).

4.  **Defina as permissões corretas para os arquivos de zona:**
    * Para Ubuntu/Debian:
        ```bash
        sudo chown -R bind:bind /etc/bind/
        sudo chmod -R 644 /etc/bind/db.*
        ```
    * Para Fedora/CentOS/RHEL:
        ```bash
        sudo chown -R named:named /var/named/
        sudo restorecon -Rv /var/named/ # Para SELinux
        ```

5.  **Verifique a sintaxe dos arquivos de zona:**
    ```bash
    sudo named-checkconf
    sudo named-checkzone meudominio.com /etc/bind/db.meudominio.com # Ubuntu/Debian
    sudo named-checkzone meudominio.com /var/named/db.meudominio.com # RHEL/CentOS
    sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192 # Ubuntu/Debian
    sudo named-checkzone 1.168.192.in-addr.arpa /var/named/db.192 # RHEL/CentOS
    ```
    Se não houver erros, a sintaxe está correta.

6.  **Reinicie o serviço BIND:**
    ```bash
    sudo systemctl restart bind9 # Ubuntu/Debian
    sudo systemctl restart named  # Fedora/CentOS/RHEL
    ```

## 3. Configuração de Firewall
---

Para permitir que outros clientes consultem seu servidor DNS, você precisa abrir as portas `53` (TCP e UDP) no firewall.

### Usando UFW (Ubuntu/Debian):

```bash
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw reload
sudo ufw status verbose
```

### Usando Firewalld (Fedora/CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

## 4. Testando o Servidor DNS com Clientes
---

Para que seus clientes usem o novo servidor DNS, eles precisam ser configurados para apontar para o endereço IP do seu servidor BIND.

1.  **Configure seu cliente para usar o novo servidor DNS:**
    * **No Linux (temporário):** Edite o arquivo `/etc/resolv.conf`. **Atenção:** Este arquivo é geralmente sobrescrito por NetworkManager ou `systemd-resolved`.
        ```bash
        sudo nano /etc/resolv.conf
        ```
        Adicione a linha (ou mude a existente para o IP do seu servidor DNS):
        ```
        nameserver 192.168.1.100 # Substitua pelo IP do seu servidor DNS
        ```
    * **No Linux (persistente em desktop):** Use as configurações de rede via interface gráfica (GNOME Settings, KDE Plasma System Settings) para alterar o DNS do seu adaptador de rede.
    * **Em outros sistemas (Windows, macOS):** Altere as configurações de DNS do adaptador de rede para o IP do seu servidor DNS (ex: `192.168.1.100`).
    * **Em um roteador:** Configure o roteador para usar seu servidor DNS como o servidor primário, distribuindo-o via DHCP para os dispositivos conectados.

2.  **Use `dig` ou `nslookup` para testar as consultas DNS:**

    * **Instale `dig` (se não tiver):**
        ```bash
        # Ubuntu/Debian
        sudo apt install dnsutils

        # Fedora/CentOS/RHEL
        sudo dnf install bind-utils
        ```

    * **Consultar seu próprio domínio (zona autoritativa) usando seu servidor DNS:**
        ```bash
        dig @192.168.1.100 meudominio.com           # Consulta o registro A do domínio principal
        dig @192.168.1.100 [www.meudominio.com](https://www.meudominio.com) A     # Consulta o registro A de 'www'
        dig @192.168.1.100 meudominio.com MX        # Consulta o registro MX (servidor de e-mail)
        dig @192.168.1.100 -x 192.168.1.100         # Consulta reversa (IP para nome)
        ```
        A seção `ANSWER SECTION` da saída do `dig` deve mostrar as informações corretas que você configurou.

    * **Consultar um domínio externo (se o servidor for recursivo/forwarder):**
        ```bash
        dig @192.168.1.100 google.com
        ```
        O servidor deve resolver o IP do Google.

    * **Usando `nslookup` (alternativa a `dig`):**
        ```bash
        nslookup meudominio.com 192.168.1.100
        nslookup google.com 192.168.1.100
        ```

    * **Usando `host` (ferramenta simples):**
        ```bash
        host meudominio.com 192.168.1.100
        host google.com 192.168.1.100
        ```

### Solução de Problemas Comuns no Cliente:

* **Não consegue resolver nenhum nome:**
    * Verifique se o cliente está realmente usando seu servidor DNS (confira `/etc/resolv.conf` ou as configurações de rede).
    * Verifique a conectividade de rede entre o cliente e o servidor DNS (use `ping 192.168.1.100`).
    * Certifique-se de que o firewall no servidor DNS (e no cliente, se houver) está permitindo o tráfego na porta 53 (TCP/UDP).
    * Verifique o status do serviço BIND no servidor: `sudo systemctl status bind9` (Ubuntu/Debian) ou `sudo systemctl status named` (Fedora/CentOS/RHEL).
    * Verifique os logs do BIND para erros: `sudo journalctl -u bind9` ou `sudo journalctl -u named`.

* **Não consegue resolver seu próprio domínio (`meudominio.com`), mas resolve outros:**
    * Indica um problema na configuração da zona (arquivo `db.meudominio.com`) ou na sua definição no `named.conf`.
    * Verifique a sintaxe da zona com `sudo named-checkzone` e o arquivo de configuração com `sudo named-checkconf`.
    * Confira se o número serial `SOA` foi incrementado após as alterações no arquivo de zona.

* **Não consegue resolver domínios externos (google.com), mas resolve seu próprio:**
    * Indica um problema na configuração de recursão ou dos `forwarders` no `named.conf`.
    * Verifique as opções `recursion yes` e `allow-recursion` e `forwarders`.

## Conclusão
---

A instalação e configuração de um servidor DNS com BIND no Linux é um processo que exige atenção aos detalhes, mas é fundamental para o gerenciamento de nomes de domínio em sua rede. Com um servidor DNS bem configurado, você pode melhorar a resolução de nomes, criar sua própria infraestrutura de domínio e otimizar o desempenho da rede. Lembre-se de manter o BIND atualizado e de seguir as melhores práticas de segurança.
