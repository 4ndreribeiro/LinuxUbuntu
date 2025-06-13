# Guia Completo: Instalação e Uso do Evolution Email no Linux
---

O **Evolution Email and Calendar** (geralmente chamado apenas de Evolution) é um software de gerenciamento de informações pessoais (PIM - Personal Information Manager) de código aberto para o ambiente de desktop GNOME e outros sistemas Linux. Ele oferece uma solução completa para e-mail, calendário, contatos e memos, atuando como uma alternativa robusta e rica em recursos a clientes de e-mail proprietários.

## O que é o Evolution Email?
---

O Evolution é mais do que um simples cliente de e-mail; ele é uma suíte de comunicação e organização. Ele foi projetado para integrar todas as suas necessidades de comunicação pessoal e profissional em uma única aplicação.

### Principais Características:

* **Cliente de E-mail:** Suporta múltiplos protocolos (IMAP, POP3, SMTP) e criptografia (SSL/TLS). Oferece recursos avançados como filtragem de spam (via SpamAssassin e outros), regras de e-mail, pastas virtuais e pesquisa avançada.
* **Calendário:** Permite gerenciar múltiplos calendários, agendar compromissos, criar eventos recorrentes e integrar com servidores de calendário (CalDAV, Google Calendar, Exchange).
* **Livro de Endereços (Contatos):** Gerencia contatos, grupos e listas de e-mail, com suporte a LDAP e integração com diretórios.
* **Memos e Listas de Tarefas:** Para anotações rápidas e gerenciamento de tarefas pessoais.
* **Integração com Microsoft Exchange:** Para usuários corporativos, o Evolution oferece bom suporte para se conectar a servidores Microsoft Exchange (dependendo da versão e configuração do Exchange).
* **Segurança:** Suporte a GPG para criptografia e assinatura de e-mails.

## Por que usar o Evolution Email no Linux?
---

* **Solução Completa:** Centraliza e-mail, calendário e contatos, melhorando a produtividade.
* **Código Aberto:** Gratuito, flexível e transparente.
* **Integração com o Desktop GNOME:** Desenvolvido para o GNOME, oferece uma experiência de usuário coesa e nativa.
* **Recursos Avançados:** Oferece funcionalidades robustas adequadas tanto para uso pessoal quanto profissional.

## 1. Instalação do Evolution Email no Linux
---

O Evolution está disponível nos repositórios padrão da maioria das distribuições Linux.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install evolution
```
* `evolution`: O pacote principal do Evolution.
* Você também pode instalar plugins adicionais se precisar de funcionalidades específicas (ex: `evolution-ews` para Exchange Web Services).
    ```bash
    sudo apt install evolution-ews
    ```

### No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install evolution
```
* Para o plugin Exchange:
    ```bash
    sudo dnf install evolution-ews
    ```

### No Arch Linux e derivados:

```bash
sudo pacman -S evolution
```
* Para o plugin Exchange:
    ```bash
    sudo pacman -S evolution-ews
    ```

## 2. Primeira Inicialização e Configuração
---

Após a instalação, abra o Evolution a partir do menu de aplicativos do seu desktop. Na primeira vez, o "Assistente de Configuração da Conta" será iniciado.

1.  **Assistente de Configuração da Conta:**
    * Clique em "Próximo".
    * O Evolution pode tentar preencher automaticamente algumas informações se você já tiver uma conta conectada ao seu ambiente desktop.
    * **Identidade:** Preencha seu nome completo e endereço de e-mail.
    * **Receber E-mail:**
        * **Tipo de servidor:** Escolha o tipo de servidor de e-mail (IMAP, POP3, Microsoft Exchange, Google, etc.). **IMAP é geralmente recomendado** pois mantém os e-mails no servidor.
        * **Servidor:** Endereço do servidor de entrada (ex: `imap.seuservidor.com` ou `imap.gmail.com`).
        * **Nome de usuário:** Seu nome de usuário de e-mail.
        * **Criptografia:** Escolha a criptografia (SSL/TLS é o mais seguro).
        * **Autenticação:** Escolha o método de autenticação (geralmente "Senha" ou "OAuth2" para Google).
    * **Enviar E-mail (SMTP):**
        * **Servidor:** Endereço do servidor de saída (ex: `smtp.seuservidor.com` ou `smtp.gmail.com`).
        * **Criptografia:** Escolha a criptografia (SSL/TLS).
        * **Autenticação:** Escolha o método de autenticação (geralmente o mesmo do servidor de entrada).
    * **Opções da Conta:** Dê um nome à sua conta para fácil identificação.
    * Clique em "Próximo" e "Aplicar" para finalizar a configuração da conta.

2.  **Configurar Calendário e Contatos (Opcional):**
    * Após configurar o e-mail, você pode configurar o calendário e os contatos.
    * No menu `Arquivo -> Novo -> Calendário` ou `Arquivo -> Novo -> Livro de Endereços`.
    * Você pode adicionar calendários CalDAV, Google Calendars, ou conectar a servidores de contatos.

## 3. Usando o Evolution
---

A interface do Evolution é dividida em diferentes seções para facilitar a navegação.

* **Barra Lateral Esquerda:** Contém atalhos para E-mail, Calendário, Contatos, Memos e Tarefas.
* **Visão de E-mail:** Exibe suas pastas de e-mail, mensagens e o painel de visualização.
    * **Escrever Novo E-mail:** Clique no botão "Novo" ou "Escrever Mensagem".
    * **Filtrar/Pesquisar:** Use a barra de pesquisa ou as ferramentas de filtro para encontrar mensagens.
    * **Regras de Mensagem:** Crie regras para organizar automaticamente seus e-mails.
* **Visão de Calendário:** Permite criar eventos, definir lembretes e visualizar sua agenda.
* **Visão de Contatos:** Gerencie seus contatos e listas.

## 4. Dicas e Solução de Problemas
---

* **Problemas de Conexão:**
    * Verifique suas credenciais (nome de usuário, senha) e os endereços dos servidores IMAP/POP3 e SMTP.
    * Confirme se as portas e métodos de criptografia/autenticação estão corretos para o seu provedor de e-mail.
    * Verifique se o seu firewall está permitindo as conexões de saída (portas 143/993 para IMAP, 110/995 para POP3, 587/465 para SMTP).
* **Spam:**
    * O Evolution se integra com o SpamAssassin (se instalado). Você pode "treinar" o filtro marcando mensagens como spam.
* **Backup de Dados:**
    * Os dados do Evolution são armazenados em `~/.local/share/evolution/`. Para fazer backup completo, você pode copiar este diretório.
* **Sincronização:**
    * Para sincronizar calendários e contatos com serviços como Google ou Microsoft Exchange, você precisará dos plugins e configurar as respectivas contas nas seções de calendário/contatos.

O Evolution Email é uma ferramenta poderosa e versátil que pode se tornar seu hub central para gerenciamento de comunicações e organização pessoal no Linux.
