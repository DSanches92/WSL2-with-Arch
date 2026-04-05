# Ambiente Desenvolvimento Android (WSL2 + Arch Linux)
Este documento descreve o setup do zero para rodar o emulador no Windows e o desenvolvimento no Arch Linux (WSL2).

---

## 1. Pré-requisitos de Hardware e Windows

### A. Habilitar Virtualização (BIOS/UEFI)
Certifique-se de que a virtualização de hardware está ativa. No Gerenciador de Tarefas > Aba Desempenho > CPU, verifique se aparece: **Virtualização: Habilitado**.

### B. Recursos do Windows
Ative os seguintes itens em "Ativar ou desativar recursos do Windows":
- Plataforma de Máquina Virtual
- Plataforma do Hipervisor do Windows
- Subsistema do Windows para Linux (WSL)

_Nota: Se você já utiliza ambiente WSL, você já possui esses itens habilitados._

---

## 2. Setup no Windows (Host)

### A. Instalação do Android Studio
Baixe e instale o Android Studio. Durante a instalação:
- Instale o **Android SDK** e o **Android Virtual Device**.
- Se o "AVD" estiver _Unavailable_, finalize a instalação e instale o **Android Emulator Hypervisor Driver (AEHD)** via SDK Manager > SDK Tools no Settings do Android Studio.

### B. Configuração do Emulador (AVD)
No Device Manager do Android Studio:
- Crie um novo dispositivo (Recomendado: **Pixel 8**).
- Selecione a **API 34 (Android 14)** e a imagem **x86_64** para melhor performance.

### C. Variáveis de Ambiente (PATH)
Adicione estes dois caminhos ao **Path do Sistema** (substitua `SeuUsuário`):
- `C:\Users\SeuUsuário\AppData\Local\Android\Sdk\platform-tools`
- `C:\Users\SeuUsuário\AppData\Local\Android\Sdk\emulator`

---

## 3. Setup no Arch Linux (WSL2)
No terminal do Arch, instale os pacotes essenciais:
```zsh
sudo pacman -Syu
sudo pacman -S android-tools
```

Adicione estas variáveis e aliases ao seu `~/.zshrc` ou `~/.bashrc`:
```zsh
# Android SDK Paths (Interno WSL)
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Alias para usar o ADB do Windows (Evita conflito de versão)
#alias adbw='/mnt/c/Users/Danilo/AppData/Local/Android/Sdk/platform-tools/adb.exe'

# Limpa conexões fantasmas e conecta ao emulador do Windows
alias adb-up='adb disconnect && adb connect localhost:5555 && adb devices && echo "Ambiente Android Conectado!"'

# Encerra processos ADB de forma limpa
alias adb-down='adb disconnect localhost:5555 && adb kill-server && echo "Ambiente Android Desconectado!"'
```

Aplique com: `source ~/.zshrc`

---

## 4. Fluxo de Trabalho (Diário)
- **Iniciar Emulador (Windows PowerShell):**
```powershell
emulator -list-avds
emulator -avd Nome_Do_Seu_AVD -no-boot-anim
```

- **Habilitar Ponte (Windows PowerShell - Nova Janela):**
```powershell
adb tcpip 5555
```

- **Conectar (Arch Terminal) via alias:**
```zsh
adb-up
```
_Nota: Se aparecer "unauthorized" no terminal, aceite o popup no celular emulado._

- **Desenvolver (Arch Terminal):**
```zsh
npx react-native start
npx react-native run-android
```

---

## 5. Encerramento (Graceful Shutdown)
Para evitar que o ADB trave portas ou consuma CPU desnecessária:

- No Arch: Execute `adb-down`.
- No Windows: Feche o emulador e execute `adb kill-server` no PowerShell.
