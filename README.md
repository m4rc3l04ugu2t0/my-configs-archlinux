```markdown
# Instalação do Arch Linux com Particionamento Btrfs

## Passos de Pré-Instalação

### 1. Seta o Teclado
```bash
loadkeys br-abnt2
```
- `loadkeys`: Carrega o layout do teclado.
- `br-abnt2`: Layout do teclado brasileiro ABNT2.

### 2. Conectar ao Wi-Fi
```bash
rfkill unblock all
iwctl station list
iwctl
```
No prompt do `iwctl`:
```bash
station list
station wlan0 get-networks
station wlan0 connect nome-da-rede
```
- `rfkill unblock all`: Desbloqueia todos os dispositivos de rádio.
- `iwctl`: Ferramenta de linha de comando do iwd.
- `station wlan0 connect nome-da-rede`: Conecta à rede especificada.

### 3. Atualizar o Relógio do Sistema
```bash
timedatectl set-ntp true
```
- `timedatectl set-ntp true`: Habilita a sincronização automática do tempo via NTP.

## Particionamento do Disco

### 4. Particionar o Disco usando `fdisk`
```bash
fdisk /dev/nvme0n1
```
Crie as seguintes partições:
- `/dev/nvme0n1p1`: Partição EFI, 512 MB
- `/dev/nvme0n1p2`: Partição Btrfs, restante do espaço

### 5. Formatar as Partições
```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -L ArchLinux /dev/nvme0n1p2
```
- `mkfs.fat -F32 /dev/nvme0n1p1`: Formata a partição EFI como FAT32.
- `mkfs.btrfs -L ArchLinux /dev/nvme0n1p2`: Formata a partição como Btrfs e define o label como "ArchLinux".

### 6. Criar e Montar Subvolumes Btrfs

#### 6.1 Montar a Partição Btrfs Temporariamente
```bash
mount /dev/nvme0n1p2 /mnt
```

#### 6.2 Criar Subvolumes
```bash
btrfs su cr /mnt/@
btrfs su cr /mnt/@home
btrfs su cr /mnt/@var
btrfs su cr /mnt/@log
btrfs su cr /mnt/@cache
btrfs su cr /mnt/@snapshots
```
- `btrfs su cr`: Cria subvolumes Btrfs.

#### 6.3 Desmontar a Partição Temporária
```bash
umount /mnt
```

#### 6.4 Montar os Subvolumes Corretamente
```bash
mount -o noatime,compress=zstd,space_cache=v2,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{boot,home,var,log,cache,.snapshots}
mount /dev/nvme0n1p1 /mnt/boot
mount -o noatime,compress=zstd,space_cache=v2,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o noatime,compress=zstd,space_cache=v2,subvol=@var /dev/nvme0n1p2 /mnt/var
mount -o noatime,compress=zstd,space_cache=v2,subvol=@log /dev/nvme0n1p2 /mnt/log
mount -o noatime,compress=zstd,space_cache=v2,subvol=@cache /dev/nvme0n1p2 /mnt/cache
mount -o noatime,compress=zstd,space_cache=v2,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
```
- `noatime`: Não atualiza a data de acesso para melhorar a performance.
- `compress=zstd`: Ativa a compressão Zstandard.
- `space_cache=v2`: Ativa a nova versão do cache de espaço.

## Instalação do Sistema Base

### 7. Instalar o Sistema Base do Arch Linux
```bash
pacstrap /mnt base linux linux-firmware base-devel vim dhcpcd
```

### 8. Gerar o `fstab`
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

### 9. Entrar no Novo Sistema Instalado
```bash
arch-chroot /mnt
```

## Configurações Pós-Instalação

### 10. Configurar o Timezone
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
date
```

### 11. Configurar a Localização
```bash
vim /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf
```

### 12. Configurar o Hostname
```bash
echo "arch" > /etc/hostname
echo "127.0.0.1    localhost" >> /etc/hosts
echo "::1          localhost" >> /etc/hosts
echo "127.0.1.1    meu-hostname.localdomain meu-hostname" >> /etc/hosts
```

### 13. Definir Senha do Root
```bash
passwd
```

### 14. Criar Usuário
```bash
useradd -m -g users -G wheel,video,audio,kvm -s /bin/bash arch
passwd arch
```

### 15. Instalar Pacotes Essenciais
```bash
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant wireless_tools dialog sudo sudo bluez bluez-utils snapper
```

### 16. Instalar o Bootloader GRUB
```bash
vim /etc/mkinitcpio.conf
MODULES=(btrfs)
mkinitcpio -p linux
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### 17. Configurar zRAM para Swap

#### 17.1 Instalar zRAM sem `yay`
```bash
sudo pacman -S zramswap
```

#### 17.2 Configurar zRAM
Edite o arquivo de configuração localizado em `/etc/default/zramswap`:
```sh
sudo vim /etc/default/zramswap
```
Exemplo de configuração:
```sh
# ZRAM size percentage of total RAM, default is 50%
PERCENTAGE=50

# compression algorithm selection, default is lz4
COMPRESSION_ALGORITHM=lz4

# the maximum size of each zram device, default is 4G
MAX_SIZE=4G
```

#### 17.3 Habilitar e iniciar o serviço `zramswap`
```bash
sudo systemctl enable zramswap.service
sudo systemctl start zramswap.service
```

### 18. Configurar Snapper
```bash
sudo snapper -c root create-config /
sudo btrfs subvolume delete /.snapshots
sudo mkdir /.snapshots
sudo mount -a
sudo chmod 750 /.snapshots
sudo chown :nomequevocequiser /.snapshots
```

#### Configurar Limites de Snapshots
Edite o arquivo `/etc/snapper/configs/root` para configurar os limites de snapshots:
```sh
# /etc/snapper/configs/root
SUBVOLUME="/"
FSTYPE="btrfs"

# limits for timeline cleanup
TIMELINE_MIN_AGE="1800"
TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="0"
TIMELINE_LIMIT_MONTHLY="0"
TIMELINE_LIMIT_YEARLY="0"
```

#### Ativar Timers do Snapper
```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

### 19. Adicionar Privilégios de Root ao Usuário
```bash
su -
EDITOR=vim visudo
```
Descomente a linha:
```sh
%wheel ALL=(ALL:ALL) ALL
```

### 20. Ativar Processos Essenciais
```bash
systemctl enable NetworkManager --now
systemctl enable dhcpcd --now
```

### 21. Verificar Conexão com a Internet
```bash
ping google.com
```

### 22. Instalar Codecs e Atualizar o Sistema
```bash
pacman -Sy gstreamer ffmpeg gst-plugins-ugly gst-plugins-good gst-plugins-base gst-plugins-bad gst-libav
pacman -Syyuu
```

### 23. Configurar Espelhos e Drivers Gráficos
```bash
pacman -S reflector
reflector -c Brazil --save /etc/pacman.d/mirrorlist
sudo pacman -S xf86-video-amdgpu mesa
```

### 24. Instalar o Ambiente de Desenvolvimento

#### Rust
```bash
pacman -S rustup
rustup-init
```

#### Python
```bash
pacman -S python
```

#### C e C++
```bash
pacman -S gcc
```

#### Arduino
```bash
pacman -S arduino
```

### 25. Instalar Editores e Ferramentas

#### LunarVim
```bash
git clone https://github.com/ChristianChiarulli/LunarVim.git ~/.local/share/lunarvim
cd ~/.local/share/lunarvim
./install.sh
```

#### Visual Studio Code
```bash
yay -S visual-studio-code-bin
```

#### Alacritty
```bash
pacman -S alacritty
```

#### Microsoft Edge
```bash
yay -S microsoft-edge-dev-bin
```

### 26. Instalar Fontes e Starship Prompt

#### Fonte FiraCode
```bash
yay -S otf-fira-code
```

#### Starship Prompt
```bash
yay -S starship
```

### 27. Instalar VirtualBox e Ferramentas de Virtualização
```bash
pacman -S virtualbox virtualbox-host-dkms linux-headers
```

### 28. Instalar OBS Studio
```bash
pacman -S obs-studio
```

### 29. Instalar Gerenciador de Arquivos
```bash
pacman -S nautilus
```

### 30. Instalar VLC Media Player
```bash
pacman -S vlc
```

## Finalização e Reinicialização

### 31. Reiniciar o Sistema
```bash
reboot
```

Após reiniciar, você terá o Arch Linux configurado com Btrfs, Snapper para snapshots, zRAM para melhorar o desempenho, além de ferramentas de desenvolvimento como Rust, Python, C/C++, Arduino, editores como LunarVim e Visual Studio Code, terminais como Alacritty, navegadores como Microsoft Edge, fontes FiraCode para o Starship Prompt, VirtualBox para virtualização, OBS Studio para streaming e gravação, um gerenciador de arquivos (Nautilus) e o VLC Media Player para reprodução de mídia.

Certifique-se de ajustar os comandos de instalação conforme necessário para o seu ambiente específico. Caso precise de mais alguma ajuda, estou à disposição!
```

Este arquivo `README.md` contém todos os passos necessários para configurar seu sistema Arch Linux com todas as ferramentas e configurações solicitadas. Lembre-se de adaptar conforme necessário para o seu ambiente específico ou preferências pessoais.
