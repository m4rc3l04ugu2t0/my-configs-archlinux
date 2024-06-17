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
pacman -Sy dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant wireless_tools dialog sudo
```

### 16. Instalar o Bootloader GRUB
```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### 17. Configurar zRAM para Swap
```bash
pacman -S zramswap
systemctl enable zramswap.service
```

### 18. Sair do `chroot`, Desmontar as Partições e Reiniciar
```bash
exit
umount -R /mnt
reboot
```

### 19. Adicionar Privilégios de Root ao Usuário
```bash
su -
EDITOR=vim visudo
```
Descomentar a linha:
```
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

### 24. Instalar o Ambiente de Desktop KDE Plasma
```bash
pacman -S xorg-server xorg-xinit plasma-wayland-session plasma xorg-xwayland
systemctl enable sddm.service
```
Em caso de erro:
```bash
sudo pacman -S sddm
systemctl disable lightdm.service
systemctl enable sddm.service
```

### 25. Reiniciar o Sistema
```bash
reboot
```

Pronto! Agora você tem o Arch Linux instalado com particionamento Btrfs e otimizado para desempenho.
```

Certifique-se de ajustar os nomes das partições e subvolumes conforme necessário para o seu sistema específico.
