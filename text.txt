# Seta o teclado
loadkeys br-abnt2
# loadkeys: Comando para carregar o layout do teclado.
# br-abnt2: Especifica o layout do teclado brasileiro ABNT2.

# Conectar ao Wi-Fi
rfkill unblock all
# rfkill: Gerencia dispositivos de rádio (Wi-Fi, Bluetooth, etc.).
# unblock all: Desbloqueia todos os dispositivos de rádio.

iwctl station list
# iwctl: Ferramenta de linha de comando do iwd para gerenciar conexões Wi-Fi.
# station list: Lista todas as interfaces de estação Wi-Fi.

iwctl
# iwctl: Inicia a ferramenta de linha de comando do iwd.

# Dentro do iwctl
station list        
# station list: Lista todas as interfaces de estação Wi-Fi.

station wlan0 get-networks
# station wlan0 get-networks: Lista todas as redes disponíveis para a interface wlan0.

station wlan0 connect nome-da-rede
# station wlan0 connect nome-da-rede: Conecta à rede especificada.

# Atualizar o relógio do sistema
timedatectl set-ntp true
# timedatectl: Comando para gerenciar configurações de data e hora.
# set-ntp true: Habilita a sincronização automática do tempo via NTP (Network Time Protocol).

# Particionar o disco usando fdisk
fdisk /dev/nvme0n1
# fdisk: Utilitário de particionamento de disco.
# /dev/nvme0n1: Especifica o disco NVMe.

# Criar a tabela de partições:
# - /dev/nvme0n1p1: Partição EFI, 512 MB
# - /dev/nvme0n1p2: Partição Btrfs, restante do espaço

# Formatar as partições
mkfs.fat -F32 /dev/nvme0n1p1
# mkfs.fat: Cria um sistema de arquivos FAT.
# -F32: Especifica o formato FAT32.
# /dev/nvme0n1p1: Partição EFI.

mkfs.btrfs /dev/nvme0n1p2
# mkfs.btrfs: Cria um sistema de arquivos Btrfs.
# /dev/nvme0n1p2: Partição Btrfs.

# Montar a partição Btrfs temporariamente para criar subvolumes
mount /dev/nvme0n1p2 /mnt
# mount: Monta um sistema de arquivos.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt: Ponto de montagem.

# Criar subvolumes Btrfs
btrfs su cr /mnt/@
# btrfs su cr: Cria um subvolume.
# /mnt/@: Subvolume para root.

btrfs su cr /mnt/@home
# /mnt/@home: Subvolume para home.

btrfs su cr /mnt/@snapshots
# /mnt/@snapshots: Subvolume para snapshots.

btrfs su cr /mnt/@var_log
# /mnt/@var_log: Subvolume para /var/log.

btrfs su cr /mnt/@var_cache_pacman
# /mnt/@var_cache_pacman: Subvolume para /var/cache/pacman/pkg.

# Desmontar a partição temporária
umount /mnt
# umount: Desmonta um sistema de arquivos.
# /mnt: Ponto de montagem a ser desmontado.

# Montar os subvolumes corretamente
mount -o subvol=@ /dev/nvme0n1p2 /mnt
# mount: Monta um sistema de arquivos.
# -o subvol=@: Especifica o subvolume a ser montado.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt: Ponto de montagem.

mkdir /mnt/{boot,home,.snapshots,var/log,var/cache/pacman/pkg}
# mkdir: Cria diretórios.
# /mnt/{boot,home,.snapshots,var/log,var/cache/pacman/pkg}: Diretórios a serem criados.

mount /dev/nvme0n1p1 /mnt/boot
# mount: Monta um sistema de arquivos.
# /dev/nvme0n1p1: Partição a ser montada.
# /mnt/boot: Ponto de montagem da partição EFI.

mount -o subvol=@home /dev/nvme0n1p2 /mnt/home
# mount: Monta um sistema de arquivos.
# -o subvol=@home: Especifica o subvolume a ser montado.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt/home: Ponto de montagem do subvolume home.

mount -o subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
# mount: Monta um sistema de arquivos.
# -o subvol=@snapshots: Especifica o subvolume a ser montado.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt/.snapshots: Ponto de montagem do subvolume snapshots.

mount -o subvol=@var_log /dev/nvme0n1p2 /mnt/var/log
# mount: Monta um sistema de arquivos.
# -o subvol=@var_log: Especifica o subvolume a ser montado.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt/var/log: Ponto de montagem do subvolume var_log.

mount -o subvol=@var_cache_pacman /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg
# mount: Monta um sistema de arquivos.
# -o subvol=@var_cache_pacman: Especifica o subvolume a ser montado.
# /dev/nvme0n1p2: Partição a ser montada.
# /mnt/var/cache/pacman/pkg: Ponto de montagem do subvolume var_cache_pacman.

# Instalar o sistema base do Arch Linux
pacstrap /mnt base linux linux-firmware base-devel vim dhcpcd
# pacstrap: Instala pacotes no sistema montado.
# /mnt: Diretório onde o sistema está montado.
# base, linux, linux-firmware, base-devel, vim, dhcpcd: Pacotes a serem instalados.

# Gerar o fstab
genfstab -U /mnt >> /mnt/etc/fstab
# genfstab: Gera um arquivo fstab.
# -U: Usa UUIDs para os sistemas de arquivos.
# /mnt: Diretório onde o sistema está montado.
# >> /mnt/etc/fstab: Acrescenta a saída ao arquivo fstab.

# Verificar o arquivo fstab para garantir que as entradas estão corretas
cat /mnt/etc/fstab
# cat: Exibe o conteúdo de um arquivo.
# /mnt/etc/fstab: Arquivo fstab gerado.

# Entrar no novo sistema instalado
arch-chroot /mnt
# arch-chroot: Ferramenta para trocar root para o novo sistema instalado.
# /mnt: Diretório onde o sistema está montado.

# Configurar o timezone
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
# ln -sf: Cria um link simbólico.
# /usr/share/zoneinfo/Region/City: Caminho do timezone.
# /etc/localtime: Destino do link simbólico.

hwclock --systohc
# hwclock --systohc: Define o relógio de hardware a partir do relógio do sistema.

# Verificar data e hora
date
# date: Exibe a data e hora atuais.

# Configurar a localização
vim /etc/locale.gen
# vim: Editor de texto.
# /etc/locale.gen: Arquivo de configuração de locales.

locale-gen
# locale-gen: Gera os locales especificados.

echo KEYMAP=br-abnt2 > /etc/vconsole.conf
# echo KEYMAP=br-abnt2 > /etc/vconsole.conf: Define o layout do teclado para o console virtual.

# Configurar o hostname
echo arch > /etc/hostname
# echo arch > /etc/hostname: Define o hostname para "arch".

# Adicionar correspondências no arquivo /etc/hosts
echo "127.0.0.1    localhost" >> /etc/hosts
# echo: Adiciona a linha especificada ao arquivo /etc/hosts.

echo "::1          localhost" >> /etc/hosts
# echo: Adiciona a linha especificada ao arquivo /etc/hosts.

echo "127.0.1.1    meu-hostname.localdomain meu-hostname" >> /etc/hosts
# echo: Adiciona a linha especificada ao arquivo /etc/hosts.

# Definir senha do root
passwd
# passwd: Define a senha do usuário root.

# Criar usuário
useradd -m -g users -G wheel,video,audio,kvm -s /bin/bash arch
# useradd: Adiciona um novo usuário.
# -m: Cria um diretório home para o usuário.
# -g users: Define o grupo primário do usuário.
# -G wheel,video,audio,kvm: Define grupos adicionais para o usuário.
# -s /bin/bash: Define o shell do usuário.
# arch: Nome do usuário.

passwd arch
# passwd arch: Define a senha para o usuário "arch".

# Pacotes essenciais
pacman -Sy dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant wireless_tools dialog sudo
# pacman: Gerenciador de pacotes do Arch Linux.
# -Sy: Sincroniza os repositórios e instala os pacotes especificados.
# dosfstools, os-prober, mtools, network-manager-applet, networkmanager, wpa_supplicant, wireless_tools, dialog, sudo: Pacotes a serem instalados.

# Instalar o bootloader GRUB
pacman -S grub efibootmgr
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala os pacotes especificados.
# grub, efibootmgr: Pacotes a serem instalados.

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch --recheck
# grub-install: Instala o GRUB.
# --target=x86_64-efi: Especifica o alvo para o GRUB.
# --efi-directory=/boot: Especifica o diretório EFI.
# --bootloader-id=arch: Define o identificador do bootloader.
# --recheck: Verifica novamente todos os dispositivos.

grub-mkconfig -o /boot/grub/grub.cfg
# grub-mkconfig: Gera o arquivo de configuração do GRUB.
# -o /boot/grub/grub.cfg: Especifica o local de saída do arquivo de configuração.

# Configurar zRAM para swap
pacman -S zramswap
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala o pacote especificado.
# zramswap: Pacote a ser instalado.

systemctl enable zramswap.service
# systemctl: Gerencia serviços do systemd.
# enable: Habilita o serviço para iniciar automaticamente.
# zramswap.service: Serviço a ser habilitado.

# Sair do chroot, desmontar as partições e reiniciar
exit
# exit: Sai do chroot.

umount -R /mnt
# umount: Desmonta um sistema de arquivos.
# -R: Recursivamente desmonta todos os sistemas de arquivos montados em /mnt.

reboot
# reboot: Reinicia o sistema.

# Adicionar privilégios de root ao usuário
su -
# su -: Troca para o usuário root.

EDITOR=vim visudo
# EDITOR=vim visudo: Abre o arquivo sudoers no editor Vim para edição.

# Descomentar a linha => %wheel ALL=(ALL:ALL) ALL
# No editor, encontre a linha com `%wheel ALL=(ALL:ALL) ALL` e remova o símbolo de comentário (#) para dar privilégios de sudo aos usuários do grupo wheel.

echo "LANG=en_US.UTF-8" > /etc/locale.conf
# echo "LANG=en_US.UTF-8" > /etc/locale.conf: Define a variável de ambiente LANG para en_US.UTF-8.

# Ativar processos
systemctl enable NetworkManager --now
# systemctl: Gerencia serviços do systemd.
# enable: Habilita o serviço para iniciar automaticamente.
# NetworkManager: Serviço a ser habilitado.
# --now: Inicia o serviço imediatamente.

systemctl enable dhcpcd --now
# systemctl: Gerencia serviços do systemd.
# enable: Habilita o serviço para iniciar automaticamente.
# dhcpcd: Serviço a ser habilitado.
# --now: Inicia o serviço imediatamente.

# Verificar se está com internet
ping google.com
# ping: Envia pacotes ICMP para testar a conectividade de rede.
# google.com: Endereço a ser testado.

# Codecs
pacman -Sy gstreamer ffmpeg gst-plugins-ugly gst-plugins-good gst-plugins-base gst-plugins-bad gst-libav
# pacman: Gerenciador de pacotes do Arch Linux.
# -Sy: Sincroniza os repositórios e instala os pacotes especificados.
# gstreamer, ffmpeg, gst-plugins-ugly, gst-plugins-good, gst-plugins-base, gst-plugins-bad, gst-libav: Pacotes a serem instalados.

pacman -Syyuu
# pacman: Gerenciador de pacotes do Arch Linux.
# -Syyuu: Força a sincronização dos repositórios e atualiza todos os pacotes para a versão mais recente.

pacman -S reflector
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala o pacote especificado.
# reflector: Pacote a ser instalado.

reflector -c Brazil --save /etc/pacman.d/mirrorlist
# reflector: Ferramenta para gerenciar a lista de espelhos do Arch Linux.
# -c Brazil: Seleciona espelhos do Brasil.
# --save /etc/pacman.d/mirrorlist: Salva a lista de espelhos em /etc/pacman.d/mirrorlist.

sudo pacman -S xf86-video-amdgpu mesa
# sudo: Executa um comando como superusuário.
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala os pacotes especificados.
# xf86-video-amdgpu, mesa: Pacotes a serem instalados.

pacman -S xorg-server xorg-xinit plasma-wayland-session plasma xorg-xwayland
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala os pacotes especificados.
# xorg-server, xorg-xinit, plasma-wayland-session, plasma, xorg-xwayland: Pacotes a serem instalados.

systemctl enable sddm.service
# systemctl: Gerencia serviços do systemd.
# enable: Habilita o serviço para iniciar automaticamente.
# sddm.service: Serviço a ser habilitado.

# Caso de erro
sudo pacman -S sddm
# sudo: Executa um comando como superusuário.
# pacman: Gerenciador de pacotes do Arch Linux.
# -S: Instala o pacote especificado.
# sddm: Pacote a ser instalado.

systemctl disable lightdm.service
# systemctl: Gerencia serviços do systemd.
# disable: Desabilita o serviço para não iniciar automaticamente.
# lightdm.service: Serviço a ser desabilitado.

systemctl enable sddm.service
# systemctl: Gerencia serviços do systemd.
# enable: Habilita o serviço para iniciar automaticamente.
# sddm.service: Serviço a ser habilitado.

reboot
# reboot: Reinicia o sistema.
