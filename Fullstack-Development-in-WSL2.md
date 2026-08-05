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

  > [!TIP]
  > Certifique-se de que o arquivo principal `/etc/sudoers` inclua a diretiva `#includedir /etc/sudoers.d`. Geralmente, isso já é o padrão no Arch Linux.
  > Para evitar erros de sintaxe ao configurar permissões, considere usar `EDITOR=nano visudo -f /etc/sudoers.d/wheel`.

  ```bash
  passwd
  echo "%wheel ALL=(ALL) ALL" | tee /etc/sudoers.d/wheel
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
  *Descomente a linha `ParallelDownloads` (remova o símbolo `#` do início dela) e altere de `5` para `10` (ou de acordo com sua conexão).*

- **Sincronizar Chaves e Atualizar:**
  ```bash
  sudo pacman-key --init
  sudo pacman-key --populate
  sudo pacman -Sy archlinux-keyring
  sudo pacman -Syyuu
  sudo pacman -S --needed git base-devel openssl curl wget
  ```

- **Configuração Global do Git:**
  Configure sua identidade global no Git para assinar seus commits corretamente:
  ```bash
  git config --global user.email "silva.danilosanches@gmail.com"
  git config --global user.name "Danilo Sanches da Silva"
  git config --global core.editor "nano"
  ```

---

### 📦 Gerenciamento de Pacotes AUR (PARU)

O [PARU](https://github.com/Morganamilo/paru) é um AUR helper moderno e eficiente, escrito em Rust, essencial para acessar o imenso repositório comunitário do Arch de forma simplificada.

```bash
cd /tmp
git clone https://aur.archlinux.org/paru-bin.git
cd paru-bin
makepkg -si
```

---

### 🐚 Terminal & Shell (ZSH + Oh My Zsh)

Um shell produtivo economiza horas de digitação através de sugestões inteligentes, atalhos rápidos e destaque de sintaxe.

**1. Instalar o ZSH e Dependências:**
```bash
sudo pacman -S zsh sqlite3
chsh -s /usr/bin/zsh
```

**2. Instalar o Oh My Zsh!:**
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

**3. Instalar Plugins de Produtividade (na pasta customizada do OMZ):**
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

**4. Configuração do `.zshrc`:**
Abra o arquivo de configuração com o nano:
```bash
nano ~/.zshrc
```

Garanta que a linha `plugins=(...)` contenha os plugins desejados:
```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

E configure o histórico padrão no final do arquivo:
```bash
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
 
**Configuração `.zshrc` (adicione no final do arquivo):**
```bash
source ~/.zsh/zsh-histdb/sqlite-history.zsh

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

Utilize o `nvm` (Node Version Manager) para gerenciar e alternar facilmente entre múltiplas versões do Node.js.

> [!IMPORTANT]
> **Sempre utilize o guia oficial para obter o script de instalação mais atualizado!**
> Para garantir que você está instalando a versão mais recente e segura do NVM, consulte o [Guia Oficial de Download do Node.js (via Gerenciador de Pacotes)](https://nodejs.org/en/download/package-manager) ou acesse diretamente o repositório oficial do [NVM no GitHub](https://github.com/nvm-sh/nvm#installing-and-updating).

**1. Instalar o NVM:**
Acesse o link acima e execute o comando de instalação atualizado. Como referência, o comando segue este formato:
```bash
# IMPORTANTE: Verifique no site oficial do Node.js/NVM a versão mais recente antes de rodar!
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

**2. Configuração `.zshrc`:**
Geralmente o script de instalação adiciona as linhas abaixo automaticamente. Caso não tenham sido adicionadas, abra o `~/.zshrc` e adicione ao final:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # Carrega o NVM
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # Carrega completions do NVM
```

**3. Instalar a Versão LTS do Node.js:**
Após carregar o shell atualizado (`source ~/.zshrc`), instale a versão LTS recomendada:
```bash
nvm install --lts
nvm use --lts
node -v # Validação da versão ativa
npm -v  # Validação do gerenciador de pacotes
```

#### Python

Utilize o `pyenv` para isolamento total de versões e pacotes.

**Dependências de Build:**
```bash
paru -S pyenv zlib bzip2 readline llvm libffi xz tk ncurses
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

Utilize `paru` ou `podman` para instalar o Golang diretamente:

```bash
paru -S go
```

**Configuração `.zshrc`:**
```bash
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

Atualize as configurações com `source ~/.zshrc`.

**Validação:**
```bash
go version

mkdir -p ~/go/hello
cd ~/go/hello

cat > main.go << 'EOF'
package main

import "fmt"

func main() {
  fmt.Println("GO OK!")
}
EOF

go run main.go
```

---

### 🐳 Containerization: Podman (Rootless)

O Podman é o substituto moderno do Docker Desktop, funcionando de forma nativa e segura no Arch Linux sem necessidade de `sudo`.

```bash
paru -S podman fuse-overlayfs
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
unqualified-search-registries = ["docker.io", "ghcr.io", "quay.io"]
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

O arquivo de configuração `/etc/wsl.conf` gerencia as configurações internas da sua distribuição do WSL. Crie ou edite este arquivo utilizando:
```bash
sudo nano /etc/wsl.conf
```

Confirme se o conteúdo do arquivo está exatamente como abaixo:

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

No **Windows Host**, crie ou edite o arquivo `.wslconfig` no diretório do seu usuário (geralmente `C:\Users\{SeuUsuario}\.wslconfig` ou `%UserProfile%\.wslconfig`) para gerenciar as configurações globais de hardware de todas as distribuições WSL2:

Abra o PowerShell do Windows e digite o seguinte comando para abrir/criar o arquivo rapidamente no Bloco de Notas:
```powershell
notepad $env:USERPROFILE\.wslconfig
```

Insira a configuração de alta performance abaixo:

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

Agora que você tem o **Oh My Zsh!** configurado, atualize a lista de plugins no seu `~/.zshrc` para carregar completações automáticas de atalhos e binários para todas as suas stacks instaladas:

```bash
plugins=(
  git
  node
  python
  pyenv
  podman
  golang
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

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

> [!NOTE]
> Para utilizar esta função, você precisa instalar o Snyk CLI globalmente através do npm:
> ```bash
> npm install -g snyk
> ```

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
