# Comandos UFW

O ufw (Uncomplicated Firewall) é uma ferramenta de linha de comando para gerenciar as regras de firewall no Linux, especialmente popular em distribuições baseadas em Debian e Ubuntu. Ele foi criado para simplificar o processo de configuração do iptables, que é o firewall nativo do Linux, mas tem uma sintaxe mais complexa.

### Como funciona o UFW:

 >O UFW atua como uma "interface" amigável para o iptables. Em vez de você ter que lidar diretamente com as cadeias e tabelas do iptables, o UFW oferece comandos mais intuitivos para permitir ou negar tráfego. Ele traduz seus comandos simples em regras complexas de iptables nos bastidores.

### Principais características e funcionamento:

> Simplificação: A principal função é tornar a configuração do firewall mais fácil e menos propensa a erros, especialmente para usuários iniciantes ou para tarefas comuns.
Políticas Padrão: Por padrão, quando você habilita o UFW, ele nega todas as conexões de entrada (incoming) e permite todas as conexões de saída (outgoing). Isso significa que seu servidor não aceitará conexões não solicitadas, mas poderá se conectar a outros serviços na internet. Essa é uma base de segurança sólida.
Regras: Você pode adicionar regras específicas para permitir ou negar tráfego para portas, protocolos, endereços IP, sub-redes e até mesmo para serviços específicos (como SSH, HTTP, HTTPS).
Perfis de Aplicações: O UFW pode reconhecer nomes de serviços comuns (como "ssh", "http", "https") e configurar as regras apropriadas para as portas padrão associadas a esses serviços. Isso é possível porque ele lê o arquivo /etc/services.
IPv4 e IPv6: O UFW gerencia regras para ambos os protocolos IPv4 e IPv6 automaticamente.
Logging: Você pode habilitar o registro (logging) das atividades do firewall para monitorar e solucionar problemas.
Como usar o comando UFW:

### Todos os comandos UFW geralmente precisam ser executados com privilégios de superusuário (root), usando sudo.

**1. Verificar o status do UFW:**

Antes de fazer qualquer alteração, é bom verificar o status atual do UFW:

Bash
```bash
sudo ufw status
```
Se o UFW estiver inativo, a saída será Status: inactive. Se estiver ativo, ele mostrará as regras configuradas. Para uma visão mais detalhada das regras e políticas, use:

Bash
```bash
sudo ufw status verbose
```
**2. Habilitar o UFW:**

> [!IMPORTANT] 
> Se o UFW estiver inativo, você precisa habilitá-lo. CUIDADO: Se você estiver conectado via SSH, certifique-se de permitir o acesso SSH antes de habilitar o UFW, caso contrário, você pode se bloquear.

Bash
```bash
sudo ufw enable
```
> [!NOTE] 
> Você será perguntado se deseja prosseguir, pois isso pode interromper as conexões SSH existentes. Digite y e Enter.

**3. Desabilitar o UFW:**

Para desabilitar o UFW (deixando seu sistema desprotegido):

Bash
```bash
sudo ufw disable
```
**4. Resetar o UFW:**

Para redefinir o UFW para suas configurações padrão (remover todas as regras personalizadas e desabilitá-lo):

Bash
```bash
sudo ufw reset
```
**5. Definir políticas padrão:**

As políticas padrão são as regras que se aplicam a todo o tráfego que não é explicitamente permitido ou negado por uma regra específica.

Negar todas as conexões de entrada (Incoming): (Esta é a política padrão quando você habilita o UFW)
Bash
```bash
sudo ufw default deny incoming
```
Permitir todas as conexões de saída (Outgoing): (Esta é a política padrão quando você habilita o UFW)
Bash
```bash
sudo ufw default allow outgoing
```
**6. Adicionar regras para permitir tráfego:**

Permitir acesso SSH (porta 22):
Bash
```bash
sudo ufw allow ssh
```
## Ou especificando a porta:
sudo ufw allow 22/tcp
Permitir acesso HTTP (porta 80):
Bash
```bash
sudo ufw allow http
```
## Ou especificando a porta:
```bash
sudo ufw allow 80/tcp
```
Permitir acesso HTTPS (porta 443):
Bash
```bash
sudo ufw allow https
```
## Ou especificando a porta:
```bash
sudo ufw allow 443/tcp
```
Permitir tráfego em uma porta específica (ex: porta 8080 para uma aplicação web):
Bash
```bash
sudo ufw allow 8080
```
## Ou especificando o protocolo (TCP é o padrão se não especificado):
```bash
sudo ufw allow 8080/tcp
sudo ufw allow 8080/udp
```
Permitir um intervalo de portas:
Bash
```bash
sudo ufw allow 6000:6007/tcp
```
Permitir tráfego de um IP específico:
Bash
```bash
sudo ufw allow from 192.168.1.100
```
Permitir tráfego de um IP específico para uma porta específica:
Bash
```bash
sudo ufw allow from 192.168.1.100 to any port 22
```
Permitir tráfego de uma sub-rede:
Bash
```bash
sudo ufw allow from 192.168.1.0/24
```
Permitir tráfego de um IP específico para uma porta específica em uma interface de rede específica:
Bash
```bash
sudo ufw allow in on eth0 from 199.1.1.1 to any port 3306
```
(Isso permite conexões na porta 3306 - MySQL - vindas do IP 199.1.1.1 na interface eth0).
**7. Adicionar regras para negar tráfego:**

Negar acesso SSH (porta 22):
Bash
```bash
sudo ufw deny ssh
```
## Ou especificando a porta:
```bash
sudo ufw deny 22/tcp
```
Negar tráfego de um IP específico:
Bash
```bash
sudo ufw deny from 192.168.1.101
```
Negar tráfego de uma sub-rede:
Bash
```bash
sudo ufw deny from 10.0.0.0/8
```
**8. Deletar regras:**

Você pode deletar regras de duas maneiras:

Pelo número da regra:

Liste as regras com números:
Bash
```bash
sudo ufw status numbered
```
Identifique o número da regra que você quer deletar e use:
Bash
```bash
sudo ufw delete [número_da_regra]
```
Exemplo: sudo ufw delete 4 (deleta a regra número 4).
Pela própria regra (exata):

Bash
```bash
sudo ufw delete allow 80/tcp
```
Você deve digitar a regra exatamente como ela foi adicionada.

**9. Limitar conexões (para proteção contra ataques de força bruta):**

O UFW pode limitar o número de conexões para uma porta específica em um curto período. Isso é útil para serviços como SSH.

Bash
```bash
sudo ufw limit ssh
```
## Ou para uma porta:
```bash
sudo ufw limit 22/tcp
```
Se um IP tentar iniciar 6 ou mais conexões em 30 segundos, ele será bloqueado por 30 minutos.

**10. Habilitar/Desabilitar logging:**

Habilitar logging:
Bash
```bash
sudo ufw logging on
```
Desabilitar logging:
Bash
```bash
sudo ufw logging off
```
Os logs do UFW são geralmente encontrados em /var/log/ufw.log.

### Considerações Importantes:

> Ordem das regras: As regras são processadas na ordem em que são adicionadas. Se uma regra "negar" estiver antes de uma regra "permitir" para o mesmo tráfego, a regra "negar" terá precedência.
> Teste sempre: Após fazer alterações significativas no seu firewall, teste o acesso para garantir que você não se bloqueou ou bloqueou serviços essenciais.
> SSH e servidores remotos: Ao configurar o UFW em um servidor remoto via SSH, SEMPRE permita a porta SSH (padrão 22) ANTES de habilitar o UFW. Caso contrário, você perderá o acesso ao servidor.
> Firewalld vs. UFW: Em algumas distribuições Linux (como RHEL/CentOS), o firewalld é o firewall padrão. Não use ufw e firewalld juntos, pois eles podem entrar em conflito. No Ubuntu, o UFW é o padrão e o recomendado.
> O UFW é uma ferramenta poderosa e essencial para a segurança de qualquer sistema Linux. Comece com as políticas padrão e adicione regras específicas conforme a necessidade, sempre com cautela e testando suas configurações.
