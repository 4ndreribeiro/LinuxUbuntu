# Guia Completo: Configurar IP Estático com Netplan no Linux
---

O **Netplan** é uma ferramenta de configuração de rede em sistemas Linux que simplifica o gerenciamento de interfaces de rede. Introduzido no Ubuntu 17.10 e adotado em várias distribuições baseadas em Debian, o Netplan usa arquivos YAML para definir as configurações de rede, que são então processadas por renderizadores (como `systemd-networkd` ou NetworkManager) para aplicar as configurações reais.

Este guia detalha como configurar um endereço IP estático para uma interface de rede usando o Netplan.

## O que é o Netplan?
---

O Netplan atua como uma abstração para as configurações de rede. Em vez de editar diretamente arquivos de configuração de rede complexos e específicos do renderizador (como `ifupdown` ou arquivos do NetworkManager), você define suas configurações de rede em arquivos YAML concisos. O Netplan então gera os arquivos de configuração necessários para o renderizador de rede escolhido.

### Por que Usar o Netplan?

* **Simplificação:** Escreve configurações de rede em um formato YAML simples e legível.
* **Consistência:** Fornece uma maneira unificada de configurar a rede em diferentes distribuições que o suportam, independentemente do backend de rede.
* **Flexibilidade:** Suporta DHCP, IPs estáticos, bridges, VLANs e outras configurações de rede avançadas.
* **Automação:** Ideal para automação e implantação de servidores, onde a consistência da configuração de rede é crucial.

## Pré-requisitos
---

* **Sistema Linux com Netplan:** Ubuntu (17.10 ou posterior), Debian (10 ou posterior, pode precisar de instalação manual do Netplan), ou outras distribuições que o utilizem.
* **Acesso Root (sudo):** Você precisará de privilégios de superusuário para modificar os arquivos de configuração de rede.
* **Nome da Interface de Rede:** Saiba o nome da interface de rede que você deseja configurar (ex: `eth0`, `enp0s3`, `ens33`).

## Como Configurar IP Estático com Netplan
---

Vamos configurar um IP estático para uma interface de rede, por exemplo, `enp0s3`, com o IP `192.168.1.100/24`, gateway `192.168.1.1` e DNS `8.8.8.8`, `8.8.4.4`.

### 1. Identificar Interfaces de Rede

Use o comando `ip a` (ou `ifconfig`, se ainda instalado) para listar suas interfaces de rede e seus nomes.

```bash
ip a
```
Procure por nomes como `enpXsY`, `eth0`, `ensZ`.

### 2. Fazer Backup da Configuração Existente

É crucial fazer um backup do arquivo de configuração atual do Netplan antes de fazer alterações. Os arquivos de configuração do Netplan são geralmente encontrados em `/etc/netplan/`.

```bash
sudo cp /etc/netplan/*.yaml /etc/netplan/backup/
```
*Se a pasta `backup` não existir, crie-a primeiro: `sudo mkdir /etc/netplan/backup`*

### 3. Criar/Editar Arquivo de Configuração do Netplan

Os arquivos de configuração do Netplan devem ter a extensão `.yaml` (ou `.yml`). É comum ter um arquivo como `01-netcfg.yaml` ou `50-cloud-init.yaml`. Você pode editar um existente ou criar um novo. Se criar um novo, certifique-se de que ele tenha um nome que garanta que ele seja carregado corretamente (nomes com números menores são processados primeiro).

1.  **Abra o arquivo de configuração para edição:**
    Vamos editar o arquivo `.yaml` que você já possui, ou criar um novo se não houver um.

    ```bash
    sudo nano /etc/netplan/00-installer-config.yaml
    # Ou use o nome do seu arquivo existente, ex: sudo nano /etc/netplan/50-cloud-init.yaml
    ```
    *Use `nano` ou `vi` ou seu editor preferido.*

2.  **Adicione ou modifique o conteúdo para configurar o IP estático:**

    ```yaml
    network:
      version: 2
      renderer: networkd # Ou NetworkManager, dependendo do seu sistema e uso
      ethernets:
        enp0s3:          # Substitua pelo nome da sua interface de rede
          dhcp4: no      # Desabilita DHCP para IPv4
          addresses:
            - 192.168.1.100/24 # Seu endereço IP estático e máscara de sub-rede
          routes:
            - to: default
              via: 192.168.1.1  # Seu gateway padrão
          nameservers:
            addresses: [8.8.8.8, 8.8.4.4] # Servidores DNS
            search: [sua_dominio_local.com] # Opcional: seu domínio de busca local
    ```

    **Explicação dos Campos:**

    * **`network:`**: Seção raiz do arquivo Netplan.
    * **`version: 2`**: Especifica a versão do formato Netplan.
    * **`renderer: networkd`**: O backend de rede que o Netplan usará. `networkd` é o padrão para a maioria dos servidores. Em desktops, pode ser `NetworkManager`.
    * **`ethernets:`**: Contém as configurações para interfaces Ethernet.
    * **`enp0s3:`**: O nome da sua interface de rede (ajuste conforme o seu sistema).
    * **`dhcp4: no`**: Desabilita o DHCP para IPv4 nesta interface.
    * **`addresses:`**: Lista de endereços IP a serem atribuídos à interface.
        * `- 192.168.1.100/24`: Seu endereço IP estático seguido da máscara de sub-rede em notação CIDR.
    * **`routes:`**: Define as rotas de rede.
        * `- to: default`: A rota padrão (todo o tráfego que não tem uma rota específica).
        * `via: 192.168.1.1`: O endereço IP do seu gateway.
    * **`nameservers:`**: Configura os servidores DNS.
        * `addresses: [8.8.8.8, 8.8.4.4]`: Lista de endereços IP dos servidores DNS.
        * `search: [sua_dominio_local.com]`: (Opcional) Domínios de busca para resolução de nomes curtos.

    **Atenção à indentação YAML:** A indentação é crucial em arquivos YAML. Use espaços (não tabs) e mantenha a consistência.

### 4. Aplicar Configuração do Netplan

Depois de salvar o arquivo de configuração, use o comando `netplan apply` para aplicar as mudanças.

```bash
sudo netplan apply
```
* Se houver erros de sintaxe no seu arquivo YAML, o Netplan os reportará. Corrija-os e tente novamente.
* Se a configuração for aplicada com sucesso, as interfaces de rede serão reconfiguradas.

### 5. Verificar Configuração

Após aplicar as mudanças, verifique se o novo IP estático foi atribuído corretamente.

```bash
ip a show enp0s3 # Substitua pelo nome da sua interface
```
Você deve ver o `inet 192.168.1.100/24` listado na saída para sua interface.

Teste a conectividade de rede:

```bash
ping google.com    # Testar resolução DNS e conectividade externa
ping 192.168.1.1   # Testar conectividade com o gateway
```

## Solução de Problemas Comuns
---

* **Erros de Sintaxe YAML:** O erro mais comum. O Netplan é muito sensível à indentação. Use um validador YAML online ou um editor de texto com suporte a YAML para verificar a sintaxe.
* **"Error in network definition: Invalid YAML"**: Indica um problema de sintaxe.
* **Conectividade Perdida após `netplan apply`:** Se você perdeu a conexão SSH ou de rede, pode ser um erro na configuração.
    * Se você estiver em um servidor físico ou máquina virtual com acesso ao console, faça login e reverta para o arquivo de backup:
        ```bash
        sudo cp /etc/netplan/backup/*.yaml /etc/netplan/
        sudo netplan apply
        ```
    * Verifique os logs do sistema para mais detalhes: `sudo journalctl -u systemd-networkd` (se `renderer: networkd`) ou `sudo journalctl -u NetworkManager` (se `renderer: NetworkManager`).
* **Conflitos de IP:** Certifique-se de que o IP estático que você configurou não está sendo usado por outro dispositivo na sua rede.
* **Problemas de DNS:** Se a navegação por nomes não funcionar, mas o ping por IP sim, verifique a seção `nameservers` no seu arquivo Netplan.

O Netplan simplifica significativamente a configuração de rede no Linux. Com este guia, você deve ser capaz de configurar IPs estáticos de forma eficaz e segura.
