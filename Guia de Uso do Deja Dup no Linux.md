# Guia Completo: Deja Dup - Ferramenta de Backup no Linux
---

O **Deja Dup** é uma ferramenta de backup simples e fácil de usar para sistemas Linux, que atua como uma interface gráfica para o poderoso utilitário de backup `duplicity`. Ele é projetado para tornar o processo de backup e restauração de dados o mais descomplicado possível, focando na automação e na recuperação fácil.

## O que é o Deja Dup?
---

O Deja Dup é uma solução de backup de código aberto que se integra perfeitamente com ambientes de desktop GNOME (e outros, como Unity, Cinnamon, MATE). Sua principal característica é a simplicidade. Ele permite que usuários criem backups criptografados, incrementais e compactados de seus arquivos pessoais ou de todo o sistema para vários destinos, como discos locais, unidades de rede ou serviços de armazenamento em nuvem.

### Principais recursos:

* **Simplicidade:** Interface gráfica intuitiva, eliminando a necessidade de comandos complexos.
* **Criptografia:** Seus backups podem ser criptografados com senha para garantir a privacidade dos dados.
* **Incrementais:** Após o primeiro backup completo, o Deja Dup faz apenas backups incrementais, salvando apenas as alterações, o que economiza espaço e tempo.
* **Compactação:** Os dados são compactados para reduzir o tamanho do backup.
* **Agendamento:** Você pode configurar backups automáticos e regulares.
* **Diversos Destinos:** Suporta armazenamento local, em rede (NFS, SMB), SFTP, Google Drive, e S3.
* **Restauração Fácil:** Permite restaurar arquivos individualmente ou o backup completo, mesmo de versões anteriores.

## Como o Deja Dup funciona?
---

O Deja Dup utiliza o `duplicity` nos bastidores. `duplicity` funciona criando volumes de arquivos compactados e criptografados.

1.  **Backup Completo Inicial:** Na primeira vez que você executa um backup, o Deja Dup cria uma cópia completa de todos os arquivos selecionados.
2.  **Backups Incrementais:** Para os backups subsequentes, ele compara os arquivos atuais com a versão anterior e salva apenas as diferenças. Isso cria uma cadeia de backups, onde cada um depende do anterior.
3.  **Criptografia e Compressão:** Antes de serem salvos, os dados são compactados e criptografados (se você optar por uma senha) para economizar espaço e proteger sua privacidade.
4.  **Meta-informações:** O Deja Dup também armazena metadados sobre os arquivos (permissões, datas, etc.) para garantir uma restauração precisa.

Quando você precisa restaurar, o Deja Dup usa essa cadeia de backups e metadados para reconstruir os arquivos na versão desejada.

## Instalação do Deja Dup no Linux
---

O Deja Dup é amplamente disponível nos repositórios da maioria das distribuições Linux.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install deja-dup
```

### No Fedora/CentOS/RHEL:

```bash
sudo dnf install deja-dup
```

### No Arch Linux:

```bash
sudo pacman -S deja-dup
```

## Como Usar o Deja Dup (Básico)
---

Após a instalação, você pode encontrar o Deja Dup no menu de aplicativos (geralmente sob o nome "Backups" ou "Deja Dup Backups").

1.  **Abrir o Deja Dup:**
    * Procure por "Backups" ou "Deja Dup" no menu de aplicativos.

2.  **Configurar o Primeiro Backup:**
    * Na tela inicial, você verá opções para "Create My First Backup" (Criar Meu Primeiro Backup) ou "Restore From a Previous Backup" (Restaurar de um Backup Anterior).
    * Clique em "Create My First Backup".

3.  **Pastas para Fazer Backup (Folders to be backed up):**
    * Aqui você seleciona quais diretórios você deseja incluir no backup. Por padrão, sua pasta pessoal (`/home/seu_usuario`) é geralmente selecionada.
    * Você também pode excluir pastas específicas (e.g., `Downloads`, `Trash`) na seção "Folders to ignore" (Pastas a ignorar) para economizar espaço e tempo.

4.  **Local de Armazenamento (Storage Location):**
    * Escolha onde seus backups serão salvos. As opções comuns incluem:
        * **Local Folder:** Um disco rígido externo ou outra partição no seu computador.
        * **Network Server:** Um servidor de rede (NFS, SMB/CIFS).
        * **Google Drive:** Para armazenamento em nuvem do Google.
        * **Local Storage (via SFTP/SSH):** Para armazenar em um servidor remoto via SSH.
        * **Amazon S3:** Para armazenamento em nuvem da AWS.
    * Selecione o local e, se necessário, forneça as credenciais ou o caminho da pasta.

5.  **Programar Backups (Scheduling):**
    * Configure a frequência dos backups (Daily, Weekly, Fortnightly, Monthly) e por quanto tempo você deseja manter os backups antigos (Forever, Until space is low, For at least one year, etc.).
    * **Recomendado:** Ative o agendamento de backups para que eles ocorram automaticamente.

6.  **Criptografar Backup (Security):**
    * Você pode optar por proteger seu backup com uma senha. **É altamente recomendado que você use uma senha forte.** Sem essa senha, ninguém poderá acessar seus dados criptografados. **Não perca essa senha!**

7.  **Iniciar Backup:**
    * Após configurar tudo, clique em "Back Up Now" (Fazer Backup Agora) para iniciar o processo. A primeira vez levará mais tempo, pois é um backup completo.

### Restauração de Dados:

1.  **Abrir o Deja Dup** e clique em "Restore From a Previous Backup".
2.  **Selecionar o Local do Backup:** Indique onde seu backup está armazenado.
3.  **Escolher a Data:** Você pode escolher de qual data/versão do backup deseja restaurar.
4.  **Selecionar Arquivos/Pastas:** Navegue pelas pastas do backup e selecione os arquivos ou diretórios que deseja restaurar. Você pode restaurar para o local original ou para um novo local.
5.  **Iniciar Restauração:** Confirme e inicie o processo de restauração. Se o backup estiver criptografado, você precisará fornecer a senha.

O Deja Dup torna o processo de backup e restauração no Linux muito mais acessível para usuários de todos os níveis, garantindo que seus dados estejam seguros e recuperáveis.
