# 🚀 WSL2 with Arch Linux: Fullstack Development Environment

Guia definitivo para configurar um ambiente de desenvolvimento de alta performance no **Windows 11 PRO**, utilizando **Arch Linux** via WSL2. Otimizado para isolamento de ferramentas, produtividade e baixa latência.

---

## 💻 Hardware Specs

| Componente  | Especificação         |
|-------------|-----------------------|
| **CPU**     | Ryzen 5 3600 (6C/12T) |
| **RAM**     | 16GB DDR4             |
| **Host OS** | Windows 11 PRO        |
| **Guest OS**| Arch Linux (WSL2)     |

---

## 🛠️ Guia de Instalação e Configuração

### 1. Ativação de Recursos do Windows

Antes de instalar o Arch, precisamos garantir que o subsistema do Windows está pronto. Execute os comandos abaixo no **PowerShell como Administrador**.

> [!IMPORTANT]
> A ordem de execução é fundamental para evitar a instalação automática do Ubuntu padrão da Microsoft Store.

- **Ativar Subsistema do Windows para Linux:**
  ```powershell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```
- **Ativar Plataforma de Máquina Virtual:**
  ```powershell
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```
- **Ativar Hyper-V:**
  ```powershell
  dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V /all /norestart
  ```

> [!CAUTION]
> Após executar os comandos acima, você **DEVE** reiniciar seu computador para aplicar as alterações.
> ```powershell
> shutdown -r -t 0
> ```

- **Pós-Reinicialização:** Atualize o kernel do WSL e defina a versão padrão:
  ```powershell
  wsl --update
  wsl --set-default-version 2
  ```

---

### 2. Instalação do Arch Linux (ArchWSL)

Utilizaremos a implementação customizada do [ArchWSL](https://github.com/yuk7/ArchWSL) para maior controle.

Obrigado / Thank You [yuk7](https://github.com/yuk7) 💕

- **Download:** Baixe o `.zip` da release estável em [ArchWSL Releases](https://github.com/yuk7/ArchWSL).
- **Preparação:** Descompacte os arquivos na pasta `C:\Arch`.
- **Bootstrapping:** Abra o Terminal do Windows na pasta `C:\Arch` e execute:
  ```powershell
  .\Arch.exe
  ```
- **Configuração de Usuário:**
  No terminal do Arch que se abriu, vamos configurar o roteamento de permissões:
  ```bash
  passwd
  echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
  useradd -m -G wheel -s /bin/bash {seu_nome}
  passwd {seu_nome}
  exit
  ```
- **Usuário Padrão:** No PowerShell do Windows (na pasta `C:\Arch`):
  ```powershell
  .\Arch.exe config --default-user {seu_nome}
  ```

---

### 3. Inicialização e Otimização do Sistema (pacman)

Agora vamos preparar o Arch Linux para ser o cavalo de batalha do seu desenvolvimento.

- **Otimizar Downloads:**
  ```bash
  sudo nano /etc/pacman.conf
  ```
  *Altere `ParallelDownloads = 5` para `10` (ou de acordo com sua conexão).*

- **Sincronizar Chaves e Atualizar:**
  ```bash
  sudo pacman-key --init
  sudo pacman-key --populate
  sudo pacman -Sy archlinux-keyring
  sudo pacman -Syyuu
  sudo pacman -S git base-devel openssl curl wget
  ```

---

### 📦 Gerenciamento de Pacotes AUR (YAY)

O [YAY](https://github.com/Jguer/yay) é essencial para acessar o imenso repositório comunitário do Arch, mas é opcional para o que faremos aqui.

```bash
cd /tmp
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

---

### 🐚 Terminal & Shell (ZSH)

Um shell produtivo economiza horas de digitação através de sugestões inteligentes.

**Instalação:**
```bash
sudo pacman -S zsh sqlite3
chsh -s /usr/bin/zsh
```

> [!TIP]
> Ao abrir o ZSH pela primeira vez, escolha a opção padrão ou configure via assistente.

**Plugins de produtividade:**
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ~/.zsh/zsh-syntax-highlighting
```

**Configuração `.zshrc`:**
```bash
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

# Histórico no ZSH
HISTFILE=~/.zsh_history
HISTSIZE=10000
SAVEHIST=10000
```

#### 🧠 Sugestões Inteligentes com Database (zsh-histdb) — Opcional
 
Diferente do histórico padrão, o `histdb` utiliza SQLite para sugerir comandos baseados no seu **contexto atual** (diretório, frequência, histórico por projeto).
 
**Instalação adicional:**
```bash
git clone https://github.com/larkery/zsh-histdb ~/.zsh/zsh-histdb
```
 
**Configuração `.zshrc` (substitui a padrão acima):**
```bash
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

#### Node.js

Utilize o `nvm` para alternar entre versões de runtime facilmente.

#### Python

Utilize o `pyenv` para isolamento total de versões e pacotes.

**Dependências de Build:**
```bash
sudo pacman -S zlib bzip2 readline llvm libffi xz tk ncurses
```

**Instalação:**
```bash
pyenv install 3.12 && pyenv global 3.12
```

**Configuração `.zshrc`:**
```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

#### GO

Utilize `pacman` para instalar o Golang diretamente:

```bash
sudo pacman -S go
```

**Configuração `.zshrc`:**
```bash
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin:$GOBIN
```

Atualize as configurações com `source ~/.zshrc`.

**Validação:**
```bash
go version
go run -e 'package main; import "fmt"; func main() { fmt.Println("Go ok!") }'
```

---

### 🐳 Containerization: Podman (Rootless)

O Podman é o substituto moderno do Docker Desktop, funcionando de forma nativa e segura no Arch Linux sem necessidade de `sudo`.

```bash
sudo pacman -S podman fuse-overlayfs
# Para uso com Podman no WSL 2, selecione: 1) crun (padrão e recomendado)
```

**1. Verifica se seu usuário está mapeado:**
```bash
grep {seu_nome} /etc/subuid /etc/subgid

# Se não estiver:
sudo usermod --add-subuids 100000-165535 {seu_nome}
sudo usermod --add-subgids 100000-165535 {seu_nome}
```

**2. Configuração de Registries (Docker Hub):**
```bash
mkdir -p ~/.config/containers
cat > ~/.config/containers/registries.conf << 'EOF'
[registries.search]
registries = ["docker.io", "ghcr.io", "quay.io"]
EOF
```

**3. Configuração de Storage (Rootless):**
```bash
cat > ~/.config/containers/storage.conf << 'EOF'
[storage]
driver = "overlay"

[storage.options.overlay]
mount_program = "/usr/bin/fuse-overlayfs"
EOF
```

**4. Ajustando `Shared Mount "/"`**

Verifique seu `sudo nano /etc/wsl.conf` e confirme o conteúdo abaixo:

```ini
[boot]
systemd=true

[automount]
enabled=true
options=metadata,uid=1000,gid=1000,umask=022
mountFsTab=true

[network]
hostname=arch-wsl
generateHosts=true
generateResolvConf=true
```

Reinicie seu WSL no PowerShell Windows com `wsl --shutdown` e siga abaixo:

```bash
# Verifica o tipo atual (se aparecer como "private", siga os passos abaixo)
findmnt -o TARGET,PROPAGATION /

# 1. Cria o serviço systemd
sudo tee /etc/systemd/system/wsl-mount-shared.service << 'EOF'
[Unit]
Description=Make root mount shared for Podman rootless
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/usr/bin/mount --make-rshared /
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

# 2. Ativa o serviço
sudo systemctl enable --now wsl-mount-shared.service
```

> [!CAUTION]
> Após as configurações, reinicie seu WSL novamente com `wsl --shutdown`.

**5. Validação completa:**
```bash
findmnt -o TARGET,PROPAGATION /   # Deve exibir "shared"
podman info | grep -E "rootless|cgroupVersion"
podman run --rm hello-world
podman run --rm alpine echo "rootless ok"
```

---

## ⚡ Otimização do WSL2 (`.wslconfig`)

No Windows, crie o arquivo `%UserProfile%\.wslconfig` para gerenciar os recursos de hardware do WSL:

```ini
[wsl2]
memory=8GB           # Reserve metade da sua RAM
processors=6         # Metade das threads lógicas (Ryzen 5 3600 = 12 threads)
swap=4GB
networkingMode=mirrored
dnsTunneling=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

---

## 🎨 Personalização Final

Para uma experiência premium, recomendo o uso do **Oh My Zsh!** com os seguintes plugins no seu `.zshrc`:
- `git`, `node`, `python`, `pyenv`, `podman`, `golang`.

**Antigravity com WSL 2:**
```bash
antigravity() {
  echo -e "\n\e[1;34m Iniciando Antigravity IDE...\e[0m\n"

  local WIN_USER=$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')
  local AG_EXE="/mnt/c/Users/$WIN_USER/AppData/Local/Programs/Antigravity IDE/bin/antigravity-ide"
  local TARGET="${1:-.}"
  local ABS_PATH=$(readlink -f "$TARGET")

  "$AG_EXE" --remote wsl+Arch "$ABS_PATH"
}
```

**Limpando o Podman:**
```bash
podman-nuke() {
  echo -e "\n\e[1;34m Iniciando limpeza total do Podman (Containers, Imagens, Volumes)...\e[0m\n"

  podman system prune --all --volumes --force && \
    podman rm -f -a && podman rmi -f -a

  echo -e "\n\e[1;32m Podman limpo com sucesso!\e[0m\n"
}
```

**Scan com o Snyk CLI:**
```bash
snyk-scan() {
  echo -e "\n\e[1;34m Iniciando Varredura Completa do Snyk...\e[0m\n"

  echo -e "\e[1;33m[1/3] Analisando Código Escrito (Security & Quality)...\e[0m"
  snyk code test

  echo -e "\n\e[1;33m[2/3] Analisando Arquivos de Configuração (IaC/Dockerfiles)...\e[0m"
  snyk iac test --exclude=node_modules,.vscode,.next,dist,out

  echo -e "\n\e[1;33m[3/3] Analisando Dependências Open-Source (SCA)...\e[0m"
  snyk test --all-projects

  echo -e "\n\e[1;32m Varredura Finalizada!\e[0m\n"
}
```

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>
