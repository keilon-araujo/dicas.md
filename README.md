# dicas.md

Dicas anotadas a partir de problemas reais, alguns irão funcionar, outros não:

## Índice
### 1. Recuperando o boot 
### 2. Cliente RDP no Linux 
### 3. Conectando no wifi com wpa_supplicant 
### 4. Adicionado chave SSH ao git 
### 5. Ativando/desativando hibernação no systemd




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

Após iniciado o sistema, e verificar partições, arquivo de configuração do grub, um item inicial que pode resolver é o `update-grub`, que irá recriar o arquivo de configuração:
```
keilon@capsule$ sudo update-grub
```

É possível que não funcione, outro método é reinstalar o grub:
```
grub-install /dev/sda

```

# 2. Cliente RDP no Linux

Para acessar clientes RDP temos disponível nos repositórios do debian buster o `xfreerdp`.

```
keilon@capsule:~$ xfreerdp

FreeRDP - A Free Remote Desktop Protocol Implementation
See www.freerdp.com for more information

Usage: xfreerdp [file] [options] [/v:<server>[:port]]

Syntax:
    /flag (enables flag)
    /option:<value> (specifies option with value)
    +toggle -toggle (enables or disables toggle, where '/' is a synonym of '+') 

```

Ele é bastante simpes e é executado via CLI, caso as instruções sejam enviadas corretamente, ele irá apresentar a tela da máquina a qual estamos tentando acesso.

Neste exemplo vou tentar acessar a máquina com endereço IP 10.160.88.45 com usuário 92041410.

```
keilon@capsule:~$ xfreerdp -u 92041410 10.160.88.44

```

Caso tenha um domínio ele será solicitado de modo interatido no terminal.

Vale ressaltar que existem várias outras opções dentro do comando, cabe ver o manul do xfreerdp.


# 3. Conectando no wifi com wpa_supplicant

* Criar o arquivo `wpa_supplicant.conf`:

$ sudo nano /etc/wpa_supplicant.conf

No arquivo aberto, incluir o nome da rede (`ssid`) e a senha (`psk`) da seguinte forma:

```
network={
         ssid="NOME_DA_SUA_REDE"
         psk="SENHA"
}
```

Salvar e sair do `nano`.

* Detectando a interface de rede wifi

Executar o comando `ip a`.

Para efeitos de exemplo, digamos que a interface identificada tenha sido `wlp2s0`.

* Ativando a rede wifi

Executar estes comandos em sequência:

```
$ sudo ip link set wlp2s0 down
$ sudo ip link set wlp2s0 up
$ sudo wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant.conf -Dnl80211,wext
$ sudo dhclient wlp2s0
```

Com isso, a conexão wifi já deve estar funcionando.


* Se precisar reiniciar antes de ter um ambiante gráfico

Basta executar novamente:

```
$ sudo wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant.conf -Dnl80211,wext
$ sudo dhclient wlp2s0
```

# 4. Adicionado chave SSH ao git

Necessário primeiro criar uma chave SSH na máquina onde será feita a conexão, é necessário o pacote openssh-client:

```
ssh-keygen -t RSA -C <email>
```

A partir daí o prompt irá solicitar o local para salvar as chaves, se pressionarmos a tecla enter será salvo em `~/.ssh/id_rsa` por padrão.
É possível também inserir uma passphrase que será solitada sempre que for necessário usar a chave.

Feito isso, bastar copiar a chave pública que foi salva em `~/.ssh/id_rsa.pub` e copiar chave que será inserida no github em:
_settings -> SSH and GPG Keys_ e inserir no campo _key_, é possível também adicionar uma descrição no campo _title_.

Para finalizar, vamos fazer a comunicação para fazer a troca de chaves:
```
SSH -T git@github.com
```

Caso tudo tenha dado certo, as chaves serão trocadas e inseridas de maneira permanente no Github.


# 5. Ativando/desativando hibernação no systemd


Se você quiser impedir que seu sistema tente hibernar, use o systemd para desabilitar a função.
A seguinte linha de comando, deve resolver o assunto:
```sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target```

Se quiser desfazer o procedimento, realize o seguinte comando:
```sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target```

Para desabilitar a suspensão quando a tampa do notebook for fechada, ajuste os seguintes parâmetros no arquivo de configuração /etc/systemd/logind.conf:
[Login]
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore

    Obs.: no meu caso eu também adicionei as linhas:
    HandlePowerKey=ignore
    HandleSuspendKey=ignore
    HandleHibernateKey=ignore

Em seguida rode o comando systemctl, da seguinte forma:
systemctl restart systemd-logind.service
