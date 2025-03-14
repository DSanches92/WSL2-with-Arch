# Instalar WSL2 e Configurar o Arch Linux

### Ativação Manual
Para não instalar automáticamente o Ubuntu, assim, te permitindo instalar outra distro.
Executar no PowerShell como Administrador.

- Ativar o recusdo de **Subsistema do Windows para Linux** na tela de Recursos do Windows
  ```shell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```
- Ativar o Recurso de **Plataforma de Máquina Virtual** na tela de Recursos do Windows
  ```sehll
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```
- Ativar o recurso de **Hyper-V** na tela de Recursos do Windows
  ```shell
  dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V /all /norestart
  ```
- Pós a ativação dos recursos, reinicie a máquina:
  ```shell
  shutdown -r -t 0
  ```
- Faça o update do WSL:
  ```shell
  wsl --update
  ```
- Configure como default a versão 2:
  ```shell
  wsl --set-default-version 2
  ```

### Instalação do Arch Linux

- Download: <https://github.com/yuk7/ArchWSL>
- Descampactar e colocar os arquivos no C:\Arch
- Com o Terminal Windows na pasta C:\Arch, executar: `.\Arch.exe`
- Fechar o Terminal e Abrir novamente
- Iniciar o Terminal com o Arch Linux
- No Terminal Arch:
  ```shell
  passwd
  ```
- No Terminal Arch:
  ```shell
  echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
  ```
- No Terminal Arch:
  ```shell
  useradd -m -G wheel -s /bin/bash {username}
  ```
- No Terminal Arch:
  ```shell
  passwd {username}
  ```
- No Terminal Arch: `exit`
- Com o Terminal Windows na pasta C:\Arch, executar:
  ```shell
  .\Arch.exe config --default-user {username}
  ```

### Inicializar e Atualizar o Arch

- No Terminal Arch:
  ```shell
  sudo nano /etc/pacman.conf
  ```
- No arquivo de configuração, procurar por **ParallelDownloads** e Alterar para 10 (no meu caso)
- No Terminal Arch:
  ```shell
  sudo pacman-key --init
  ```
- No Terminal Arch:
  ```shell
  sudo pacman-key --populate
  ```
- No Terminal Arch:
  ```shell
  sudo pacman -Sy archlinux-keyring
  ```
- No Terminal Arch:
  ```shell
  sudo pacman -Su
  ```
- No Terminal Arch:
  ```shell
  sudo pacman -S git base-devel
  ```

### Instalando o YAY - Gerenciador de Pacotes AUR

- Projeto YAY: <https://github.com/Jguer/yay>
- No Terminal Arch: `cd /tmp`
- No Terminal Arch:
  ```shell
  git clone https://aur.archlinux.org/yay-bin.git
  ```
- No Terminal Arch: `cd yay-bin`
- No Terminal Arch: `makepkg -si`

### Instalando o Shell ZSH e configurando o Powerlevel10k

- No Terminal Arch: `yay -S zsh`
- Projeto Powerlevel10k: <https://github.com/romkatv/powerlevel10k>
- Clique no link de **Installation**, depois no link **Arch Linux**
- No Terminal Arch:
  ```shell
  yay -S --noconfirm zsh-theme-powerlevel10k-git
  ```
- No Terminal Arch:
  ```shell
  echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
  ```
- No Terminal Arch:
  ```shell
  chsh -s /usr/bin/zsh
  ```
- No Terminal Arch: `exit`
- Projeto Powerlevel10k: <https://github.com/romkatv/powerlevel10k>
- Clique no link de **Fonts**
- Baixe e Instale as fonts **MesloLGS** no Windows
- Abra o Terminal
- Altere a Font nas configurações de Aparência do Terminal Arch
- Abra o Terminal no Arch e Realizar a configuração do Powerlevel10k
- Caso, ao finalizar essa configuração, queira refazer. No terminal Arch: `p10k configure`

### ZSH History Database

- No Terminal Arch:
  ```
  yay -S --noconfirm sqlite3
  ```
- No Terminal Arch:
  ```shell
  git clone https://github.com/larkery/zsh-histdb ~/.zsh/zsh-histdb
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo:
  ```shell
  source ~/.zsh/zsh-histdb/sqlite-history.zsh
  ```
- Adicionar no arquivo:
  ```zsh
  _zsh_autosuggest_strategy_histdb_top() {
    local query="
        select commands.argv from history
        left join commands on history.command_id = commands.rowid
        left join places on history.place_id = places.rowid
        where commands.argv LIKE '$(sql_escape $1)%'
        group by commands.argv, places.dir
        order by places.dir != '$(sql_escape $PWD)', count(*) desc
        limit 1 "

    suggestion=$(_histdb_query "$query")
  }

  ZSH_AUTOSUGGEST_STRATEGY=histdb_top
  ```

### ZSH Auto Suggestions

- No Terminal Arch:
  ```shell
  git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo para iniciar com o ZSH:
  ```shell
  source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
  ```

### ZSH Syntax High Lighting

- No Terminal Arch:
  ```shell
  git clone https://github.com/zsh-users/zsh-syntax-highlighting ~/.zsh/zsh-syntax-highlighting
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo:
  ```shell
  source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
  ```
