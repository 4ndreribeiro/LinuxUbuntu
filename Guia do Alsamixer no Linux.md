# Guia Completo: Alsamixer - Controle de Áudio no Linux
---

O **Alsamixer** é uma ferramenta de mixer de áudio baseada em terminal para Linux, que permite controlar os níveis de volume e outras configurações de á áudio de sua placa de som. Faz parte do pacote ALSA (Advanced Linux Sound Architecture), que é o framework de áudio padrão na maioria das distribuições Linux modernas.

## O que é o Alsamixer?
---

O Alsamixer é uma interface de usuário simples e leve para as configurações de áudio do ALSA. Diferente de mixers gráficos mais complexos (como PulseAudio Volume Control), o Alsamixer oferece controle direto sobre os canais de áudio da sua placa de som, incluindo:

* **Controle de Volume:** Ajusta o volume de saída principal, fones de ouvido, microfone, etc.
* **Mute/Unmute:** Ativa ou desativa canais de áudio.
* **Seleção de Entrada/Saída:** Em algumas placas de som, permite alternar entre diferentes portas de entrada (line-in, microfone) ou saída (line-out, fones de ouvido).
* **Ganho do Microfone (Mic Boost):** Aumenta a sensibilidade do microfone.
* **Loopback:** Controla se o áudio de entrada é roteado diretamente para a saída.

Ele é particularmente útil em ambientes de servidor sem interface gráfica, em sistemas com poucos recursos ou para diagnosticar problemas de áudio de baixo nível.

## Instalação do Alsamixer
---

O Alsamixer geralmente já vem pré-instalado como parte do ALSA na maioria das distribuições Linux. Se por algum motivo não estiver, você pode instalá-lo através do gerenciador de pacotes da sua distribuição.

### No Ubuntu/Debian e derivados:

```bash
sudo apt update
sudo apt install alsa-utils
```

### No Fedora/CentOS/RHEL e derivados:

```bash
sudo dnf install alsa-utils
```

## Como Usar o Alsamixer
---

Para iniciar o Alsamixer, simplesmente digite `alsamixer` no terminal:

```bash
alsamixer
```

### Interface do Alsamixer:

Ao iniciar o Alsamixer, você verá uma interface baseada em caracteres que mostra seus controles de áudio.

* **Linha Superior:**
    * `Card:`: Nome da sua placa de som ou dispositivo de áudio.
    * `Chip:`: Modelo do chip de áudio.
    * `View:`: Visão atual (playback, capture, all).
    * `Item:`: Nome do controle atualmente selecionado.

* **Controles de Volume:**
    * Representados por barras verticais. `00` indica que o canal está mutado (com som desativado).
    * `MM` (Mute Master): Indica que o canal mestre de saída está mutado.
    * `OO` (Output): Indica que o canal está ativo.

### Navegação e Comandos:

* **`F1`:** Ajuda (nem sempre visível, mas funciona).
* **`F2`:** Exibe informações do sistema (não comum).
* **`F3`:** Alterna para a visão de **Playback** (saída de áudio).
* **`F4`:** Alterna para a visão de **Capture** (entrada de áudio, microfone).
* **`F5`:** Alterna para a visão **All** (todos os controles).
* **`Esquerda` / `Direita` (setas):** Move entre os controles de áudio (Master, Headphone, Speaker, Mic, etc.).
* **`Cima` / `Baixo` (setas):** Ajusta o volume do controle selecionado.
* **`M`:** Ativa/desativa o áudio (mute/unmute) para o controle selecionado.
* **`Espaço`:** Em alguns controles, pode ser usado para ativar/desativar uma opção ou alternar entre fontes.
* **`Tab`:** Alterna entre as placas de som disponíveis (se você tiver mais de uma).
* **`Esc`:** Sai do Alsamixer.

### Salvar Configurações:

As alterações feitas no Alsamixer são geralmente temporárias e podem ser perdidas após uma reinicialização. Para salvar as configurações permanentemente:

```bash
sudo alsactl store
```
Este comando salva o estado atual do mixer ALSA em um arquivo (geralmente `/var/lib/alsa/asound.state` ou `/etc/asound.state`), que será carregado automaticamente na próxima inicialização do sistema.

## Exemplos de Uso
---

1.  **Ajustar o volume principal de saída:**
    * Execute `alsamixer`.
    * Use as setas `Esquerda` / `Direita` para selecionar o controle `Master`.
    * Use as setas `Cima` / `Baixo` para ajustar o volume.
    * Pressione `M` para mutar/desmutar.
    * Pressione `Esc` para sair.
    * Salve as alterações: `sudo alsactl store`

2.  **Ajustar o volume do microfone e habilitar Mic Boost:**
    * Execute `alsamixer`.
    * Pressione `F4` para ir para a visão de `Capture`.
    * Use as setas `Esquerda` / `Direita` para selecionar o controle `Mic` ou `Capture`.
    * Use as setas `Cima` / `Baixo` para ajustar o nível.
    * Se houver um controle `Mic Boost`, selecione-o e ajuste o ganho.
    * Pressione `M` para ativar/desativar o microfone.
    * Pressione `Esc` para sair.
    * Salve as alterações: `sudo alsactl store`

3.  **Alternar entre placas de som:**
    * Execute `alsamixer`.
    * Pressione `Tab` para alternar entre as placas de som detectadas no seu sistema.

## Considerações Finais
---

* **ALSA vs. PulseAudio:** Em muitos desktops Linux, o PulseAudio (ou PipeWire) é a camada de som de alto nível que gerencia o áudio. O Alsamixer controla as configurações de baixo nível da placa de som através do ALSA, e o PulseAudio opera sobre o ALSA. Às vezes, as configurações do Alsamixer podem ser "ignoradas" ou redefinidas pelo PulseAudio se não forem salvas corretamente.
* **Solução de Problemas:** O Alsamixer é uma ferramenta excelente para depurar problemas de áudio, especialmente quando há um canal mutado ou um volume muito baixo em um nível que mixers gráficos podem não expor facilmente.
* **Sistemas Embarcados/Servidores:** Em sistemas sem GUI, o Alsamixer é a ferramenta principal para gerenciar as configurações de volume.

O Alsamixer é uma ferramenta simples, mas poderosa e essencial para quem precisa de controle granular sobre o áudio no Linux, especialmente em ambientes de terminal.
