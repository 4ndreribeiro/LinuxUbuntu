# Guia Completo: Alterar Cores de Fonte e Fundo no Menu do GRUB no Linux
---

O **GRUB** (GRand Unified Bootloader) não permite apenas a personalização da imagem de fundo; você também pode alterar as cores do texto do menu, do fundo e do destaque para combinar com sua estética ou melhorar a legibilidade. Este guia detalha como personalizar as cores de fonte e fundo do menu do GRUB.

## Pré-requisitos
---

Antes de começar, certifique-se de ter:

1.  **Acesso Root (sudo):** Você precisará de privilégios de superusuário para modificar os arquivos de configuração do GRUB.
2.  **Conhecimento Básico do GRUB:** Familiaridade com a edição de `/etc/default/grub` e a atualização da configuração do GRUB.

## 1. Compreendendo as Definições de Cores do GRUB
---

O GRUB usa variáveis `color_normal` e `color_highlight` para definir o esquema de cores do menu. O formato é `cor_do_texto/cor_do_fundo`.

* **`color_normal`:** Define a cor do texto normal e do fundo padrão do menu.
* **`color_highlight`:** Define a cor do texto e do fundo do item de menu atualmente selecionado (destacado).

**Cores disponíveis para o GRUB:**

O GRUB suporta uma lista limitada de cores, que podem variar ligeiramente dependendo da versão do GRUB e do hardware de vídeo. As cores mais comuns e seguras incluem:

* `black`
* `blue`
* `green`
* `cyan`
* `red`
* `magenta`
* `brown` (ou `yellow` para o fundo, mas `brown` para o texto)
* `light-gray` (ou `lightgray`)
* `dark-gray` (ou `darkgray`)
* `light-blue`
* `light-green`
* `light-cyan`
* `light-red`
* `light-magenta`
* `yellow`
* `white`

Você também pode usar `transparent` como cor de fundo para permitir que a imagem de fundo apareça.

## 2. Editando o Arquivo de Configuração do GRUB (`/etc/default/grub`)
---

Este é o arquivo principal onde você define as configurações que serão usadas para gerar o arquivo de configuração final do GRUB.

1.  **Faça um backup do arquivo original:**
    ```bash
    sudo cp /etc/default/grub /etc/default/grub.bak
    ```

2.  **Abra o arquivo para edição:**
    ```bash
    sudo nano /etc/default/grub
    # ou
    sudo vi /etc/default/grub
    ```

3.  **Localize e Adicione/Modifique as Linhas de Cor:**

    Adicione ou descomente (remova o `#` do início) as seguintes linhas, ajustando as cores conforme sua preferência. Você pode colocá-las em qualquer lugar na seção global do arquivo (geralmente próximo às outras variáveis `GRUB_`).

    ```
    # Cores para o texto normal e o fundo padrão do menu
    GRUB_COLOR_NORMAL="white/black"

    # Cores para o texto e o fundo do item de menu selecionado
    GRUB_COLOR_HIGHLIGHT="black/light-green"
    ```

    **Exemplos de combinações de cores:**

    * **Fundo Transparente (se tiver imagem de fundo):**
        ```
        GRUB_COLOR_NORMAL="white/transparent"
        GRUB_COLOR_HIGHLIGHT="yellow/dark-gray"
        ```
        Certifique-se de que `GRUB_BACKGROUND` também está configurado, conforme o guia anterior.

    * **Contraste Alto:**
        ```
        GRUB_COLOR_NORMAL="light-gray/blue"
        GRUB_COLOR_HIGHLIGHT="yellow/red"
        ```

    * **Simples e Escuro:**
        ```
        GRUB_COLOR_NORMAL="light-gray/dark-gray"
        GRUB_COLOR_HIGHLIGHT="white/blue"
        ```

    **Importante:** Se você estiver usando um **tema GRUB** (definido por `GRUB_THEME`), as configurações de cor diretamente no `/etc/default/grub` podem ser **sobrescritas** pelo tema. Se esse for o caso, você precisaria editar os arquivos CSS ou de configuração do tema específico. Para este guia, estamos focando na alteração de cores sem um tema GRUB complexo.

4.  **Salve e feche o arquivo.**

## 3. Atualize a Configuração do GRUB
---

Após editar `/etc/default/grub`, você precisa gerar o arquivo de configuração final do GRUB (`/boot/grub/grub.cfg` ou `/boot/grub2/grub.cfg`). Este arquivo é lido pelo GRUB durante o boot.

### Para Ubuntu/Debian e derivados:

```bash
sudo update-grub
```

### Para Fedora/CentOS/RHEL e derivados:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 4. Reinicie o Sistema
---

Para que as alterações entrem em vigor, você deve reiniciar o seu computador.

```bash
sudo reboot
```

Ao reiniciar, você deverá ver as novas cores de fonte e fundo aplicadas ao menu do GRUB.

## Solução de Problemas Comuns
---

* **Cores não mudaram:**
    * Verifique se as linhas `GRUB_COLOR_NORMAL` e `GRUB_COLOR_HIGHLIGHT` não estão comentadas (`#`) no `/etc/default/grub`.
    * Certifique-se de ter executado o comando `sudo update-grub` ou `sudo grub2-mkconfig`.
    * Confirme se você usou os nomes de cores corretos conforme listado na seção "Cores disponíveis para o GRUB".
    * Se você estiver usando um tema GRUB, ele pode estar sobrescrevendo suas configurações de cor. Tente comentar a linha `GRUB_THEME` em `/etc/default/grub` e atualizar o GRUB novamente para ver se as cores aparecem.
* **Texto ilegível:**
    * Você escolheu uma combinação de cores com baixo contraste. Edite o `/etc/default/grub` novamente e selecione cores que contrastem melhor (ex: texto claro em fundo escuro, ou vice-versa).
* **Tela preta ou branca:**
    * Pode ser um problema de compatibilidade com a resolução (`GRUB_GFXMODE`) ou com o modo gráfico do terminal (`GRUB_TERMINAL_OUTPUT`). Certifique-se de que essas opções estejam configuradas corretamente, conforme o guia de imagem de fundo do GRUB.

A personalização das cores do menu do GRUB permite um controle ainda maior sobre a aparência do seu sistema Linux, tornando a experiência de boot mais agradável.