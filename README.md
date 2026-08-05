# 🏔️ WSL2 with Arch Linux: Setup Guides

Bem-vindo aos meus guias personalizados de configuração para o **WSL2** rodando **Arch Linux**. Este repositório foca em alto desempenho, isolamento de ferramentas e uma experiência premium de desenvolvimento no Windows 11.

---

## 🏗️ Escolha o seu Ambiente

Existem dois caminhos principais documentados aqui, dependendo do seu foco de trabalho:

### 1. 🚀 [Fullstack Development](./Fullstack-Development-in-WSL2.md)
Guia de configuração do zero, incluindo:
- Ativação de recursos do Windows.
- Instalação e atualização do Arch Linux.
- Configuração de **ZSH**, **NVM**, **Pyenv** e **Podman (Rootless)**.
- Otimizações de hardware via `.wslconfig`.

### 2. 📱 [Android SDK Nativo](./Android-SDK-Native-WSL2.md)
Setup especializado para desenvolvimento mobile 100% nativo (sem Android Studio), cobrindo:
- Instalação do JDK, Android SDK (cmdline-tools) e NDK direto no Arch Linux.
- Passthrough de dispositivo físico via USB (usbipd-win) — sem emulador.
- Fluxo de trabalho otimizado para React Native, Expo e Flutter.

---

## 🛠️ Requisitos Rápidos
| Item | Descrição |
| :--- | :--- |
| **OS** | Windows 11 PRO (Altamente recomendado) |
| **WSL** | Versão 2 (Kernel atualizado) |
| **Hardware** | Virtualização de hardware ativa na BIOS |

---

<p align="center">
  Criado com ❤️ por <a href="https://github.com/DSanches92">Danilo Sanches</a>
</p>
