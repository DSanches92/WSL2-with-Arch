# 🚀 WSL2 with Arch Linux: Fullstack Development Environment

Guia definitivo para configurar um ambiente de desenvolvimento de alta performance no **Windows 11 PRO**, utilizando **Arch Linux** via WSL2. Otimizado para isolamento de ferramentas, produtividade e baixa latência.

---

## 💻 Hardware Specs

| Componente | Especificação |
| :--------- | :------------ |
| **CPU**    | Ryzen 5 3600 (6C/12T) |
| **RAM**    | 16GB DDR4 |
| **Host OS**| Windows 11 PRO |
| **Guest OS**| Arch Linux (WSL2) |

---

## 🛠️ Guia de Instalação e Configuração

### 1️⃣ Ativação de Recursos do Windows
Antes de instalar o Arch, precisamos garantir que o subsistema do Windows está pronto. Execute os comandos abaixo no **PowerShell como Administrador**.

> [!IMPORTANT]
> A ordem de execução é fundamental para evitar a instalação automática do Ubuntu padrão da Microsoft Store.

*   **Ativar Subsistema do Windows para Linux:**
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    ```
*   **Ativar Plataforma de Máquina Virtual:**
    ```powershell
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```
*   **Ativar Hyper-V:**
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V /all /norestart
    ```

> [!CAUTION]
> Após executar os comandos acima, você **DEVE** reiniciar seu computador para aplicar as alterações.
> ```powershell
> shutdown -r -t 0
> ```

*   **Pós-Reinicialização:** Atualize o kernel do WSL e defina a versão padrão:
    ```powershell
    wsl --update
    wsl --set-default-version 2
    ```

---

### 2️⃣ Instalação do Arch Linux (ArchWSL)
Utilizaremos a implementação customizada do [ArchWSL](https://github.com/yuk7/ArchWSL) para maior controle.

1.  **Download:** Baixe o `.zip` da release estável em [ArchWSL Releases](https://github.com/yuk7/ArchWSL).
2.  **Preparação:** Descompacte os arquivos na pasta `C:\Arch`.
3.  **Bootstrapping:** Abra o Terminal do Windows na pasta `C:\Arch` e execute:
    ```powershell
    .\Arch.exe
    ```
4.  **Configuração de Usuário:**
    No terminal do Arch que se abriu, vamos configurar o roteamento de permissões:
    ```bash
    passwd                         # Define a senha do root
    echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
    useradd -m -G wheel -s /bin/bash {seu_nome}
    passwd {seu_nome}               # Define a senha do seu usuário
    exit                           # Sai do terminal do Arch
    ```
5.  **Usuário Padrão:** No PowerShell do Windows (na pasta `C:\Arch`):
    ```powershell
    .\Arch.exe config --default-user {seu_nome}
    ```

---

### 3️⃣ Inicialização e Otimização do Sistema (Pacman)
Agora vamos preparar o Arch Linux para ser o cavalo de batalha do seu desenvolvimento.

*   **Otimizar Downloads:**
    ```bash
    sudo nano /etc/pacman.conf
    ```
    *Altere `ParallelDownloads = 5` para `10` (ou de acordo com sua conexão).*

*   **Sincronizar Chaves e Atualizar:**
    ```bash
    sudo pacman-key --init
    sudo pacman-key --populate
    sudo pacman -Sy archlinux-keyring
    sudo pacman -Syyuu              # Atualização completa do sistema
    sudo pacman -S git base-devel   # Ferramentas essenciais de build
    ```

---

### 📦 Gerenciamento de Pacotes AUR (YAY)
O [YAY](https://github.com/Jguer/yay) é essencial para acessar o imenso repositório comunitário do Arch.

```bash
cd /tmp
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

---

### 🐚 Terminal & Shell (ZSH + HistDB)
Um shell produtivo economiza horas de digitação através de sugestões inteligentes.

**Instalação básica:**
```bash
yay -S --noconfirm zsh sqlite3
chsh -s /usr/bin/zsh
```

> [!TIP]
> Ao abrir o ZSH pela primeira vez, escolha a opção padrão ou configure via assistente.

#### 🧠 Sugestões Inteligentes com Database (zsh-histdb)
Diferente do histórico padrão, o `histdb` utiliza SQLite para sugerir comandos baseados no seu contexto atual.

1. **Instalação:**
   ```bash
   git clone https://github.com/larkery/zsh-histdb ~/.zsh/zsh-histdb
   git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
   git clone https://github.com/zsh-users/zsh-syntax-highlighting ~/.zsh/zsh-syntax-highlighting
   ```

2. **Configuração `.zshrc`:**
   ```zsh
   # Plugins Essenciais
   source ~/.zsh/zsh-histdb/sqlite-history.zsh
   source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
   source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

   # Estratégia HistDB para Auto-Suggestions
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

---

### ⚡ Desenvolvimento & Stacks

*   **Node.js:** Utilize o `nvm` para alternar entre versões de runtime facilmente.
*   **Python:** Utilize o `pyenv` para isolamento total de versões e pacotes.
    *   **Dependências de Build:**
        ```bash
        sudo pacman -S --needed base-devel openssl zlib bzip2 readline sqlite3 curl llvm libffi xz tk ncurses
        ```
    *   **Instalação:**
        ```bash
        pyenv install 3.12 && pyenv global 3.12
        ```

---

### 🐳 Containerization: Podman (Rootless)
O Podman é o substituto moderno do Docker Desktop, funcionando de forma nativa e segura no Arch Linux sem necessidade de `sudo`.

```bash
sudo pacman -S podman fuse-overlayfs
```

**Configuração de Storage (Rootless):**
```bash
mkdir -p ~/.config/containers
cat << 'EOF' > ~/.config/containers/storage.conf
[storage]
driver = "overlay"
graphroot = "/home/{seunome}/.local/share/containers/storage"

[storage.options.overlay]
mount_program = "/usr/bin/fuse-overlayfs"
EOF
```

---

## ⚡ Otimização do WSL2 (`.wslconfig`)
No Windows, crie o arquivo `%UserProfile%\.wslconfig` para gerenciar os recursos de hardware do WSL:

```ini
[wsl2]
memory=8GB           # Reserve metade da sua RAM
processors=6         # Metade dos núcleos (ou todos, se preferir)
swap=4GB
networkingMode=mirrored
dnsTunneling=true
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

---

## 🎨 Personalização Final
Para uma experiência premium, recomendo o uso do **Oh My Zsh!** com os seguintes plugins no seu `.zshrc`:
- `git`, `node`, `python`, `pyenv`, `podman`.

**Configuração Pyenv indispensável:**
```zsh
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>
