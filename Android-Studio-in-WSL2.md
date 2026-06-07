# 📱 Ambiente de Desenvolvimento Android (WSL2 + Arch Linux)

Este guia descreve a configuração completa para executar o emulador Android rodando no Windows, mantendo o ambiente de desenvolvimento (React Native, Flutter, Kotlin) isolado e performático dentro do **Arch Linux** via WSL2.

---

## 🧩 Visão Geral da Arquitetura

```
Windows Host
├── Android Studio (Emulador / AVD)
├── Android SDK  →  C:\Users\<seu_user>\AppData\Local\Android\Sdk
└── ADB server (porta 5555 via tcpip)

WSL2 — Arch Linux
├── Node.js + npm + Expo CLI
├── JDK 17 (compilação via Gradle)
├── android-tools (adb nativo Linux)
├── ~/.android-sdk/              ← ANDROID_HOME
│   ├── platform-tools/         ← symlink → /usr/bin/adb (Linux!)
│   ├── build-tools/            ← symlink → SDK do Windows
│   ├── platforms/              ← symlink → SDK do Windows
│   ├── emulator/               ← symlink → SDK do Windows
│   ├── licenses/               ← symlink → SDK do Windows
│   └── ndk/27.1.12297006/      ← NDK Linux nativo (baixado separado!)
└── Expo Metro Bundler (porta 8081)
```

> [!TIP]
> **Por que essa estrutura híbrida?**
> O Expo/Gradle precisa do `adb` como binário Linux — o `adb.exe` do Windows não é executável no WSL. Ao mesmo tempo, o resto do SDK (build-tools, platforms) pode vir do Windows via symlink, evitando duplicar gigabytes de dados.
> O NDK é a exceção: o `clang` dentro do NDK do Windows é um `.exe` e não compila código nativo no Linux. Por isso o NDK precisa ser a versão Linux, instalada separadamente.

---

## 🛠️ 1. Pré-requisitos de Hardware e Sistema

Antes de iniciar, certifique-se de que a virtualização está ativa.

> [!CAUTION]
> A virtualização de hardware na BIOS/UEFI é obrigatória.
> Verifique em: **Gerenciador de Tarefas > Desempenho > CPU > Virtualização: Habilitado**.

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
2. Durante o setup, instale o **Android SDK**, **Android Virtual Device (AVD)** e o **Android SDK Command-line Tools**.

> [!TIP]
> Se o motor do emulador (AVD) estiver indisponível, instale manualmente o **Android Emulator Hypervisor Driver (AEHD)** via *SDK Manager > SDK Tools*.

### B. Criação do Emulador (AVD)

- No Device Manager, crie um novo dispositivo (ex: **Pixel 9**).
- Utilize a **API 36 (Android 16)** e a imagem **x86_64** para máxima aceleração de hardware.

### C. Variáveis de Ambiente (PATH & JAVA_HOME)

Adicione estes caminhos ao **Path do Sistema** no Windows (substitua `<seu_user>`):

| Nome do ENV | Caminho                                                      |
|-------------|--------------------------------------------------------------|
| JAVA_HOME   | C:\Program Files\Android\Android Studio\jbr                  |
| PATH        | %JAVA_HOME%\bin                                              |
| PATH        | C:\Users\<seu_user>\AppData\Local\Android\Sdk\emulator       |
| PATH        | C:\Users\<seu_user>\AppData\Local\Android\Sdk\platform-tools |

---

## 🐧 3. Setup no Arch Linux (WSL2)

Agora, configure a ponte de comunicação entre o Linux e o Windows.

### Instalação de Ferramentas

No terminal do Arch:

```bash
sudo pacman -Syu
sudo pacman -S android-tools jdk17-openjdk
```

- `android-tools` → fornece o `adb` nativo Linux (`/usr/bin/adb`)
- `jdk17-openjdk` → necessário para o Gradle compilar o projeto

### Dependências do React Native DevTools (Chromium embutido)

O `@react-native/debugger-shell` é um binário Electron/Chromium e precisa de várias libs do sistema:

```bash
sudo pacman -S nss nspr atk at-spi2-atk mesa cups alsa-lib gtk3
```

> [!TIP]
> Se ainda aparecer erro de lib faltando ao iniciar o DevTools, rode para diagnóstico:
> ```bash
> ldd ~/projects/<seu-projeto>/node_modules/@react-native/debugger-shell/bin/react-native-devtools | grep "not found"
> ```
> Cada linha retornada é uma lib que precisa ser instalada via `sudo pacman -S <lib>`.

### 🏗️ Estrutura do ANDROID_HOME no Arch

```bash
# Cria a pasta base
mkdir -p ~/.android-sdk/platform-tools
mkdir -p ~/.android-sdk/ndk

# Symlink do adb → binário Linux nativo
ln -sf /usr/bin/adb ~/.android-sdk/platform-tools/adb

# Symlinks para o resto do SDK no Windows
ln -sf /mnt/c/Users/<seu_user>/AppData/Local/Android/Sdk/build-tools ~/.android-sdk/build-tools
ln -sf /mnt/c/Users/<seu_user>/AppData/Local/Android/Sdk/platforms ~/.android-sdk/platforms
ln -sf /mnt/c/Users/<seu_user>/AppData/Local/Android/Sdk/emulator ~/.android-sdk/emulator
ln -sf /mnt/c/Users/<seu_user>/AppData/Local/Android/Sdk/licenses ~/.android-sdk/licenses
```

### Instala o NDK Linux nativo

O projeto usa NDK `27.3.13750724`, que corresponde ao release `r27d`:

```bash
cd ~
wget https://dl.google.com/android/repository/android-ndk-r27d-linux.zip
unzip android-ndk-r27d-linux.zip
mv android-ndk-r27d-linux ~/.android-sdk/ndk/27.3.13750724

# Confirma que é binário Linux (deve mostrar "ELF 64-bit")
file ~/android-sdk/ndk/27.3.13750724/toolchains/llvm/prebuilt/linux-x86_64/bin/clang-18
```

### ⚙️ Configuração de Shell (`~/.zshrc`)

Adicione estes aliases e caminhos para automatizar a conexão.

```zsh
# Android SDK (estrutura híbrida WSL + Windows)
export ANDROID_HOME=$HOME/.android-sdk
export PATH=$ANDROID_HOME/platform-tools:$PATH:$ANDROID_HOME/emulator

# Java (necessário para o Gradle)
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$PATH:$JAVA_HOME/bin

# Alias: Conectar ao emulador do Windows (ADB via TCP)
# Limpa conexões fantasmas e liga ao localhost:5555
adb-up() {
  echo -e "\n\e[1;34m Iniciando conexão SDK Android in WSL...\e[0m\n"

  adb disconnect && \
    adb connect localhost:5555 && \
      adb devices

  echo -e "\n\e[1;32m Ponte Android Ativa!\e[0m\n"
}

# Alias: Desconectar (Graceful Shutdown)
adb-down() {
  echo -e "\n\e[1;34m Finalizando conexão SDK Android in WSL...\e[0m\n"

  adb disconnect localhost:5555 && \
    adb kill-server

  echo "\n\e[1;32m Ponte Android Desconectada!\e[0m\n"
}
```

> [!TIP]
> **Verifique após o `source ~/.zshrc`:**
> ```bash
> which adb        # deve ser ~/.android-sdk/platform-tools/adb
> echo $ANDROID_HOME  # deve ser /home/<seu_user>/.android-sdk
> java -version    # deve mostrar openjdk 17
> ```

---

## 🔑 4. Licenças do Android SDK

As licenças são aceitas via `sdkmanager` no Windows. Para rodar o `sdkmanager.bat` sem instalar o Java separadamente, use o JDK embutido no Android Studio:

**No PowerShell (Windows):**

```powershell
cd C:\Users\<seu_user>\AppData\Local\Android\Sdk\cmdline-tools\latest\bin
.\sdkmanager.bat --licenses
# Responda 'y' para todas as licenças
```

> [!TIP]
> **Pré-requisito:** O `cmdline-tools` deve estar instalado. Caso não esteja:
> Android Studio → Settings → SDK Manager → SDK Tools → ✅ Android SDK Command-line Tools (latest) → Apply

Como o `~/.android-sdk/licenses` é um symlink para a pasta de licenças do Windows, o Gradle no WSL enxerga as licenças automaticamente.

---

## 🔄 5. Fluxo de Trabalho Diário

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

> [!IMPORTANT]
> Se o terminal exibir `unauthorized`, aceite a solicitação de depuração USB que aparecerá na tela do celular emulado.

---

## 🧹 6. Finalização (Graceful Shutdown)

Para economizar bateria e CPU ao terminar o trabalho:

1. No **Arch**: Execute `adb-down`.
2. No **Windows**: Feche o emulador e execute `adb kill-server`.

---

## 📦 Resumo das Versões Utilizadas

| Ferramenta              | Versão                 |
|-------------------------|------------------------|
| Android NDK             | r27b (`27.1.12297006`) |
| Android SDK Build-Tools | 36.0.0                 |
| Android SDK Platform    | API 36                 |
| JDK                     | OpenJDK 17             |
| Expo SDK                | 53+ (bare workflow)    |
| React Native            | 0.79+                  |

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>
