# Instalar WSL2 e Configurar o Arch Linux

### Ativação Manual

- Ativar o recusdo de **Subsistema do Windows para Linux** na tela de Recursos do Windows
- Ativar o recurso de **Hyper-V** na tela de Recursos do Windows
- Ativar o Recurso de **Plataforma de Máquina Virtual** na tela de Recursos do Windows
- Seguir as Etapas descritas no site da Microsoft, pulando as etapas das ativações

### Instalação do Arch Linux

- Download: <https://github.com/yuk7/ArchWSL>
- Descampactar e colocar os arquivos no C:\Arch
- Com o Terminal Windows na pasta C:\Arch, executar: `.\Arch.exe`
- Fechar o Terminal e Abrir novamente
- Iniciar o Terminal com o Arch Linux
- No Terminal Arch: `passwd`
- No Terminal Arch: `echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel`
- No Terminal Arch: `useradd -m -G wheel -s /bin/bash {username}`
- No Terminal Arch: `passwd {username}`
- No Terminal Arch: `exit`
- Com o Terminal Windows na pasta C:\Arch, executar: `.\Arch.exe config --default-user {username}`

### Inicializar e Atualizar o Arch

- No Terminal Arch: `sudo nano /etc/pacman.conf`
- No arquivo de configuração, procurar por **ParallelDownloads** e Alterar para 10 (no meu caso)
- No Terminal Arch: `sudo pacman-key --init`
- No Terminal Arch: `sudo pacman-key --populate`
- No Terminal Arch: `sudo pacman -Sy archlinux-keyring`
- No Terminal Arch: `sudo pacman -Su`
- No Terminal Arch: `sudo pacman -S git base-devel`

### Instalando o YAY - Gerenciador de Pacotes AUR

- Projeto YAY: <https://github.com/Jguer/yay>
- No Terminal Arch: `cd /tmp`
- No Terminal Arch: `git clone https://aur.archlinux.org/yay-bin.git`
- No Terminal Arch: `cd yay-bin`
- No Terminal Arch: `makepkg -si`

### Instalando o Shell ZSH e configurando o Powerlevel10k

- No Terminal Arch: `yay -S zsh`
- Projeto Powerlevel10k: <https://github.com/romkatv/powerlevel10k>
- Clique no link de **Installation**, depois no link **Arch Linux**
- No Terminal Arch: `yay -S --noconfirm zsh-theme-powerlevel10k-git`
- No Terminal Arch: `echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc`
- No Terminal Arch: `chsh -s /usr/bin/zsh`
- No Terminal Arch: `exit`
- Projeto Powerlevel10k: <https://github.com/romkatv/powerlevel10k>
- Clique no link de **Fonts**
- Baixe e Instale as fonts **MesloLGS** no Windows
- Abra o Terminal
- Altere a Font nas configurações de Aparência do Terminal Arch
- Abra o Terminal no Arch e Realizar a configuração do Powerlevel10k
- Caso, ao finalizar essa configuração, queira refazer. No terminal Arch: `p10k configure`

### Instalando e Configurando o ASDF

- ASDF é um gerenciador de versão para linguagens
- ASDF VM: <https://asdf-vm.com/>
- No Terminal Arch: `yay -S asdf-vm`
- No Terminal Arch: `source /opt/asdf-vm/asdf.sh`
- Adicionar o comando acima no arquivo **.zshrc** para o ASDF iniciar ao abrir o terminal

### Adicionando o NodeJS com o ASDF

- No Terminal Arch: `asdf plugin add nodejs`
- No Terminal Arch: `asdf list-all nodejs`
- No Terminal Arch: `asdf install nodejs lts`
- Para setar uma versão do NodeJS especifica para o projeto, na pasta do projeto, execute
- No Terminal Arch: `asdf local nodejs 18.15.0`
