# 🚀 WSL2 with Arch Linux: Fullstack Development Environment
Repositório dedicado à minha configuração personalizada de desenvolvimento no **Windows 11 PRO**, utilizando **Arch Linux** via WSL2. Um ambiente otimizado para performance, focado em isolamento de ferramentas e produtividade.

---

## 💻 Hardware Specs
- **CPU:** Ryzen 5 3600 (6C/12T)
- **RAM:** 16GB
- **OS:** Windows 11 PRO (Host) / Arch Linux (Guest)

---

## 🛠️ Guia de Instalação e Configuração
### Ativação Recursos Windows
Para não instalar automáticamente o Ubuntu, assim, te permitindo instalar outra distro. Executar no PowerShell como Administrador.

- Ativar o recusdo de **Subsistema do Windows para Linux** na tela de Recursos do Windows:
  ```shell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```
- Ativar o Recurso de **Plataforma de Máquina Virtual** na tela de Recursos do Windows:
  ```sehll
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```
- Ativar o recurso de **Hyper-V** na tela de Recursos do Windows:
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
  ```bash
  passwd
  ```
- No Terminal Arch:
  ```bash
  echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
  ```
- No Terminal Arch:
  ```bash
  useradd -m -G wheel -s /bin/bash {username}
  ```
- No Terminal Arch:
  ```bash
  passwd {username}
  ```
- No Terminal Arch: `exit`
- Com o Terminal Windows na pasta C:\Arch, executar:
  ```shell
  .\Arch.exe config --default-user {username}
  ```

### Inicializar e Atualizar o Arch

- No Terminal Arch:
  ```bash
  sudo nano /etc/pacman.conf
  ```
- No arquivo de configuração, procurar por **ParallelDownloads** e Alterar para 10 (no meu caso)
- No Terminal Arch:
  ```bash
  sudo pacman-key --init
  ```
- No Terminal Arch:
  ```bash
  sudo pacman-key --populate
  ```
- No Terminal Arch:
  ```bash
  sudo pacman -Sy archlinux-keyring
  ```
- No Terminal Arch:
  ```bash
  sudo pacman -Syyuu
  ```
- No Terminal Arch:
  ```bash
  sudo pacman -S git base-devel
  ```

### Instalando o YAY - Gerenciador de Pacotes AUR

- Projeto YAY: <https://github.com/Jguer/yay>
- No Terminal Arch: `cd /tmp`
- No Terminal Arch:
  ```bash
  git clone https://aur.archlinux.org/yay-bin.git && cd yay-bin && makepkg -si
  ```

### ZSH - History Database and Syntax

- No Terminal Arch:
  ```bash
  yay -S --noconfirm zsh
  ```
- No Terminal Arch:
  ```bash
  chsh -s /usr/bin/zsh
  ```
- Feche o Terminal e abra novamente com o novo Shell (deve aparecer o script para configuração inicial)

#### Database
- No Terminal Arch:
  ```bash
  yay -S --noconfirm sqlite3
  ```
- No Terminal Arch:
  ```bash
  git clone https://github.com/larkery/zsh-histdb ~/.zsh/zsh-histdb
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo:
  ```bash
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

#### Auto Suggestions

- No Terminal Arch:
  ```bash
  git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo para iniciar com o ZSH:
  ```bash
  source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
  ```

#### Syntax Highlighting

- No Terminal Arch:
  ```bash
  git clone https://github.com/zsh-users/zsh-syntax-highlighting ~/.zsh/zsh-syntax-highlighting
  ```
- No Terminal Arch: `nano .zshrc`
- Adicionar no arquivo:
  ```bash
  source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
  ```
 
 ---
 
### Gerenciamento de Stacks (Node & Python)

- **Node.js:** Instalado via **NVM** para gestão de versões.
- **Python:** Instalado via **pyenv** para isolamento total.
- Dependências de compilação para o Arch
  ```bash
  sudo pacman -S --needed base-devel openssl zlib bzip2 readline sqlite3 curl llvm libffi xz tk ncurses
  ```
- Instalação e Configuração Python
  ```bash
  pyenv install 3.12 && pyenv global 3.12
  ```

### Containerization: Podman (Rootless)
Substituto do Docker Desktop, configurado para rodar sem `sudo`.

- Instalação Podman e Fuse OverlayFS
  ```bash
  sudo pacman -S podman fuse-overlayfs
  ```
- Storage
  ```bash
  cat << 'EOF' > ~/.config/containers/storage.conf
  [storage]
  driver = "overlay"
  graphroot = "/home/{username}/.local/share/containers/storage"

  [storage.options.overlay]
  mount_program = "/usr/bin/fuse-overlayfs"
  EOF
  ```

---

## ⚡ WSL2 Optimization `.wslconfig`
Crie este arquivo em `%UserProfile%\.wslconfig` no Windows para tunar o ambiente:
<pre><code>
[wsl2]
memory=8GB
processors=6
swap=4GB
networkingMode=mirrored
dnsTunneling=true
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
</code></pre>

---

## 🎨 Terminal & Shell
- **Shell:** ZSH com **Oh My Zsh!**
- **Plugins:** `git`, `node`, `python`, `pyenv`, `podman`.
- **Configuração Pyenv no `.zshrc`:**
  <pre><code>
  export PYENV_ROOT="$HOME/.pyenv"
  export PATH="$PYENV_ROOT/bin:$PATH"
  eval "$(pyenv init --path)"
  eval "$(pyenv init -)"
  </code></pre>

---

Criado por [Danilo Sanches](https://github.com/DSanches92).
