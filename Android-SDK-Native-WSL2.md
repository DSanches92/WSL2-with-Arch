# 📱 Ambiente de Desenvolvimento Android 100% Nativo (WSL2 + Arch Linux)

Este guia descreve a configuração definitiva para criar um ambiente de desenvolvimento Android de altíssima performance **100% rodando no Arch Linux (WSL2)**. Otimizado para **React Native**, **Flutter** ou desenvolvimento Android nativo utilizando um **dispositivo físico via cabo USB** (passthrough com usbipd-win).

---

## 🧩 Visão Geral da Arquitetura

Tudo roda nativamente dentro do sistema de arquivos Linux (Ext4) de alta velocidade, eliminando a lentidão gerada pelo cruzamento de sistema de arquivos do WSL2 (`/mnt/c`).

```
Windows Host
└── usbipd-win (Apenas roteia a porta USB física para o WSL2)
      │
      ▼ (Conexão via Cabo USB)
WSL2 — Arch Linux (TUDO LOCAL, SEGURO E RÁPIDO)
├── Node.js + Metro Bundler (porta 8081)
├── JDK 17 (compilação local ultra rápida via Gradle)
└── Android SDK Nativo (~/Android/Sdk)
    ├── cmdline-tools (sdkmanager nativo)
    ├── platform-tools (adb local que se comunica direto com o celular)
    ├── build-tools
    ├── platforms
    └── ndk (Linux nativo compilando rápido)
```

> [!IMPORTANT]
> **Por que 100% no WSL2?**
> Ao manter o SDK e o NDK dentro do WSL2 (partição Ext4) e não no Windows (`/mnt/c`), o Gradle lê e compila as dependências localmente. O tempo de build (compilação) do React Native diminui em até **10 vezes** devido ao ganho drástico de leitura/escrita de disco.

---

## 💻 1. Setup no Windows Host (Redirecionamento de USB)

Como faremos tudo local no Linux, o Windows Host servirá apenas para "enviar" a conexão do cabo USB do seu celular para dentro do WSL2.

### A. Instalar o usbipd-win

> [!IMPORTANT]
> **Sempre use a versão mais recente!**
> Não fixe uma versão específica do `usbipd-win` neste guia — acesse sempre a página de [Releases no GitHub](https://github.com/dorssel/usbipd-win/releases) e baixe o `.msi` mais recente disponível, garantindo compatibilidade com as versões atuais do WSL2.

1. No Windows, baixe e instale a última release do [usbipd-win GitHub Releases](https://github.com/dorssel/usbipd-win/releases) (arquivo `.msi`). Alternativamente, se preferir, instale via **winget** direto no PowerShell:
   ```powershell
   winget install usbipd
   ```
2. Abra o **PowerShell como Administrador** para validar a instalação:
   ```powershell
   usbipd --version
   ```

---

## 🐧 2. Setup no Arch Linux (WSL2)

Agora vamos preparar o Arch Linux com o SDK Android e as ferramentas de build nativas.

### A. Instalação do JDK (Java) e Ferramentas de Sistema

No terminal do Arch Linux:

```bash
sudo pacman -Syu
sudo pacman -S --needed jdk17-openjdk usbutils hwdata
```
- `jdk17-openjdk` → Java 17 LTS. Recomendado para máxima compatibilidade com a maioria dos projetos React Native e Expo.
- `usbutils` e `hwdata` → Fornece o utilitário `lsusb` para podermos testar e listar dispositivos USB dentro do Linux.

> [!TIP]
> **JDK 17 vs JDK 21 — qual escolher?**
> A relação entre JDK, Gradle e o Android Gradle Plugin (AGP) é uma das maiores causas de dor de cabeça em builds Android.
> - **JDK 17** é atualmente a versão mais estável e universalmente recomendada para a maioria dos projetos React Native (0.73 a 0.75) e Expo (SDK 51). O AGP 8.x exige **no mínimo** o JDK 17 para rodar.
> - **JDK 21**: o próprio AGP não impede o uso do JDK 21, mas quem geralmente barra é o **Gradle** — versões do Gradle anteriores à **8.5** não sabem rodar em JDK 21 e falham com o erro `Unsupported class file major version 65`. O suporte oficial a JDK 21 só chegou no Gradle 8.5+ (tipicamente usado a partir do AGP 8.5+, presente no React Native 0.76+ e Expo SDK 52/53).
>
> Por isso, este guia mantém o `jdk17-openjdk` como recomendação padrão, para garantir que projetos legados e atuais compilem sem quebrar. Se o seu projeto usa React Native 0.76+, Expo SDK 52+ ou Gradle 8.5+, você pode instalar o `jdk21-openjdk` no lugar — basta ajustar o `JAVA_HOME` de acordo na seção de variáveis de ambiente abaixo.

### B. Dependências para Depuração e DevTools (Chromium embutido)

O `@react-native/debugger-shell` e outras ferramentas necessitam de bibliotecas gráficas e fontes instaladas no Linux para iniciar corretamente:

```bash
sudo pacman -S --needed nss nspr atk at-spi2-atk mesa cups alsa-lib gtk3
```

---

## 🏗️ 3. Instalação do Android SDK e NDK (100% Linux)

Faremos a instalação sem depender do instalador visual do Android Studio.

### A. Estrutura de Pastas
Crie a estrutura de diretórios padrão usada pelo Android:
```bash
mkdir -p ~/Android/Sdk/cmdline-tools
```

### B. Download do Command Line Tools

> [!IMPORTANT]
> **Sempre use o link oficial para obter o download mais recente!**
> A Google atualiza as ferramentas de linha de comando constantemente. Acesse a página de downloads do [Android Studio (Seção Command Line Tools Only)](https://developer.android.com/studio#command-line-tools-only), clique no link para **Linux**, aceite os termos e copie o link real do download (geralmente segue o formato abaixo).

Execute os comandos abaixo para baixar e estruturar as ferramentas (substitua o link pelo que você copiou do site oficial):
```bash
cd /tmp
# Acesse o site oficial do Android para pegar o link mais atualizado. Exemplo:
wget https://dl.google.com/android/repository/commandlinetools-linux-14742923_latest.zip
unzip commandlinetools-linux-*_latest.zip

# O sdkmanager exige que os binários estejam dentro de uma pasta chamada "latest"
mv cmdline-tools ~/Android/Sdk/cmdline-tools/latest
```

### C. Configurando Variáveis de Ambiente no `~/.zshrc`

Abra seu arquivo de configuração (`nano ~/.zshrc`) e adicione as seguintes variáveis e caminhos no final do arquivo:

```bash
# Java (ajuste de acordo com a versão instalada, ex: java-17-openjdk ou java-21-openjdk)
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$PATH:$JAVA_HOME/bin

# Android SDK Nativo Linux
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# NDK (Defina e ajuste a versão conforme o NDK exigido pelo seu projeto)
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/27.3.13750724
```

Carregue as novas variáveis:
```bash
source ~/.zshrc
```

### D. Instalação de Pacotes do SDK via `sdkmanager`

Agora que seu `sdkmanager` está configurado no path, vamos usá-lo para instalar as ferramentas de compilação essenciais e aceitar as licenças.

> [!IMPORTANT]
> **Como escolher as versões das plataformas, build-tools e NDK?**
> Abra o seu projeto React Native e olhe o arquivo `android/build.gradle` ou `android/gradle.properties`. Verifique quais versões de `compileSdkVersion` (ou `compileSdk`), `buildToolsVersion` e `ndkVersion` estão declaradas e substitua nas chamadas do `sdkmanager` abaixo para garantir um build local sem erros de mismatch.

1. **Instalar as dependências essenciais do SDK (Exemplo usando API 36 e Build Tools 36.0.0):**
   ```bash
   # Substitua "platforms;android-XX" e "build-tools;XX.X.X" pelas versões do seu projeto
   sdkmanager --install "platform-tools" "platforms;android-36" "build-tools;36.0.0"
   ```
   > [!NOTE]
   > A partir de **31/08/2026**, o Google Play passa a exigir `targetSdk 36` (Android 16) para apps novos e atualizações — API 35 deixa de ser aceita em novas submissões. Se o `build.gradle` do seu projeto ainda estiver em 35, vale planejar a atualização.

2. **Instalar o NDK Linux Nativo (Substitua pela versão de ndkVersion exigida no build.gradle do seu projeto):**
   ```bash
   # Exemplo de instalação do NDK r29 (atual estável no momento da escrita):
   sdkmanager --install "ndk;29.0.14206865"
   ```

   > [!WARNING]
   > **O "pulo do gato" do NDK:**
   > Cada versão do React Native/Expo especifica uma versão **exata** do NDK no arquivo `android/build.gradle` ou `android/gradle.properties` (ex: `ndkVersion = "26.1.10909125"`). Se você instalar apenas a versão genérica sugerida acima e o seu projeto exigir outra, o Gradle tentará baixar a versão correta por conta própria durante o build — gastando gigabytes extras de disco e banda — ou o build falhará. Sempre abra o arquivo do seu projeto, identifique o `ndkVersion` declarado e instale exatamente essa versão com o `sdkmanager`.
   >
   > Além disso, o Android 16 (API 36) passou a exigir que bibliotecas nativas (`.so`) estejam alinhadas a páginas de memória de **16 KB**. Se o seu projeto usa NDK, prefira uma versão r28+ (ou aplique o alinhamento manualmente em versões mais antigas) para evitar rejeição no Google Play.

3. **Aceitar todas as Licenças do Android (Crucial para o Gradle conseguir compilar):**
   ```bash
   sdkmanager --licenses
   # Pressione 'y' e Enter para todas as licenças que aparecerem no terminal
   ```

---

## 🔌 4. Conectando o Celular Físico ao WSL2 (Fluxo Diário)

Siga este processo simples para enviar o seu celular conectado no Windows para o terminal do Linux.

### Passo 1: No Windows Host (PowerShell)
Plugue o seu celular físico no cabo USB e garanta que a **Depuração USB** está ativada nas opções de desenvolvedor do aparelho.

1. Liste os dispositivos USB conectados:
   ```powershell
   usbipd list
   ```
   *Identifique o barramento (BUSID) do seu celular (ex: `1-4` ou `2-1`).*

2. Vincule (attach) o barramento desejado ao WSL2:
   ```powershell
   usbipd attach --wsl --busid <BUSID_DO_SEU_CELULAR>
   ```

### Passo 2: No Arch Linux (Terminal)
Valide que o celular foi integrado com sucesso ao Linux.

1. Liste os dispositivos USB locais:
   ```bash
   lsusb
   # O seu celular físico deve aparecer listado aqui!
   ```

2. Valide com o ADB local do Arch:
   ```bash
   adb devices
   ```
   *Se aparecer `unauthorized`, olhe para a tela do seu celular e autorize a chave de depuração.*
   O terminal deve exibir o ID do dispositivo seguido de `device`.

---

## ⚡ 5. Fluxo de Execução do Projeto React Native

Com o celular enxergado perfeitamente pelo ADB dentro do WSL2, basta configurar as portas de desenvolvimento.

```bash
# 1. Direciona a porta de desenvolvimento do Metro Bundler para o celular físico
adb reverse tcp:8081 tcp:8081

# 2. Acesse seu projeto e inicie
cd ~/projetos/seu-projeto
npm run android
```
> [!NOTE]
> Não existe uma flag `--localhost` no CLI do React Native — quem direciona o Metro para o dispositivo físico é o `adb reverse` do passo 1. Se precisar apontar para um dispositivo específico, use `npm run android -- --deviceId <ID>` (obtido via `adb devices`).

---

## 🎨 6. Atalhos Úteis no seu `~/.zshrc` (Opcional)

Adicione estes aliases úteis para tornar o fluxo diário ainda mais confortável ao abrir o terminal:

```bash
# Alias: Redirecionamento rápido de porta e execução
alias rn-reverse="adb reverse tcp:8081 tcp:8081 && echo '🔄 Porta 8081 redirecionada para o celular!'"

# Atalho para monitorar logs do react native direto do dispositivo
alias adb-log="adb logcat *:S ReactNative:V ReactNativeJS:V"
```

---

## 🧹 7. Desconectando o Aparelho (Ao Final do Dia)

Quando terminar de trabalhar, você pode desconectar o barramento do USB do WSL2 para que ele volte ao Windows host.

**No Windows Host (PowerShell):**
```powershell
usbipd detach --busid <BUSID_DO_SEU_CELULAR>
```

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>
