# Select-Boot
Este projeto tem como inspiração um projeto desenvolvido por [Stephen Holdaway](https://github.com/stecman/hw-boot-selection), 
onde ele cria um hardware para selecionar qual Sistema Operacional será inicializado no momento do Boot do sistema, 
ambos os projeto (meu e dele) tem como principal objetivo, trabalhar em cima do Grub.

Veja esse [site](https://hackaday.io/project/179539-hardware-boot-selection-switch) para mais detalhes do projeto dele.

## O que você vai precisar?
Você vai precisar de um Sistema Linux em dual-boot com Windows, a escolha entre qual sistema deve inicializar deve ser feita usando Grub,
caso você use outro Bootloader, esse projeto pode não funcionar sem algumas adaptações.

Você vai precisar também de um pendrive, vamos usar ele para indicar para o Grub que, 
caso o pendrive esteja conectado, ele deve iniciar o Windows, caso não esteja, ele vai inicializar o Sistema padrão.

Um benefício é que você ainda vai poder usar seu pendrive como pendrive, além dele também ser sua chave para iniciar o Windows automaticamente.

## Colocando a mão na massa
Antes de tudo, no terminal, rode o comando `blkid` para obter o UUID do pendrive que você escolheu (o pendrive precisa estar conectado ;) ).
```
$ sudo blkid 
/dev/sde1: UUID="CA0A-5452" TYPE="vfat" PARTUUID="800f8612-01"

# Acima o exemplo do pendrive que escolhi, removi as demais indentificações para não atrapalhar.
```
Agora com o UUID *CA0A-5452* em mãos, vamos modifiar o arquivo principal do Grub, 
você deve modificar o arquivo `/etc/grub.d/00_header`, troque-o no código abaixo pelo seu:
```
cat << EOF
# Procura o pendrive conectado:
search --no-floppy --fs-uuid --set foundboot CA0A-545

# Caso encontre:
if [ "\${foundboot}" ] ; then
  # Seleciona o boot do Windows:
  set default="2"
else
  # Se nao encontrar, usa o boot padrão (normalmente é o Linux)
  set default="${GRUB_DEFAULT}"
fi
EOF
```

Eu tentei colocar o `if/else` sem usar *Here document*, mas não funcionou para mim, 
então eu vi que muitas funções nesse arquivo usam *Here document*, quando eu passei a usar também, 
pude notar que minha alteração nesse arquivo funcionou.

Vale notar algumas coisas nesse código que eu passei:
```
# Vamos focar na linha abaixo:
search --no-floppy --fs-uuid --set foundboot CA0A-5452
```
Sem entrar em muitos detalhes, *foundboot* é uma variável, você pode dar o nome que quiser.
Já o ID *CA0A-5452*, é o UUDI do FileSystem do pendrive, como mostrado mais acima, dessa forma, 
vamos ter esse pendrive sendo reconhecido unicamente em todo o sistema, até que você formate esse pendrive.
  
Vale ressaltar que ***set default="2"*** é referente ao Boot do Windows, no seu caso pode ser que seja diferente, então altere caso necessário.
```
Você pode descobrir qual o número do Boot do Windows olhando para seu Grub, veja um exemplo do meu Grub:
  Linux Mint 20 Cinnamon
  Advanced options for Linux Mint 20 Cinnamon
  Windows Boot Manager (on /dev/sda1)
  Arch Linux (on /dev/sdc5)
  Advanced options for Arch Linux (on /dev/sdc5)
  UEFI Firmware Settings

A contagem se inicia em 0, sendo a opção do Windows a de número 2, veja a saída abaixo para facilitar:
N° = Opção
0 =  Linux Mint 20 Cinnamon
1 =  Advanced options for Linux Mint 20 Cinnamon
2 =  Windows Boot Manager (on /dev/sda1)
3 =  Arch Linux (on /dev/sdc5)
4 =  Advanced options for Arch Linux (on /dev/sdc5)
5 =  UEFI Firmware Settings
```

Agora que você colocou essa linha no seu arquivo, com a informações corretas para seu pendrive, 
gere um novo arquivo de configuração do Grub:
```
$ sudo update-grub
```
Agora vamos no certificar que nossa alteração entrou em vigor, acesse o arquivo `/boot/grub/grub.cfg`,
e verifique se você consegue ver a sua alteração nesse arquivo, segue um exemplo do meu:
```
119 # Procura o pendrive conectado:
120 search --no-floppy --fs-uuid --set foundboot CA0A-5452
121 
122 # Caso encontre:
123 if [ "${foundboot}" ] ; then
124   # Seleciona o boot do Windows:
125   set default="2"
126 else
127   # Se nao encontrar, usa o boot padrão (normalmente e o Linux)
128   set default="0"
129 fi
130 ### END /etc/grub.d/00_header ###
```
Podemos notar que a nossa alteração está no arquivo, tendo início na linha 119.
Agora quando ligar/reiniciar seu Linux, coloque o pendrive em qualquer USB e o Windows será selecionado no Boot.
