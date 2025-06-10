# Guia Completo: Resetar Senhas Esquecidas de Usuários e do Root no Linux
---

Esquecer a senha de um usuário ou, pior ainda, a senha de `root` no Linux pode ser uma situação frustrante, mas felizmente, é possível resetá-las. O processo varia um pouco dependendo se você tem acesso a outro usuário com privilégios `sudo` ou se a senha do `root` é a única que você esqueceu e precisa recuperá-la diretamente.

**Atenção:** Estes procedimentos exigem acesso físico ao servidor/máquina e, em alguns casos, acesso ao menu de boot do GRUB. Realize-os com cuidado, pois erros podem afetar a integridade do sistema.

## 1. Resetar Senha de um Usuário Comum (com acesso `sudo`)
---

Se você tem acesso a um usuário com privilégios `sudo` (que pode executar comandos como administrador), resetar a senha de outro usuário comum é o procedimento mais simples.

1.  **Abra o Terminal.**
2.  **Use o comando `passwd` com `sudo`:**
    ```bash
    sudo passwd nome_do_usuario
    ```
    * Substitua `nome_do_usuario` pelo nome do usuário cuja senha você quer resetar.
3.  **Digite sua senha de `sudo`** quando solicitado.
4.  **Digite a nova senha** para o `nome_do_usuario` e confirme-a.
    ```
    New password:
    Retype new password:
    passwd: password updated successfully
    ```
    A senha do usuário será resetada imediatamente.

## 2. Resetar Senha do Root (com acesso `sudo`)
---

Da mesma forma, se você tem acesso a um usuário com privilégios `sudo`, pode resetar a senha do usuário `root` (se o login de `root` estiver habilitado) de forma direta.

1.  **Abra o Terminal.**
2.  **Use o comando `passwd` para o usuário `root`:**
    ```bash
    sudo passwd root
    ```
3.  **Digite sua senha de `sudo`** quando solicitado.
4.  **Digite a nova senha** para o `root` e confirme-a.
    ```
    New password:
    Retype new password:
    passwd: password updated successfully
    ```
    A senha do `root` será resetada.

## 3. Resetar Senha do Root ou de Qualquer Usuário (sem acesso `sudo` / senha do root perdida)
---

Esta é a situação mais complexa, onde você não tem acesso a nenhuma conta de administrador. O procedimento envolve editar as opções de boot do GRUB (Grand Unified Bootloader) para acessar o sistema em modo de usuário único (single-user mode) ou um shell de recuperação, onde você terá privilégios de root.

### Procedimento Geral (Variações podem ocorrer por distribuição)

1.  **Reinicie o computador.**
2.  **Acesse o menu do GRUB:**
    * Assim que o computador ligar, antes do sistema operacional começar a carregar, comece a pressionar a tecla `Shift` (ou `Esc` em alguns sistemas) repetidamente. Isso deve exibir o menu do GRUB. Se o GRUB não aparecer, pode ser que seu sistema inicialize muito rápido, ou ele esteja oculto.
3.  **Edite a Entrada de Boot:**
    * No menu do GRUB, selecione a entrada do seu sistema Linux (geralmente a primeira opção).
    * Pressione a tecla `e` para editar os parâmetros de boot.
4.  **Modifique a Linha do Kernel:**
    * Procure pela linha que começa com `linux` ou `linuxefi`.
    * Nessa linha, encontre o parâmetro `ro` (read-only) e mude-o para `rw` (read-write).
    * No final dessa mesma linha (após `quiet splash` ou similar), adicione `init=/bin/bash` ou `rd.break` ou `single`.
        * `init=/bin/bash`: Inicializa diretamente para um shell bash como root.
        * `rd.break`: Entra no shell do `initramfs` (ambiente de recuperação). Pode ser necessário `chroot /sysroot` depois.
        * `single`: Inicializa em modo de usuário único.

    **Exemplo (para Ubuntu/Debian com `init=/bin/bash`):**
    `... ro quiet splash init=/bin/bash`
    **Exemplo (para Fedora/CentOS/RHEL com `rd.break`):**
    `... ro crashkernel=auto resume=/dev/mapper/centos-swap rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet rd.break`

5.  **Inicialize com as Alterações:**
    * Após editar a linha, pressione `Ctrl + x` ou `F10` (dependendo do GRUB) para inicializar com as alterações.

6.  **Acesse o Shell de Root:**
    * Se tudo correr bem, você será levado a um prompt de comando (`#` ou `root@localhost:/#`), significando que você está logado como `root`.

7.  **Montar o Sistema de Arquivos como Gravável (se necessário):**
    * Se você usou `rd.break` ou se o sistema de arquivos ainda estiver somente leitura:
        ```bash
        mount -o remount,rw /       # Monta a partição raiz como leitura/escrita
        mount -o remount,rw /sysroot # Se estiver no shell initramfs após rd.break
        chroot /sysroot             # Entrar no ambiente chroot para o sistema real
        ```

8.  **Resetar a Senha:**
    * Agora você pode usar o comando `passwd` para resetar a senha do `root` ou de qualquer usuário:
        ```bash
        passwd root             # Para resetar a senha do root
        passwd nome_do_usuario  # Para resetar a senha de outro usuário
        ```
    * Digite a nova senha e confirme.

9.  **Sair do Shell de Recuperação e Reiniciar:**
    * Se você usou `init=/bin/bash`:
        ```bash
        sync                # Sincroniza dados para o disco
        exec /sbin/init     # Tenta reiniciar o processo de inicialização normal
        ```
        Ou simplesmente:
        ```bash
        reboot -f           # Força um reboot
        ```
    * Se você usou `rd.break` e depois `chroot /sysroot`:
        ```bash
        exit                # Saia do ambiente chroot
        exit                # Saia do shell initramfs
        ```
        O sistema deve continuar o processo de inicialização normal.

10. **Testar a Nova Senha:**
    * Após a reinicialização, tente fazer login com a nova senha.

## Considerações de Segurança
---

* O acesso físico ao sistema é um risco de segurança. Qualquer pessoa com acesso físico pode, com conhecimento adequado, resetar senhas ou acessar dados.
* **Criptografia de Disco:** Se o seu disco raiz estiver criptografado (ex: com LUKS), o processo de recuperação de senha será mais complexo e pode exigir a chave de criptografia ou a senha do volume. O procedimento acima geralmente não funciona diretamente em discos criptografados sem etapas adicionais.
* **GRUB com Senha:** Para maior segurança, você pode proteger o menu do GRUB com uma senha para impedir que usuários não autorizados modifiquem as entradas de boot.

Este guia fornece os passos essenciais para recuperar o acesso ao seu sistema Linux em caso de senha esquecida.
