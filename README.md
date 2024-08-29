# Clone-Linux-system-disks
Clone Linux system disks with or without defects and transform into VM

Recentemente me deparei com um trabalho, transformar uma maquina fisica em VM, até ai tudo bem porém a maquina possui Bad Blocks em uma partição.
E ai começa essa saga explicando como você deve proceder para contornar esses problemas.
Objetivo: Maquina Fisica com Debian + GLPI+GRAFANA+BANCO DE DADOS MARIADB+ DOCKER = CLONAR E VIRTUALIZAR CORRIGIR ERROS DO SISTEMA.

#PROCEDIMENTOS:

Para lidar com bad blocks e garantir que eles sejam tratados adequadamente durante a clonagem, você pode combinar dd com outras ferramentas e técnicas para isolar ou marcar setores defeituosos. Aqui estão algumas abordagens que você pode considerar:
1. Usar ddrescue para Clonagem

O ddrescue é uma ferramenta especializada para fazer cópias de discos com setores defeituosos. Ele é projetado para lidar com discos que têm problemas de leitura, tentando recuperar o máximo de dados possível e registrando os setores defeituosos para que possam ser tratados posteriormente.
Instalar ddrescue:

No Debian ou Ubuntu que tambem pode ser utilizado, você pode instalar ddrescue com o seguinte comando:

bash

sudo apt update
sudo apt install gddrescue

Criar a Imagem com ddrescue:

Use ddrescue para criar a imagem do disco, tratando setores defeituosos de forma mais eficiente do que dd:

bash

sudo ddrescue /dev/sdX /path/to/backup.img /path/to/rescue.log

    /dev/sdX: O disco de origem.
    /path/to/backup.img: O caminho onde a imagem será salva.
    /path/to/rescue.log: O arquivo de log que armazena informações sobre setores defeituosos e progresso.

O ddrescue tenta ler setores defeituosos várias vezes e registra os setores que não podem ser recuperados, permitindo a recuperação de dados críticos.
2. Verificar e Isolar Bad Blocks no Sistema de Arquivos

Antes de clonar, você pode usar ferramentas para verificar e isolar bad blocks no sistema de arquivos.
Para Sistemas de Arquivos ext2/ext3/ext4:

    Executar e2fsck:

    Use e2fsck para verificar e isolar bad blocks no sistema de arquivos. Certifique-se de que o sistema de arquivos não está montado antes de executar o comando.

    bash

    sudo e2fsck -cf /dev/sdXn

        -c: Verifica os blocos defeituosos.
        -f: Força a verificação.
        -y: Assume "sim" para todas as correções.

Para Sistemas de Arquivos NTFS:

Se o sistema de arquivos for NTFS, você pode usar ntfsfix para tentar corrigir problemas básicos, mas ferramentas especializadas do Windows, como chkdsk, podem ser mais eficazes.
3. Criar uma Imagem Ignorando Setores Defeituosos

Se você deseja criar uma imagem do disco ignorando setores defeituosos e não precisa de uma recuperação completa, você pode usar opções específicas de dd para continuar a operação mesmo na presença de erros.
Usar dd com Opções de Ignorar Erros:

bash

sudo dd if=/dev/sdX of=/path/to/backup.img bs=4M conv=noerror,sync

    conv=noerror: Ignora erros de leitura e continua a operação.
    sync: Preenche blocos de erro com zeros.

Resumo dos Passos:

    Usar ddrescue:

    bash

sudo ddrescue /dev/sdX /path/to/backup.img /path/to/rescue.log

Verificar e isolar bad blocks antes da clonagem:

Para ext2/ext3/ext4:

bash

sudo e2fsck -cf /dev/sdXn

Para NTFS, considere usar ferramentas específicas do Windows.

Usar dd com opções para ignorar erros (se necessário):

bash

    sudo dd if=/dev/sdX of=/path/to/backup.img bs=4M conv=noerror,sync

Essas abordagens devem ajudar a lidar com discos que contêm bad blocks e garantir que você consiga criar uma imagem utilizável, mesmo na presença de setores defeituosos.

#CONVERTER .IMG PARA VHD

*Recomendo que você tenha salvo essa imagem em um HD Externo para poder facilitar o transporte mas tambem pode ser via rede e compartilhamento.


Para converter uma imagem de disco (backup.img) para o formato VHDX, você pode usar a ferramenta qemu-img, que faz parte do pacote qemu-utils. 
O qemu-img é uma ferramenta poderosa para converter entre diferentes formatos de imagens de disco, e pode ser usada para converter a imagem raw (ou seja, uma imagem de disco no formato bruto) para o formato VHDX.

Aqui está um passo a passo para converter backup.img para backup.vhdx:
1. Instalar o qemu-utils

Se você ainda não tem o qemu-utils instalado, você precisa instalá-lo. No Debian, você pode fazer isso com o seguinte comando:

bash

sudo apt update
sudo apt install qemu-utils

2. Converter a Imagem com qemu-img

Após instalar o qemu-utils, você pode usar o qemu-img para converter a imagem backup.img para o formato VHDX.

    Converter a Imagem:

    Execute o seguinte comando para converter backup.img para backup.vhdx:

    bash

    qemu-img convert -f raw -O vhdx /path/to/backup.img /path/to/backup.vhdx

        -f raw: Especifica o formato de entrada da imagem (neste caso, raw).
        -O vhdx: Especifica o formato de saída (neste caso, vhdx).
        /path/to/backup.img: O caminho para o arquivo de imagem de entrada.
        /path/to/backup.vhdx: O caminho para o arquivo de imagem de saída.

3. Verificar a Imagem VHDX

Após a conversão, você pode verificar a integridade da imagem VHDX. Embora não seja possível verificar diretamente a integridade do VHDX sem montá-lo, você pode montar a imagem em um ambiente de teste para garantir que ela esteja funcionando corretamente.
Montar a Imagem VHDX (no Linux):

Se você estiver usando Linux, pode montar a imagem VHDX para verificar o conteúdo. Para isso, use qemu-nbd para criar um dispositivo de bloco virtual e depois monte-o.

    Carregar o módulo NBD:

    bash

sudo modprobe nbd

Conectar a imagem VHDX ao dispositivo NBD:

bash

sudo qemu-nbd -c /dev/nbd0 /path/to/backup.vhdx

Montar a partição (se houver):

Liste as partições para identificar a partição correta:

bash

sudo fdisk -l /dev/nbd0

Monte a partição (ajuste a partição conforme necessário):

bash

sudo mount /dev/nbd0p1 /mnt

Desmontar e desconectar o NBD após a verificação:

bash

    sudo umount /mnt
    sudo qemu-nbd -d /dev/nbd0

Considerações Finais para você:

    Espaço em Disco: Certifique-se de que você tem espaço suficiente no disco para armazenar a imagem VHDX resultante dessa conversão. Imagens de disco podem ser grandes e podem exigir bastante espaço de armazenamento.
    Consistência dos Dados: Após converter a imagem, é uma boa prática verificar a consistência e integridade dos dados, especialmente se a imagem original continha setores defeituosos.
    Importante lembrar esse VHDX pode ser utilizado em Virtual Box Hypervi em fim você ja deve ter entendido né, eu porem me deparei com algo organizar o dhcp e dns da clonagem para ter acesso a internet.
    Isso que fiz no final para ter acesso e atualizar, tive que corrigir uns erros de disco tambem.
    Backup: Sempre mantenha backups dos dados importantes antes de realizar operações críticas, faça uma copia da imagem em fim nunca realize testes sem copias como conversões de imagens de disco.

