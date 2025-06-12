# Guia Completo: Dual Boot Ubuntu Linux e Windows 10 no VirtualBox (Host Linux)
---

A configuração de um sistema **Dual Boot** (dois sistemas operacionais na mesma máquina) é comum para usuários que precisam de ambos os ambientes. Ao fazer isso em uma **Máquina Virtual (MV) no VirtualBox**, você ganha flexibilidade para experimentar e desenvolver sem afetar seu sistema operacional principal (host). Este guia detalha o processo de instalação do **Windows 10** e do **Ubuntu Linux** em Dual Boot dentro de uma MV do VirtualBox, rodando em um host Linux.

## Por que Dual Boot em uma VM?
---

* **Segurança:** Experimente novas configurações ou softwares sem risco para o sistema host.
* **Flexibilidade:** Alterne facilmente entre Windows e Linux sem reiniciar o computador físico.
* **Desenvolvimento/Testes:** Crie um ambiente isolado para testar compatibilidade de aplicações em diferentes sistemas.
* **Recursos Otimizados:** Aloca recursos (CPU, RAM, Disco) dinamicamente para a MV, controlando o impacto no host.

## Pré-requisitos
---

Para seguir este guia, você precisará dos seguintes itens:

1.  **Sistema Host Linux:** Uma distribuição Linux (Ubuntu, Fedora, etc.) com o **VirtualBox** instalado.
    * Se não tiver o VirtualBox, instale-o:
        ```bash
        # Ubuntu/Debian
        sudo apt update
        sudo apt install virtualbox

        # Fedora/CentOS/RHEL
        sudo dnf install virtualbox
        ```
2.  **Imagem ISO do Windows 10:** Baixe a imagem ISO oficial do Windows 10 da Microsoft ou de uma fonte confiável.
3.  **Imagem ISO do Ubuntu Linux:** Baixe a imagem ISO da versão LTS (Long Term Support) mais recente do Ubuntu (ex: Ubuntu 22.04 LTS) do site oficial [ubuntu.com/download](https://ubuntu.com/download).
4.  **Recursos do Host:** Garanta que seu computador host tenha recursos suficientes (pelo menos **8GB de RAM** para uma experiência confortável, e espaço em disco para a MV).

## 1. Criação da Máquina Virtual no VirtualBox
---

Primeiro, vamos criar uma nova máquina virtual que será o ambiente para o dual boot.

1.  **Abra o VirtualBox.**
2.  Clique em **"Nova"** (ou "New") para criar uma nova máquina virtual.
3.  **Nome e Sistema Operacional:**
    * **Nome:** Dê um nome descritivo (ex: `DualBoot_Win10_Ubuntu`).
    * **Pasta da Máquina:** Escolha onde os arquivos da MV serão armazenados.
    * **Tipo:** `Microsoft Windows`.
    * **Versão:** `Windows 10 (64-bit)` (ou 32-bit, dependendo da sua ISO).
    * Clique em "Próximo" (Next).
4.  **Memória RAM:**
    * Aloque pelo menos **4096 MB (4GB)** para a MV. Ambos os sistemas se beneficiarão, e você poderá alternar entre eles.
    * Clique em "Próximo".
5.  **Disco Rígido:**
    * Selecione **"Criar um novo disco rígido virtual agora"**.
    * **Tipo de Arquivo de Disco Rígido:** Deixe como `VDI (VirtualBox Disk Image)`.
    * **Alocação de Armazenamento:** Selecione **"Dinamicamente alocado"**. Isso economiza espaço no seu host, crescendo o disco virtual conforme necessário.
    * **Tamanho do Disco:** Aloque pelo menos **60 GB a 80 GB** para a MV. Isso dará espaço suficiente para ambos os sistemas operacionais e alguns aplicativos.
    * Clique em "Criar" (Create).

A MV será criada. Agora, vamos ajustar algumas configurações adicionais antes de instalar.

### Configurações Adicionais da MV:

1.  Com a MV selecionada, clique em **"Configurações"** (Settings).
2.  **Sistema (System) -> Placa-Mãe (Motherboard):**
    * Certifique-se de que a **Ordem de Boot** inclua `Óptico` e `Disco Rígido`.
3.  **Sistema (System) -> Processador (Processor):**
    * Aloque pelo menos **2 ou 4 CPUs** para a MV, se seu host tiver. Isso melhorará o desempenho.
4.  **Tela (Display) -> Tela (Screen):**
    * Aumente a **Memória de Vídeo** para `128 MB` ou `256 MB`.
    * Habilite a **Aceleração 3D**.
5.  **Armazenamento (Storage):**
    * Sob **"Controlador: IDE"** ou **"Controlador: SATA"**, clique no ícone de "Disco Óptico Vazio".
    * À direita, clique no ícone de disco pequeno e selecione **"Escolher um arquivo de disco"**.
    * Navegue e selecione o arquivo **ISO do Windows 10**.
6.  Clique em "OK" para salvar as configurações.

## 2. Instalação do Windows 10
---

Agora, vamos instalar o Windows 10 na primeira partição do nosso disco virtual.

1.  Com a MV selecionada no VirtualBox, clique em **"Iniciar"** (Start).
2.  O instalador do Windows 10 será inicializado a partir da ISO.
3.  **Siga o processo de instalação do Windows 10:**
    * Selecione o idioma, formato de hora e teclado.
    * Clique em "Instalar agora".
    * Insira sua chave de produto (se solicitada, ou "Não tenho uma chave de produto" para ativar depois).
    * Aceite os termos de licença.
    * Na tela "Que tipo de instalação você deseja?", selecione **"Personalizada: Instalar apenas o Windows (avançado)"**.
    * **Criação de Partição:**
        * Você verá o disco virtual completo (ex: 60GB de espaço não alocado).
        * Selecione o "Espaço não alocado" e clique em **"Novo"**.
        * Defina um tamanho para a partição do Windows (ex: **30000 MB para 30 GB**). O Windows criará partições adicionais (Recuperação, Sistema, MSR) automaticamente.
        * Selecione a partição principal do Windows que você acabou de criar e clique em "Avançar".
    * A instalação do Windows 10 começará. Siga as instruções para configurar seu usuário e preferências.
4.  **Após a instalação do Windows 10, instale as Guest Additions do VirtualBox:**
    * No menu do VirtualBox (da janela da MV), vá em **"Dispositivos" (Devices) -> "Inserir imagem de CD de Adições de Convidado..." (Insert Guest Additions CD image...)**.
    * Dentro do Windows 10, abra o "Explorador de Arquivos" e execute o instalador das Guest Additions do CD virtual (geralmente `VBoxWindowsAdditions.exe`).
    * Siga o processo de instalação e reinicie o Windows 10 quando solicitado.
    * As Guest Additions melhoram a integração (resolução de tela, clipboard compartilhado, etc.).

## 3. Instalação do Ubuntu Linux (em Dual Boot)
---

Agora, vamos instalar o Ubuntu na partição restante do disco virtual.

1.  **Remova a ISO do Windows 10:**
    * No menu do VirtualBox (da janela da MV), vá em **"Dispositivos" (Devices) -> "Unidades Ópticas" (Optical Drives)**.
    * Desmarque a imagem ISO do Windows 10 (ou selecione "Remover disco da unidade virtual").
2.  **Insira a ISO do Ubuntu:**
    * Vá em **"Dispositivos" (Devices) -> "Unidades Ópticas" (Optical Drives)** novamente e selecione **"Escolher/Criar uma Imagem de Disco..."**.
    * Navegue e selecione o arquivo **ISO do Ubuntu Linux**.
3.  **Reinicie a Máquina Virtual:**
    * No menu do VirtualBox, vá em **"Máquina" (Machine) -> "Reiniciar" (Reset)**.
    * A MV deve iniciar a partir da ISO do Ubuntu.
4.  **Inicie o Instalador do Ubuntu:**
    * No menu de boot do Ubuntu, selecione "Try or Install Ubuntu".
    * Após o ambiente Live carregar, clique em **"Instalar Ubuntu"** (Install Ubuntu).
5.  **Siga o processo de instalação do Ubuntu:**
    * Selecione o idioma e o layout do teclado.
    * Na tela "Atualizações e outros softwares", selecione "Instalação normal" e marque "Baixar atualizações durante a instalação" e "Instalar software de terceiros" (opcional).
    * **Tipo de Instalação (CRUCIAL para Dual Boot):**
        * Nesta tela, o instalador do Ubuntu detectará o Windows Boot Manager.
        * Selecione a opção **"Instalar o Ubuntu ao lado do Windows Boot Manager"** (Install Ubuntu alongside Windows Boot Manager). Esta é a forma mais fácil de configurar o dual boot.
        * O instalador apresentará um controle deslizante para você ajustar o tamanho das partições para o Windows e para o Ubuntu. Arraste-o para alocar espaço para o Ubuntu (ex: 30GB).
        * Clique em "Instalar agora". O instalador fará o particionamento automático e instalará o GRUB.
    * Continue com as configurações de região, usuário e senha.
    * Aguarde a conclusão da instalação.
6.  **Reinicie quando solicitado.** Remova a ISO do Ubuntu (como fez com a do Windows) e a MV reiniciará.

## 4. Gerenciamento do Dual Boot (GRUB)
---

Após a reinicialização, o **GRUB (GRand Unified Bootloader)** será o primeiro a carregar. Ele detectará tanto o Ubuntu quanto o Windows Boot Manager.

* Você verá um menu com opções como:
    * `Ubuntu` (opção padrão para iniciar o Ubuntu)
    * `Advanced options for Ubuntu`
    * `Windows Boot Manager (on /dev/sdaX)` (para iniciar o Windows 10)
    * `System setup`

* Use as setas do teclado para selecionar o sistema operacional que deseja iniciar e pressione `Enter`.

## 5. Instalar as Guest Additions do VirtualBox no Ubuntu
---

Para uma melhor experiência no Ubuntu (resolução de tela, clipboard compartilhado, arrastar e soltar, etc.):

1.  Inicie o Ubuntu na MV.
2.  No menu do VirtualBox (da janela da MV), vá em **"Dispositivos" (Devices) -> "Inserir imagem de CD de Adições de Convidado..." (Insert Guest Additions CD image...)**.
3.  No Ubuntu, uma janela pode aparecer perguntando se você deseja executar o instalador. Clique em "Executar" (Run) ou "Run Software".
4.  Se não aparecer, abra o gerenciador de arquivos, navegue até a unidade de CD virtual e execute o script `VBoxLinuxAdditions.run` no terminal:
    ```bash
    cd /media/<seu_usuario>/VBox_GAs_<versão>/
    sudo ./VBoxLinuxAdditions.run
    ```
5.  Siga as instruções do instalador e reinicie o Ubuntu quando concluído.

## Solução de Problemas Comuns
---

* **GRUB não aparece ou não detecta Windows/Ubuntu:**
    * Certifique-se de que o tipo de instalação no Ubuntu foi "Instalar ao lado do Windows Boot Manager".
    * Se o GRUB não detectar o Windows, inicie o Ubuntu e execute: `sudo update-grub`. Isso fará com que o GRUB procure por outros sistemas operacionais.
* **Problemas de resolução de tela ou integração:**
    * Verifique se as Guest Additions foram instaladas corretamente em ambos os sistemas.
    * Certifique-se de que a aceleração 3D está habilitada nas configurações da MV do VirtualBox.
* **Lentidão na MV:**
    * Aumente a RAM e o número de CPUs alocados para a MV (nas configurações da MV).
    * Verifique se seu disco rígido host não está fragmentado ou cheio.
    * Instale as Guest Additions em ambos os sistemas.

Com este guia, você deve ser capaz de configurar com sucesso um ambiente de dual boot com Windows 10 e Ubuntu Linux dentro do VirtualBox, utilizando seu sistema Linux como host.
