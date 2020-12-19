# dicas.md

Dicas anotadas a partir de problemas reais, alguns irão funcionar, outros não:

# Índice
# 1. Recuperando o boot [#1. Recuperando o boot]




# 1. Recuperando Boot

Ao entrar no Grub Shell algumas poucas opões são oferecidas, pois trata-se de um mini-shell.

E porque ele entra no Grub Shell?
Pode ser que nao tenha sido encontrado o initrd ou o kernel por exemplo.

Para nos situarmos podemos usar o comando `ls` pra verificar as partições...

```
grub> ls
(hd0) (hd0,msdos1) (hd1) (hd1,msdos7) ...
```

Para listar o conteúdo disponivel nas partiçoes usamos o `/` ao final do comando:
```
grub> ls -l (hd0,msdos1)/
```

Neste caso estamos na raiz do sistema, podemos analisar se o initrd e o kernel estão presentes:

```
grub> ls -l (hd0,msdos1)/boot/

```

Caso ambos estejam presentes, podemos bootar o sistema a partir daqui, com os seguintes comandos:

* Primeiramente setar a partição que iremos trabalhar:

```
grub> set root=(hd0,msdos1)
```

Agora podemos definir o kernel e initrd a serem inicializados:

* Kernel

```
grub> linux /boot/vmlinuz-4.15.0-66-generic root=/dev/sda1
```

* Initrd
```
grub> initrd /boot/initrd.img-4.15.0-66-generic
```

Se as informações estiverem corretas, neste ponto bastar digitar o comando `boot` que será inicializado.

