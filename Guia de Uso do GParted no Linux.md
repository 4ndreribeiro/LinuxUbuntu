# Guia Completo: GParted - Gerenciamento Gráfico de Partições no Linux
---

O **GParted** (GNOME Partition Editor) é uma ferramenta gráfica poderosa e fácil de usar para gerenciar partições de disco no Linux. Ele permite que você crie, exclua, redimensione, mova, copie e verifique partições, além de formatá-las com diversos sistemas de arquivos. O GParted é especialmente útil para quem prefere uma interface visual em vez de comandos de linha para operações de disco.

## O que é o GParted?
---

GParted é um editor de partições de disco de código aberto que suporta uma vasta gama de sistemas de arquivos, incluindo `ext2/3/4`, `FAT16/32`, `NTFS`, `XFS`, `Btrfs`, e muitos outros. Ele opera diretamente nos dispositivos de bloco do disco, permitindo manipulações de baixo nível nas partições.

Sendo uma ferramenta gráfica, o GParted é ideal para:
* Usuários que não se sentem confortáveis com ferramentas de linha de comando como `fdisk` ou `gdisk`.
* Operações complexas de redimensionamento e movimentação de partições.
* Visualização clara da disposição das partições no disco.
* Preparação de discos para a instalação de novos sistemas operacionais ou para adicionar armazenamento.

**Importante:** Assim como qualquer ferramenta de particionamento, o GParted lida com dados críticos do disco. **Sempre faça backup de seus dados importantes antes de realizar qualquer operação de particionamento**, pois erros ou interrupções podem levar à perda de dados.

## Como o GParted funciona?
---

O GParted funciona detectando os discos rígidos e seus layouts de partição. Ele exibe uma representação visual do disco, mostrando o tamanho de cada partição, seu tipo de sistema de arquivos e o espaço livre.

Para realizar operações, o GParted primeiro "marca" as ações que você deseja realizar (como redimensionar, criar, etc.). Essas ações são colocadas em uma "fila de operações". As alterações reais no disco só são aplicadas quando você explicitamente confirma e clica no botão "Aplicar todas as operações". Isso permite que você revise suas ações antes que elas sejam gravadas no disco, reduzindo o risco de erros.

Geralmente, o GParted requer que as partições não estejam montadas para serem manipuladas (especialmente para redimensionamento ou movimentação). Por essa razão, é comum executar o GParted a partir de um **Live CD/USB** (como o Ubuntu Live USB) para gerenciar o disco do sistema principal.

## Instalação do GParted no Linux
---

O GParted geralmente não vem pré-instalado em todas as distribuições, mas está amplamente disponível nos repositórios.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install gparted
```

### No Fedora/CentOS/RHEL:

```bash
sudo dnf install gparted
```

### No Arch Linux:

```bash
sudo pacman -S gparted
```

## Como Usar o GParted (Básico)
---

Após a instalação, você pode iniciar o GParted. Normalmente, ele requer privilégios de root para ser executado.

1.  **Abrir o GParted:**
    * Procure por "GParted" no menu de aplicativos do seu ambiente de desktop.
    * Ou execute-o a partir do terminal:
        ```bash
        sudo gparted
        ```
    * Você pode ser solicitado a digitar sua senha de superusuário.

2.  **Selecionar o Disco:**
    * Na parte superior direita da janela do GParted, haverá um menu suspenso que lista todos os discos detectados no seu sistema (ex: `/dev/sda`, `/dev/sdb`). Selecione o disco que você deseja gerenciar.

3.  **Visualizar e Analisar Partições:**
    * A interface principal mostrará um mapa visual do disco selecionado, com cada partição representada por um bloco. Informações como tipo de sistema de arquivos, tamanho, espaço usado e livre serão exibidas.

4.  **Operações Comuns:**

    * **Criar uma Nova Partição:**
        1.  Clique com o botão direito em um espaço **não alocado** (espaço livre) no disco.
        2.  Selecione "New".
        3.  Configure o tamanho, o tipo de partição (primária/estendida/lógica para MBR; não aplicável para GPT), o tipo de sistema de arquivos (ex: `ext4`), e o rótulo (Label) se desejar.
        4.  Clique em "Add".

    * **Redimensionar/Mover uma Partição:**
        1.  Clique com o botão direito na partição que deseja redimensionar ou mover.
        2.  Selecione "Resize/Move".
        3.  Uma janela se abrirá com um controle deslizante. Você pode arrastar as bordas da partição ou digitar os valores para ajustar o novo tamanho e a posição. Certifique-se de deixar espaço livre adjacente para redimensionar.
        4.  Clique em "Resize/Move".

    * **Excluir uma Partição:**
        1.  Clique com o botão direito na partição que deseja excluir.
        2.  Selecione "Delete".

    * **Formatar uma Partição:**
        1.  Clique com o botão direito na partição que deseja formatar.
        2.  Selecione "Format to".
        3.  Escolha o novo sistema de arquivos (ex: `ext4`, `NTFS`).
        4.  Clique em "Format".

    * **Copiar/Colar Partição:**
        1.  Clique com o botão direito na partição que deseja copiar e selecione "Copy".
        2.  Clique com o botão direito em um espaço não alocado no mesmo disco ou em outro disco e selecione "Paste".
        3.  Ajuste o tamanho e a posição se necessário.
        4.  Clique em "Paste".

    * **Verificar uma Partição (fsck):**
        1.  Clique com o botão direito na partição.
        2.  Selecione "Check". Isso pode ajudar a corrigir erros no sistema de arquivos.

5.  **Aplicar as Operações:**
    * Após adicionar todas as operações à fila (elas aparecerão na parte inferior da janela principal), clique no botão **"Apply All Operations" (ícone de marca de verificação verde ou "✓")** na barra de ferramentas.
    * O GParted solicitará uma confirmação, avisando que os dados podem ser perdidos. Confirme para iniciar as operações reais no disco.
    * Um pop-up mostrará o progresso das operações.

### Dicas Importantes:
* **Desmontar Partições:** Antes de realizar operações como redimensionar ou mover, o GParted geralmente exigirá que a partição esteja desmontada. Se a partição estiver em uso (por exemplo, sua partição raiz), você terá que usar um Live CD/USB para realizar a operação.
* **Não Interrompa:** Nunca desligue o computador ou feche o GParted enquanto as operações estiverem sendo aplicadas. Isso pode causar corrupção de dados.
* **Live CD/USB:** Para gerenciar a partição onde seu sistema Linux está instalado, você **deve** inicializar o computador a partir de um Live CD/USB (como um Live CD do Ubuntu) e executar o GParted a partir dele.

O GParted é uma ferramenta indispensável para qualquer usuário Linux que precise gerenciar o armazenamento de forma visual e segura.
