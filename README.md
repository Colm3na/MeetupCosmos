# MeetupCosmos
Repositorio para el meetup virtual de Cosmos.

>¿Qué hace falta para este meetup?
>- Conexión a internet.
>- Ordenador con Windows y una maquina virtual instalada con Ubuntu, en el ejemplo se usará una versión minimal de [Ubuntu 18.04LTS](http://releases.ubuntu.com/18.04.4/).

> _Podéis encontrar el enlace para la descarga de una imagen de Ubuntu con todo lo necesario para el meetup en la carpeta [virtualbox](virtualBox/) o podéis descargarlo desde la [web de la Colmena](https://www.colmenalabs.org/vm/), el archivo pesa `3,3GB`._ 

>_Las credenciales para la maquina virtual son las siguientes:_
>```
>user: user
>passwd: CosmosMeetup
>```

# ¿Qué aprenderemos en este meetup?

La meta es aprender a usar [gaia](https://github.com/cosmos/gaia) e interactuar con un nodo sin necesidad de instalarlo ni de usar wallets de terceros, siendo autosuficientes para la gestion de nuestra wallet. Aprender a realizar transacciones, delegar y redelegar.

**1. ¿Qué es Gaia?**
 Gaia es el nombre de la aplicacion de Cosmos-SDK para el Hub de Cosmos, como tal dentro podemos encontrar varias partes _(el mejor simil es el hardware y el software de un ordenador)_. Así pues, en Gaia podemos encontrar `gaiacli` y `gaiad`.
 
`gaiad` es el demonio de Gaia, se usa para hacer funcionar el nodo, sería el hardware del ordenador.
`gaiacli` Es la interface en línea de comandos para comunicarnos con nuestro nodo, sería el software de nuestro ordenador.
`gaia` viene con los siguientes módulos (_más info [aquí](https://github.com/wimel/gaia/blob/master/docs/translations/es/what-is-gaia.md#qu%C3%A9-es-gaia)_):
- `x/auth`: Cuentas y firmas.
- `x/bank`: Transferencia de tokens.
- `x/staking`: Lógica para el _staking_.
- `x/mint`: Lógica para la inflacción.
- `x/distribution`: Lógica para la distribución del FEE.
- `x/slashing`: Lógica para la penalización.
- `x/gov`: Lógica para la gobernanza.
- `x/ibc`: Transferencia entre blockchains.
- `x/params`: Controla los parámetros del nivel de la aplicación.

**2. ¿Cómo instalar Gaia y usar gaiad y gaiacli sin tener un _full-node_?**

- En windows:
Tenemos varias formas de instalar gaia en Windows, una es usando el subsistema de windows para Linux, y la otra es crear una máquina virtual e instalar Linux. [Aquí](https://ubunlog.com/wsl-como-instalar-y-usar-el-susbistema-ubuntu-en-windows-10/) una guia para activar el susbsistema de Linux en Windows y [aquí](https://blog.desdelinux.net/virtualbox-6-1-ya-esta-disponible-llega-con-soporte-de-kernel-de-linux-5-4-reproduccion-de-video-acelerada-y-mas/) para crear una máquina virtual en Windows. Hay algunos problemas con la virtualización en Windows, si tenemos problemas observar si en la BIOS tenemos activada la opción de virtualización. La otra forma _(no probada)_ es usar [`Hyper-V`](https://blogs.itpro.es/eduardocloud/2016/05/26/usas-virtual-box-y-tienes-windows-10-no-lo-necesitas-2/)
_Desde ahora pasamos al siguiente punto, se asume que se tiene instalado un sistema Linux operativo._

- En Linux _(probada con Ubuntu 18.04LTS versión minimal)_:
Los mayores fallos respecto a la instalación de Gaia en Linux vienen por **las variables de entorno y el `$PATH`** _(para más información leer [este](https://elpuig.xeill.net/Members/rborrell/articles/los-archivos-bashrc-bash_profile-etc-bashrc-etc-profile-los-archivos-bashrc-bash_profile-etc-bashrc-etc-profile-cual-utilizar) artículo)_, mucho cuidado con ellas pues nos darán error y no podremos interactuar con el nodo, podéis ver más info en la [documentación oficial](https://golang.org/doc/install) de Go (_lo que está escrito después de `#` son comentarios_).

```bash
#actualizamos:
sudo apt update && sudo apt -y upgrade
#instalamos dependencias:
sudo apt install -y git make gcc
#instalamos Go:
wget -c 'https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz' -O  go1.14.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.14.2.linux-amd64.tar.gz 
sudo rm -Rf go1.14.2.linux-amd64.tar.gz
#añadimos a nuestro path lo siguiente, abrimos el archivo con "nano .bashrc" y añadimos al final del mismo:
export PATH="$PATH:/usr/local/go/bin"
export GOPATH="$HOME/go"
export PATH="$PATH:$GOROOT/bin:$GOPATH/bin"
export GOBIN="$GOPATH/bin"
#recargarmos la terminal para cargar la versión de Go:
source .bashrc
#comprobamos la versión de Go, el resultado debería ser "go version go1.14.2 linux/amd64":
go version
```
>Instalamos gaia _(comprobar en el repositorio oficial de Gaia la [última versión](https://github.com/cosmos/gaia/releases))_

```bash
#clonamos el repositorio y entramos dentro de la carpeta:
git clone https://github.com/cosmos/gaia.git && cd gaia/
#usamos la rama correcta, por favor comprobad antes en el repositorio de Cosmos la última versión o la versión correcta que necesitamos, para cambiar de versión simplemente hacemos un "git checkout v2.0.8" y compilamos de nuevo con "make install"(esta versión es para la testnet):
git checkout v2.0.3
#compilamos
make install
#comprobamos la version de gaiacli y gaiad, debería aparecernos "version: 2.0.3":
gaiacli version --long
gaiad version --long
#creamos nuestra wallet (en caso de querer usar la Ledger, simplemente añadimos la opción "--ledger"), nos preguntará una contraseña y es importante hacer varias copias de las 24 palabras, pues es la única forma de recuperar nuestra wallet si le pasara algo a nuestro ordenador:
gaiacli keys add wimelDelega
#creamos los primeros archivos de configuración (cambiad "wimelDelega" por vuestro nombre, no es necesario realizar este paso):
gaiad init wimelDelega
```

>Si hemos seguido estos pasos ya tenemos instalado en nuestro ordenador Gaia, y una wallet lista para ser usada. Es importante, muy importante **hacer más de una copia de las 24 palabras** para poder recuperar nuestra wallet si algo pasara en nuestro ordenador.

**3. ¿Cómo interactuar con un nodo sin instalar un _full-node_?**

Aquí la clave está en el archivo situado en `.gaiacli/config` llamado `config.toml`, dentro de ese archivo podemos modificar los valores de la red con la cual queremos interactuar, por ejemplo para la testnet sería:
```bash
#configuramos el nodo externo para mandarle nuestras transacciones:
gaiacli config node tcp://128.199.44.53:26657
#configuramos la red en la cual vamos a interactuar:
gaiacli config chain-id gaia-13007
```
>Nota: En mainnet, podemos usar los siguientes valores:
>```bash
>#configuramos el nodo externo para mandarle nuestras transacciones, recordad pedir a vuestros validadores un nodo con el cual podamos interactuar, los siguientes valores son del nodo de DelegaNetworks:
>gaiacli config node tcp://144.76.155.231:26657
>#configuramos la red en la cual vamos a interactuar:
>gaiacli config chain-id cosmoshub-3
>```

**4. Realizando una transacción _(recordad cambiar `umuon` a `uatom` cuando hagamos transacciones en mainnet, en caso de que queramos usar la Ledger, solo debemos añadir el flag "`--ledger`")_**

>El token de Cøsmos en mainnet es el `atom` y en testnet `muon`

>1 `atom` = 1000000 `uatom`

>1 `muon` = 1000000 `umuon`

```bash
gaiacli tx send "NUESTRAWALLET" "WALLETDESTINO" "CANTIDAD"umuon --trust-node=true
```

**5. ¿Cómo realizar una delegación?**
```bash
gaiacli tx staking delegate "cosmosvaloper169..." "CANTIDAD"umuon --from NUESTRAWALLET --trust-node=true
```

**6. ¿Cómo redelegar nuestros fondos sin pasar por el período de _unbond_ de 21 dias**

>_La redelegación sólo lo podremos realizar una vez, en caso de querer volver a realizar una redelegación debemos esperar 21 días_

```bash
gaiacli tx staking redelegate ANTIGUOVALIDADOR NUEVOVALIDADOR "cantidad"umuon --from NUESTRAWALLET --trust-node=true
```

**7. ¿Cómo votar en un proceso de gobernanza abierto?**

>Recuerda que debes **delegar fondos antes de participar en los procesos de gobernanza**

>Para ver los procesos de gobernanza que actualmente están en período de votación:
>```bash
>gaiacli query gov proposals --status voting_period
>```

```bash
gaiacli tx gov vote "IDPROPOSICION" "Yes-No-NoWithVeto-Abstain" --from NUESTRAWALLET  -y
```

# Enlaces de interés:

[Github Gaia](https://github.com/cosmos/gaia)

[Github Cosmos](https://github.com/cosmos)

[Portal para desarrolladores](https://cosmos.network/developers)

[Documentación Cosmos SDK](https://docs.cosmos.network/)

[Documentación Cosmos Hub](https://hub.cosmos.network/master/hub-overview/overview.html)

[Telegram Cosmos _(inglés)_](https://t.me/cosmosproject)

[Telegram Cosmos _(español)_](https://t.me/Cosmos_Network_ES)

[Discord _(general)_](https://discord.com/channels/669268347736686612/669275164999155742)

[Discord _(español)_](https://discord.com/channels/669268347736686612/669488959159533584)

[Repo Gavinly, buenas prácticas en los procesos de gobernanza_(en inglés)_](https://github.com/gavinly/CosmosCommunitySpend)
