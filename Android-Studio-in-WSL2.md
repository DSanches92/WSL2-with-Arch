# 📱 Ambiente de Desenvolvimento Android (WSL2 + Arch Linux)

Este guia descreve a configuração completa para executar o emulador Android nativamente no Windows, mantendo o ambiente de desenvolvimento (React Native, Flutter, Kotlin) isolado e performático dentro do **Arch Linux** via WSL2.

---

## 🛠️ 1. Pré-requisitos de Hardware e Sistema

Antes de iniciar, certifique-se de que a virtualização está ativa.

[!CAUTION]
A virtualização de hardware na BIOS/UEFI é obrigatória.
Verifique em: **Gerenciador de Tarefas > Desempenho > CPU > Virtualização: Habilitado**.

### ✅ Recursos do Windows Ativos
Ative as seguintes funcionalidades em "Ativar ou desativar recursos do Windows":
- [x] **Plataforma de Máquina Virtual**
- [x] **Plataforma do Hipervisor do Windows**
- [x] **Subsistema do Windows para Linux (WSL)**

---

## 💻 2. Setup no Windows (Host)

O segredo para a melhor performance é rodar o **Emulador no Windows** e a **Stack no Linux**.

### A. Instalação e SDK
1. Baixe o [Android Studio](https://developer.android.com/studio).
2. Durante o setup, instale o **Android SDK** e o **Android Virtual Device (AVD)**.

[!TIP]
Se o motor do emulador (AVD) estiver indisponível, instale manualmente o **Android Emulator Hypervisor Driver (AEHD)** via *SDK Manager > SDK Tools*.

### B. Criação do Emulador (AVD)
- No Device Manager, crie um novo dispositivo (ex: **Pixel 8**).
- Utilize a **API 34 (Android 14)** e a imagem **x86_64** para máxima aceleração de hardware.

### C. Variáveis de Ambiente (PATH)
Adicione estes caminhos ao **Path do Sistema** no Windows (substitua `<SeuUsuário>`):
```text
C:\Users\<SeuUsuário>\AppData\Local\Android\Sdk\platform-tools
C:\Users\<SeuUsuário>\AppData\Local\Android\Sdk\emulator
```

---

## 🦅 3. Setup no Arch Linux (WSL2)

Agora, configure a ponte de comunicação entre o Linux e o Windows.

### Instalação de Ferramentas
No terminal do Arch:
```bash
sudo pacman -Syu
sudo pacman -S android-tools
```

### Configuração de Shell (`~/.zshrc`)
Adicione estes aliases e caminhos para automatizar a conexão.

```zsh
# Android SDK Paths (Interno WSL)
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools

# 🔌 Alias: Conectar ao emulador do Windows (ADB via TCP)
# Limpa conexões fantasmas e liga ao localhost:5555
alias adb-up='adb disconnect && adb connect localhost:5555 && adb devices && echo "🚀 Ponte Android Ativa!"'

# 🛑 Alias: Desconectar (Graceful Shutdown)
alias adb-down='adb disconnect localhost:5555 && adb kill-server && echo "😴 Ponte Android Desconectada!"'
```

---

## 🔄 4. Fluxo de Trabalho Diário

Siga esta sequência para iniciar seu desenvolvimento sem atritos:

1.  **💻 Iniciar Emulador (Windows PowerShell):**
    ```powershell
    emulator -list-avds              # Veja o nome do seu device
    emulator -avd Nome_Do_Seu_AVD    # Inicia o emulador
    ```

2.  **🌉 Habilitar Ponte de Rede (Windows PowerShell):**
    ```powershell
    adb tcpip 5555                   # Abre a porta para o WSL
    ```

3.  **🦅 Conectar no Arch (Terminal Linux):**
    ```bash
    adb-up                           # Utiliza o alias criado acima
    ```

[!IMPORTANT]
Se o terminal exibir `unauthorized`, aceite a solicitação de depuração USB que aparecerá na tela do celular emulado.

---

## 🧹 5. Finalização (Graceful Shutdown)

Para economizar bateria e CPU ao terminar o trabalho:
1. No **Arch**: Execute `adb-down`.
2. No **Windows**: Feche o emulador e execute `adb kill-server`.

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>